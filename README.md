Hermes Agent for macOS

一個原生的 macOS 桌面應用程式，用於 Hermes Web UI。使用 Swift 和 WKWebView 開發——沒有 Electron，除了 Xcode Command Line Tools 外沒有其他依賴。由 @redsparklabs 建立。

<img width="1470" height="922" alt="Hermes Agent 截圖" src="https://github.com/user-attachments/assets/8e704e62-c736-4827-ba50-c41a21d9922f" />

---

需求

Hermes Agent 是一個用於 Hermes Web UI 的原生視窗。應用程式本身只是個包裝，必須有 Hermes Web UI 在某處運行才能使用。否則啟動時會看到連線錯誤。

必要條件： Hermes Web UI 在本機或遠端伺服器運行。
遠端伺服器選用： 已設定 SSH 金鑰驗證。
macOS 版本： 12 (Monterey) 或更新版本。

---

安裝設定

根據你的情況選擇路徑。

路徑 1 — 新手，全部在本機安裝

首先在 Mac 上運行 Hermes Web UI：

git clone https://github.com/nesquena/hermes-webui.git ~/hermes-webui-public
cd ~/hermes-webui-public
bash start.sh

這會在 http://localhost:8787 啟動 Hermes 伺服器。依照 Hermes Web UI README 於首次啟動時設定 API 金鑰。

接著安裝 Hermes Agent：

從 Releases 下載最新的 Hermes-Agent-vX.X.X.dmg
打開 DMG 並將 Hermes Agent 拖到 Applications 資料夾
啟動應用程式——它預設連線到 http://localhost:8787，也就是剛啟動的 Hermes Web UI

無需額外設定，開箱即用。

---

路徑 2 — 已在本機運行 Hermes Web UI

如果 Hermes Web UI 已經在 Mac 上運行：

從 Releases 下載最新 DMG
將 Hermes Agent 拖到 Applications 並啟動

預設目標 URL 為 http://localhost:8787。如果你使用不同的埠號，打開 偏好設定 (⌘,) 更新目標 URL，點擊 Save & Reconnect。

可先用 Test Connection 驗證連線再儲存。

---

路徑 3 — 遠端伺服器上的 Hermes Web UI

如果 Hermes Web UI 在可透過 SSH 存取的伺服器運行：

開始前： 確保 SSH 金鑰驗證運作正常，能用 ssh user@your-server 免密碼連線。

從 Releases 下載最新 DMG
將 Hermes Agent 拖到 Applications 並啟動
打開 偏好設定 (⌘,)
將 Connection Mode 設為 SSH Tunnel
填寫：
Username — 伺服器的 SSH 使用者
Host — 伺服器的主機名稱或 IP
Local Port — Mac 上的本地埠（預設：8787）
Remote Port — 伺服器上 Hermes Web UI 的埠（預設：8787）
點擊 Test Connection 驗證後，再按 Save & Reconnect

應用程式在啟動時會建立 SSH 通道、監控並在退出時自動關閉。視窗底部狀態列會顯示通道狀態，若斷線可一鍵重連。

---

安裝方式

下載 DMG（推薦）

前往 Releases，下載最新 DMG，打開後將 Hermes Agent 拖到 Applications。應用程式已使用開發者 ID 簽署並經 Apple 公證——不會觸發 Gatekeeper 警告。

從原始碼建置

需要 Xcode Command Line Tools：

xcode-select --install   # 若尚未安裝

git clone https://github.com/hermes-webui/hermes-swift-mac.git
cd hermes-swift-mac
./build.sh

這會編譯應用程式、打包圖示並安裝到 /Applications/Hermes Agent.app。

---

功能

原生 macOS 應用程式——Dock 圖示、標準選單列、行為如同其他 Mac App
以 WKWebView 載入 Hermes Web UI（無需瀏覽器，無 Electron）
本機直接模式與遠端 SSH 通道模式
剪貼簿整合——可直接貼上文字與圖片 (⌘V) 到對話
透過迴紋針按鈕上傳檔案
偏好設定 (⌘,) 視窗，含連線測試按鈕
狀態列顯示 SSH 通道狀態，一鍵重連
背景執行時 AI 回覆完成會發送 macOS 通知
語音輸入——首次使用會請求麥克風權限
外部連結自動在 Safari 開啟
Sparkle 自動更新——啟動時檢查新版本，可手動檢查更新
已簽署並公證——首次開啟無 Gatekeeper 警告

