# 🐸 Kaeru Tunnel

> 無事カエル — 平安回家。連回自己家的私人通道，讓一般人也能在外面安全使用家裡的網路。
> Your private way back home — WireGuard & NetBird made simple on OpenWrt.

**這不是翻牆或換 IP 的商業 VPN 服務。** Kaeru Tunnel 讓你在外面用手機或筆電，安全地連回「自己家」的 OpenWrt 路由器。

## 為什麼做這個

OpenWrt 本身就支援 WireGuard 和 NetBird，但設定時得懂公網 IP、轉發埠、金鑰、AllowedIPs、防火牆區域，對一般人來說門檻太高。

Kaeru Tunnel 的出發點：**真正的門檻不是介面，而是網路環境。** 所以它最重要的工作，是先幫你判斷家裡的網路能走哪條路，再用你聽得懂的話一步步帶你設定好。

## 它會做什麼

- **自動檢測網路環境**：判斷有沒有公網 IP、是不是 CGNAT 或雙層 NAT，再建議要用哪種模式
- **兩種回家方式**
  - 有公網 IP → 原生 WireGuard，不依賴任何外部平台
  - 沒有公網 IP → NetBird，透過控制平台穿透 NAT
- **一鍵新增裝置**：輸入名字 → 自動分配 IP → 掃 QR code 就能連；QR 只顯示一次，之後路由器上不留手機的私鑰
- **用生活語言**：「出門時網路全部走家裡」／「只連家裡的設備」，不出現 peer、AllowedIPs
- **裝置清單**：顯示「3 分鐘前在線」，手機遺失可以一鍵停用
- **安全網**：設定套用失敗會自動回滾；頁面只開放給區域網路

## 設計原則

- 手機優先的簡易網頁，和 LuCI 共用同一份 UCI 設定
- LuCI 保留給進階使用者，Kaeru Tunnel 不取代它
- 後端走 rpcd / ubus，盡量不增加額外的背景服務

## 狀態

🚧 規劃階段，尚未有可安裝的版本。

- 第一階段：WireGuard 設定精靈＋網路環境檢測
- 第二階段：NetBird 模式

詳見 [開發路線圖](docs/roadmap.md)。

## 相容性（目標）

- OpenWrt 24.10（opkg）、25.12 以上（apk）
- 優先測試 x86_64 與 arm64

## 開發文件

| 文件 | 內容 |
|---|---|
| [docs/handoff.md](docs/handoff.md) | 專案交接單：一頁看完所有決策與待辦 |
| [docs/research.md](docs/research.md) | WireGuard vs NetBird 評估與參考資料 |
| [docs/ux.md](docs/ux.md) | UX 原則、畫面草稿、用語對照表 |
| [docs/architecture.md](docs/architecture.md) | 技術架構、UCI 設定對應、網路檢測邏輯 |
| [docs/roadmap.md](docs/roadmap.md) | 分階段開發計畫與檢查清單 |
| [docs/naming.md](docs/naming.md) | 專案命名由來與決策紀錄 |
| [docs/decisions.md](docs/decisions.md) | 待決定事項 |

## 授權

待定（候選：GPL-2.0 或 Apache-2.0，見 [docs/decisions.md](docs/decisions.md)）。
