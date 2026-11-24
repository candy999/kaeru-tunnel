# 技術架構

> 本文件是規劃階段的建議架構。標示「待驗證」的部分，動工前要在實機上確認。

## 總覽

```
手機瀏覽器（LAN）
      │  HTTP（uhttpd）
      ▼
/www/kaeru/          靜態網頁（HTML + JS，手機優先）
      │  JSON-RPC（/ubus，帶 rpcd session）
      ▼
rpcd 外掛  kaeru.*   後端邏輯（ucode 或 shell，待選）
      │
      ├── UCI：network（wg 介面與 peer）、firewall、ddns
      ├── wg show / netbird status（讀狀態）
      └── 外部查詢：公網 IP（檢測用）
```

原則：

- **不新增常駐背景服務**。網頁經由 uhttpd 提供，邏輯放在 rpcd 外掛裡，只在被呼叫時執行。
- **與 LuCI 共用 UCI**。Kaeru 寫進去的設定，LuCI 的 Network → Interfaces 裡看得到也改得了。
- **登入沿用路由器的 root 密碼**：透過 rpcd 的 `session login` 取得 session，權限用 `/usr/share/rpcd/acl.d/` 的 ACL 限制在 Kaeru 需要的範圍。

## 套件結構（規劃）

```
kaeru-tunnel/
├── root/
│   ├── www/kaeru/                    # 前端靜態檔
│   ├── usr/share/rpcd/acl.d/kaeru.json
│   └── usr/share/rpcd/ucode/kaeru.uc # 或 /usr/libexec/rpcd/kaeru（shell）
├── luasrc 或 htdocs/                 # 第二階段：luci-app 入口（LuCI 選單捷徑）
└── Makefile                          # OpenWrt 套件 Makefile
```

## WireGuard 模式的 UCI 對應

Kaeru 管理一個固定名稱的介面（暫定 `kaeru`），避免碰到使用者自己建的 WireGuard 介面。

```
# /etc/config/network
config interface 'kaeru'
    option proto 'wireguard'
    option private_key '<路由器私鑰>'
    option listen_port '51820'
    list addresses '10.77.0.1/24'          # 網段待定，需避開常見 LAN 網段

config wireguard_kaeru
    option description '爸爸的手機'         # 介面上的「裝置名稱」
    option public_key '<手機公鑰>'
    list allowed_ips '10.77.0.2/32'
    option route_allowed_ips '1'
    # private_key 只在產生 QR 的瞬間存在，顯示後刪除
```

```
# /etc/config/firewall
config zone
    option name 'kaeru'
    list network 'kaeru'
    option input 'ACCEPT'
    option output 'ACCEPT'
    option forward 'REJECT'

config forwarding                      # 連回家裡的設備
    option src 'kaeru'
    option dest 'lan'

config forwarding                      # 「全部走家裡」時才需要
    option src 'kaeru'
    option dest 'wan'

config rule
    option name 'Allow-Kaeru-WireGuard'
    option src 'wan'
    option proto 'udp'
    option dest_port '51820'
    option target 'ACCEPT'
```

### 裝置的兩種模式（寫在手機端設定裡）

| 介面上的說法 | 手機端 `AllowedIPs` | 手機端 `DNS` |
|---|---|---|
| 網路全部走家裡 | `0.0.0.0/0, ::/0` | 路由器的 Kaeru IP（例：10.77.0.1） |
| 只連家裡的設備 | 家中 LAN 網段＋Kaeru 網段 | 可不設 |

「全部走家裡」時 dnsmasq 要能在 `kaeru` 介面上回應 DNS（待驗證預設是否已涵蓋）。

### 新增裝置流程

1. 找下一個可用 IP（掃描現有 `wireguard_kaeru` 的 `allowed_ips`）
2. 在路由器上產生手機的金鑰對（`wg genkey | wg pubkey`）
3. 寫入 peer（只存 public_key），`uci commit`、重載介面
4. 組出手機端設定（含手機私鑰、路由器公鑰、endpoint、AllowedIPs、DNS），產生 QR 回傳給前端
5. 手機私鑰不寫入 UCI，回傳後即丟棄 → 「只顯示一次」

### 裝置狀態

- `wg show kaeru latest-handshakes` → 每個公鑰的最後握手時間 → 換算成「3 分鐘前在線」
- 也可評估直接用 `luci-proto-wireguard` 提供的 rpcd 介面（待驗證名稱與輸出格式）

## 網路環境檢測

| 檢查 | 做法 | 判斷 |
|---|---|---|
| WAN 位址 | `ubus call network.interface.wan status` 取 IPv4 | — |
| 實際公網 IP | 向外部服務查詢（例：STUN 或 IP echo 服務，待選） | — |
| WAN 是私有位址 | WAN IP 落在 10/8、172.16/12、192.168/16 | 前面還有一台 NAT 設備（雙層 NAT） |
| WAN 是 CGNAT 位址 | WAN IP 落在 100.64.0.0/10 | 電信商 CGNAT，WireGuard 連不進來 |
| WAN = 公網 IP | 兩者相同 | 可直接用 WireGuard |
| IPv6 | 檢查 WAN 是否有全域 IPv6 | 加分項：沒有 IPv4 公網時，IPv6 也可能是解法（第二階段評估） |

**限制：** 從路由器內部無法 100% 證明外面連得進來（上游可能還有防火牆）。最終驗證靠 UX 畫面 3：用手機行動網路實際連一次，看到握手才算成功。

## DDNS

公網 IP 多半是浮動的，需要 DDNS。第一階段可以：

- 偵測是否已裝 `ddns-scripts`、是否已設定 → 有就直接拿 hostname 當 endpoint
- 沒有的話，先讓使用者手動填，或暫時用目前的公網 IP（並提示 IP 可能會變）

要不要內建一鍵 DDNS（選哪家免費服務）列入待決定事項。

## 套用失敗自動回滾

參考 LuCI 的 apply-confirm 機制：套用網路／防火牆設定後，前端必須在 N 秒內回報「我還連得到」，否則後端還原到套用前的 UCI 快照（待驗證可否直接沿用 rpcd 的 `uci apply` rollback 功能）。

## NetBird 模式（第二階段）

- 安裝：偵測架構，優先用官方 release，其他架構退回 OpenWrt feed（參考 luci-app-netbird 的做法）
- 綁定：使用者貼上或掃描 setup key → `netbird up --setup-key <key>`
- 網路：建立 Unmanaged 介面（device `wt0`）與防火牆區域、LAN 轉發
- 狀態：`netbird status` 的 JSON 輸出（待驗證參數）
- 「出門時網路全部走家裡」：需要在 NetBird 後台把路由器設成出口節點 → 需要 Management API token，或改成引導使用者到後台操作

## 安全

- 頁面只綁 LAN：uhttpd 與防火牆層都只允許 LAN 存取
- ACL 最小權限：只開放 Kaeru 需要的 UCI 設定檔與指令
- 手機私鑰不落地（不寫入 UCI、不寫入 log）
- 停用裝置 = 從 UCI 刪除該 peer 並重載介面
