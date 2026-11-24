# Kaeru Tunnel — 給 AI 協作者的說明

## 專案是什麼

給一般人用的 OpenWrt 網頁，讓手機在外面能安全連回「自己家」。不是翻牆或換 IP 的商業 VPN。
自動判斷網路環境（公網 IP／CGNAT／雙層 NAT），有公網 IP 走原生 WireGuard，沒有則走 NetBird（第二階段）。

## 開始工作前先讀

1. `docs/handoff.md`：一頁看完所有決策
2. `docs/architecture.md`：技術架構與 UCI 對應
3. `docs/ux.md`：UX 原則與用語對照表
4. `docs/decisions.md`：還沒決定的事——不要自行替使用者決定，有需要就問

## 規則

- 文件與介面文字使用**繁體中文（台灣用語）**
- 介面上不出現 peer、AllowedIPs、endpoint、exit node 等技術名詞，一律照 `docs/ux.md` 的對照表
- 手機私鑰不得寫入 UCI、log 或任何檔案
- 不新增常駐背景服務；後端邏輯放在 rpcd 外掛
- Kaeru 只管理自己建立的 `kaeru` 介面與防火牆區域，不改動使用者其他設定
- 所有設定變更必須可回滾
- 目標相容 OpenWrt 24.10（opkg）與 25.12 以上（apk）
- `docs/architecture.md` 中標示「待驗證」的項目，實作前先在實機確認，確認後更新文件
