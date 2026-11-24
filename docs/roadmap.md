# 開發路線圖

## 第零階段：準備

- [ ] 決定授權（GPL-2.0 或 Apache-2.0），加入 LICENSE
- [ ] 決定目標使用者（家人自用 or 產品化）
- [ ] 準備測試機：x86_64（Proxmox VM）與 arm64（例如 NanoPi R2S）
- [ ] 準備兩種網路環境：有公網 IP、CGNAT（例如手機熱點當 WAN）
- [ ] 在實機上驗證 [architecture.md](architecture.md) 中所有「待驗證」項目

## 第一階段：WireGuard 精靈（MVP）

### 後端
- [ ] rpcd 外掛骨架與 ACL
- [ ] 網路環境檢測 API（WAN IP、公網 IP、私有／CGNAT 判斷）
- [ ] 建立 `kaeru` WireGuard 介面與防火牆區域（冪等：重複執行不會壞）
- [ ] 新增裝置：自動分配 IP、產生金鑰、回傳 QR 內容、不保留手機私鑰
- [ ] 裝置清單與最後握手時間
- [ ] 停用裝置
- [ ] 切換「全部走家裡／只連家裡」
- [ ] 套用失敗自動回滾

### 前端
- [ ] 畫面 0：環境檢測與建議
- [ ] 畫面 1：狀態首頁與裝置清單
- [ ] 畫面 2：新增裝置與 QR（只顯示一次）
- [ ] 畫面 3：用行動網路測試連線
- [ ] 錯誤訊息全部改成生活語言（見 [ux.md](ux.md) 對照表）

### 打包
- [ ] OpenWrt 套件 Makefile
- [ ] 同時產出 24.10（ipk）與 25.12（apk）
- [ ] 安裝與移除都不殘留設定（移除時詢問是否保留裝置）

### 驗收
- [ ] 不懂網路的人，在沒有說明的情況下 5 分鐘內讓手機連回家
- [ ] LuCI 裡看得到 Kaeru 建立的介面與 peer，且兩邊修改不會互相弄壞

## 第二階段：NetBird 模式

- [ ] 檢測到 CGNAT 時引導進 NetBird
- [ ] 評估：自己整合 vs 依賴 luci-app-netbird
- [ ] 安裝 NetBird（架構偵測、版本選擇）
- [ ] 用 setup key 綁定路由器
- [ ] 建立 wt0 介面與防火牆
- [ ] 狀態頁
- [ ] 評估串 Management API token，做到「全部走家裡」一鍵開關

## 之後可以考慮

- [ ] IPv6 公網可達時的 WireGuard 方案（沒有 IPv4 公網也能直連）
- [ ] 一鍵 DDNS
- [ ] 多語系（繁中、英文、日文）
- [ ] LuCI 選單捷徑（luci-app-kaeru-tunnel）
- [ ] 提交到 OpenWrt packages feed