---

設定

打開 偏好設定 (⌘,)：

設定
說明
Connection Mode
直接（本機）或 SSH 隧道
Target URL
載入的 URL（預設：http://localhost:8787）
Username
SSH 使用者（僅 SSH 模式）
Host
SSH 伺服器主機或 IP（僅 SSH 模式）
Local Port
本機通道埠（僅 SSH 模式）
Remote Port
伺服器上 Hermes Web UI 埠（僅 SSH 模式）

設定會在重啟後保留。

---

鍵盤快捷鍵

快捷鍵
動作
⌘,
打開偏好設定
⌘R
重新載入 WebUI 頁面
⌘W
隱藏視窗（應用程式仍在 Dock 運行）
⌘⇧H
從任何應用程式喚回 Hermes（全域）
⌘+ / ⌘−
放大 / 縮小
⌘0
重設縮放

全域快捷鍵 ⌘⇧H 可在系統任意位置將 Hermes 立即帶到前景。

---

SSH 安全性

StrictHostKeyChecking=accept-new — 首次連接新主機時會自動將金鑰加入 ~/.ssh/known_hosts。之後主機金鑰若變更會被拒絕，避免 MITM 攻擊。
ExitOnForwardFailure=yes — 若無法建立埠轉發會立即失敗，而非無聲連線。
必須使用 SSH 金鑰驗證——不支援密碼登入。

---

疑難排解

啟動時連線錯誤或空白頁面
Hermes Web UI 未運行。如果使用直接模式，啟動它：
cd ~/hermes-webui-public && bash start.sh
然後打開偏好設定並點擊 Save & Reconnect，或直接重啟應用程式。

測試連線顯示 "Unreachable"
直接模式：Hermes Web UI 未在設定的 URL 運行。請檢查 URL 與埠號。
SSH 模式：Hermes Web UI 未在遠端伺服器運行，或 SSH 金鑰驗證未設定。先用 Terminal 測試 ssh user@your-server。

SSH 通道立即顯示 "disconnected"
ssh user@your-server 必須在 Terminal 可免密碼連線。若需密碼，先設定金鑰驗證。
遠端埠必須符合 Hermes Web UI 實際監聽埠（預設：8787）。

語音輸入無法使用
macOS 需明確授權。首次啟動有系統彈窗——若拒絕：
打開 系統設定 → 隱私與安全性 → 麥克風
啟用 Hermes Agent
重啟應用程式

Gatekeeper 阻擋應用程式
版本低於 v1.0.4。請下載最新版本——v1.0.4 之後首次開啟不會提示警告。

從原始碼建置後圖示模糊
執行 killall Dock 重新整理圖示快取。

---

架構

Sources/HermesAgent/
├── main.swift                        — 入口點，訊號處理
├── AppDelegate.swift                 — App 生命週期、選單、Sparkle 更新
├── BrowserWindowController.swift     — WKWebView 視窗、剪貼簿、通知、錯誤頁面
├── TunnelManager.swift               — SSH 流程管理、埠探測、監控
├── PreferencesWindowController.swift — 設定 UI、連線測試
└── SplashWindowController.swift      — 啟動畫面

檔案
用途
Package.swift
Swift Package Manager 設定檔（macOS 12+、Swift 5.9+）
build.sh
建置腳本——編譯、打包 .app、轉換圖示、安裝到 /Applications
scripts/release.sh
發行輔助——先推 main 再推 tag 以確保 CI 觸發
Tests/HermesAgentTests/
單元測試——使用 swift test 執行

---

發行

從乾淨的 main 建立新簽署並公證的版本：

scripts/release.sh v1.0.9

腳本會先推送 main，然後再單獨推送 tag。如果同時推送 commit 與 tag（如 git push --follow-tags），GitHub 偶爾會漏掉事件，CI 不會觸發。分開推送可避免此問題。

若兩分鐘內 workflow 未觸發，可手動啟動：Actions → Build and Release macOS App → Run workflow → 輸入 tag。

---

鳴謝

基於 @redsparklabs 在 hermes-webui PR #544 中的原始 macOS 原生應用程式貢獻。

授權

與 hermes-webui 相同授權。