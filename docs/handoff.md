# 🐸 Kaeru Tunnel 專案交接單

> 一頁看完這個專案的來龍去脈。細節請看各份文件。

## 1. 專案基本資料

- 專案名稱：Kaeru Tunnel
- Repo：`candy999/kaeru-tunnel`
- 未來 LuCI 套件名稱：`luci-app-kaeru-tunnel`
- 名稱由來：日文「帰る」＝回家，與「蛙」同音；日本青蛙御守「無事カエル」＝平安回家（見 [naming.md](naming.md)）
- 一句話定位：連回自己家的私人通道（不是翻牆服務），讓一般人也能在外面安全使用家裡的網路
- 平台：OpenWrt 路由器
- 授權：待定

## 2. 要解決的問題

OpenWrt 本身支援 WireGuard 與 NetBird，但設定時要懂公網 IP、轉發埠、金鑰、AllowedIPs、防火牆區域，一般人做不到。

**核心觀點：真正的門檻不是介面，而是網路環境。** UI 最大的價值，是幫使用者自動判斷該走哪條路，使用者不必懂兩種模式的差別。

## 3. 兩種模式摘要（詳見 [research.md](research.md)）

| | 原生 WireGuard | NetBird |
|---|---|---|
| 適用情境 | 家裡有公網 IP | 沒有公網 IP、CGNAT |
| 一般人的門檻 | 網路條件：公網 IP、DDNS、上游數據機轉發埠 | 帳號與平台：setup key、控制平台 |
| 外部依賴 | 無 | 雲端控制平台或自架 VPS |
| 手機端 | WireGuard app 掃 QR | NetBird app 登入 |
| 路由器能完全掌控 | 能 | 出口節點與路由在 NetBird 後台，需串 API token |

## 4. 產品形式決策

- 做一個**路由器上的獨立簡易網頁**，以手機為主，約 3 個畫面（狀態／新增裝置／出門模式）
- 定位是 LuCI 的前台，不取代 LuCI；LuCI 留給進階使用者
- 後端建議走 rpcd / ubus 讀寫 UCI，與 LuCI 共用同一份設定
- 不做雲端服務或獨立 App

## 5. UX 原則（詳見 [ux.md](ux.md)）

1. 自動檢測：比對 WAN IP 和實際公網 IP，判斷有沒有公網 IP、CGNAT、雙層 NAT，再直接建議用哪個模式
2. 用生活語言取代 peer、AllowedIPs、exit node
3. 一鍵新增裝置：輸入名字 → 自動分配 IP → 顯示 QR；QR 只顯示一次，之後刪掉路由器上存的手機私鑰
4. 裝置清單用最後一次握手時間顯示「3 分鐘前在線」；遺失可一鍵停用
5. 設定套用失敗就自動回滾
6. 頁面只開放給 LAN

## 6. 開發路線（詳見 [roadmap.md](roadmap.md)）

- 第一階段（MVP）：原生 WireGuard 設定精靈＋網路環境檢測
- 第二階段：檢測到沒有公網 IP 時，引導進 NetBird 模式

## 7. 相容性目標

- OpenWrt 24.10（opkg）、25.12 以上（apk）
- 優先測試 x86_64 與 arm64（例如 NanoPi R2S）

## 8. 待決定事項（詳見 [decisions.md](decisions.md)）

- [ ] 目標使用者：自己家人，還是做成產品給別人用
- [ ] 授權：GPL-2.0 或 Apache-2.0
- [ ] 前端技術選型
- [ ] 第二階段 NetBird 用雲端還是自架控制平台
