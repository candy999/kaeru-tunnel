# 評估：原生 WireGuard vs NetBird

> 研究日期：2026-11-17。外部專案和價格會變動，動工前請再確認一次。

## 一句話選擇

- 想讓路由器完全自主、不依賴外部控制平台，且家裡有公網 IP → **原生 WireGuard**
- 沒有公網 IP，但仍想把家裡的 OpenWrt 當成「回家跳板」或「出口節點」→ **NetBird**

## 原生 WireGuard

### 一般人會卡在哪

- **網路條件**：需要公網 IP、DDNS；如果 OpenWrt 前面還有 ISP 數據機，上游也要轉發埠（預設 UDP 51820）。這些不是介面做漂亮就能解決的。
- **QR code 的私鑰問題**：LuCI（`luci-proto-wireguard`）可以產生金鑰和 QR，但只有在 peer 設定裡存著遠端裝置的私鑰時才產生得出 QR。也就是手機的私鑰由路由器產生並留在路由器上。
  - Kaeru 的對策：QR 只顯示一次，顯示後從 UCI 刪掉該 peer 的 `private_key`，只保留 `public_key`。
- **IP 與 AllowedIPs**：使用者常卡在「下一個 WireGuard IP 要用哪個」「Allowed IPs 要填什麼」。社群已有人做一鍵 peer 產生器，證明痛點真實存在。

### 優點

- 不依賴任何外部平台，UX 可以完全自己掌控
- OpenWrt 官方支援成熟，核心模組效能好

## NetBird

### 一般人會卡在哪

- **原生安裝流程**：SSH 安裝套件 → `netbird login --setup-key` → 在 LuCI 手動新增 Unmanaged 介面（device `wt0`）→ 建立防火牆區域並設定轉發。
- **套件版本落後**：OpenWrt feed 裡的 NetBird 常落後官方好幾版，維護者很少回移植到穩定版。
- **Flash 空間**：NetBird 是 Go 寫的，執行檔相對大，小容量路由器可能裝不下。
- **控制平台是必要的**：
  - 雲端免費方案：最多 5 位使用者、100 台機器
  - 自架：Linux VM 至少 1 CPU、2 GB RAM，對外開 TCP 80/443、UDP 3478，還要有一個指向它的公開網域
- **出口節點與路由在後台設定**：這些設定在 NetBird 管理後台，不在路由器上。若要讓路由器頁面一手包辦，需要串 NetBird Management API（personal access token）。

### 優點

- 手機端最友善：裝 NetBird app、登入就好
- 不需要公網 IP，能穿透 CGNAT

### 既有社群專案：luci-app-netbird

- 2026/6 發布，在 LuCI 中提供連線／斷線／註銷、自架管理伺服器、斷線自動重連、peer 狀態與即時 log、防火牆區域與 LAN↔mesh 轉發自動化
- 執行檔預設抓官方 release，只支援 amd64、arm64、386、armv6；其他架構退回 OpenWrt feed 版本
- 相容 OpenWrt 24.x（opkg）與 25.x（apk）
- **對 Kaeru 的意義**：第二階段可以參考它的防火牆自動化做法，或評估直接整合、不重造輪子。

## 參考資料

- [luci-app-netbird（OpenWrt 論壇）](https://forum.openwrt.org/t/luci-app-netbird-web-ui-for-netbird-on-openwrt/251349)
- [luci-app-netbird GitHub](https://github.com/looong-cat/luci-app-netbird)
- [NetBird 支援討論串（OpenWrt 論壇）](https://forum.openwrt.org/t/netbird-support-discussion-thread/237831)
- [NetBird OpenWrt 安裝文件](https://docs.netbird.io/get-started/install/openwrt)
- [WireGuard QR 需要 peer 私鑰的討論](https://forum.openwrt.org/t/wireguard-qr-image-missing/136560/10)
- [LuCI 將 client 私鑰存在 server 設定的問題回報](https://forum.openwrt.org/t/luci-wireguard-interface-incorrectly-stores-client-private-keys-in-server-configuration/242852)
- [WireGuard 1-Click Peer generator](https://forum.openwrt.org/t/wireguard-1-click-peer-generator/252038)
- [openwrt-wireguard-installer（互動式安裝腳本，含回滾）](https://github.com/coralesoft/openwrt-wireguard-installer)
- [OpenWrt WireGuard Server 官方文件](https://openwrt.org/docs/guide-user/services/vpn/wireguard/server)
- [OpenWrt WireGuard road-warrior 自動化](https://openwrt.org/docs/guide-user/services/vpn/wireguard/road-warrior)
- [NetBird 方案與價格](https://docs.netbird.io/how-to/plans-and-billing)
- [NetBird 自架 Quickstart](https://docs.netbird.io/selfhosted/selfhosted-quickstart)
