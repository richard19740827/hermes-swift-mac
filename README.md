# Hermes Agent for macOS

一個為 [Hermes Web UI](https://github.com/nesquena/hermes-webui) 量身打造的原生 macOS 桌面應用程式。使用 Swift 與 WKWebView 構建 —— 非 Electron 架構，除了 Xcode 命令列工具外，無任何額外依賴。由 [@redsparklabs](https://github.com/redsparklabs) 建立。

<img width="1470" height="922" alt="Hermes Agent 截圖" src="https://github.com/user-attachments/assets/8e704e62-c736-4827-ba50-c41a21d9922f" />

---

## 準備工作

Hermes Agent 是 [Hermes Web UI](https://github.com/nesquena/hermes-webui) 的原生視窗外殼。App 本身只是一個包裝 —— 它需要後端正在運行的 Hermes Web UI 才能發揮作用。如果後端未啟動，您在啟動 App 時會看到連線錯誤。

**必要條件：** 在您的 Mac 或遠端伺服器上運行的 Hermes Web UI。
**遠端伺服器選配：** 該伺服器需配置好 SSH 金鑰驗證。
**macOS 版本：** 12 (Monterey) 或更高版本。

---

## 安裝設定

請根據您的目前情況選擇對應的路徑。

### 路徑 1 — 第一次使用 Hermes：本地安裝所有組件

首先，讓 Hermes Web UI 在您的 Mac 上跑起來：

```bash
git clone https://github.com/nesquena/hermes-webui.git ~/hermes-webui-public
cd ~/hermes-webui-public
bash start.sh
```

這會在 `http://localhost:8787` 啟動 Hermes 伺服器。請參考 [Hermes Web UI README](https://github.com/nesquena/hermes-webui#readme) 在首次運行的引導過程中設定您的 API 金鑰。

接著安裝 Hermes Agent：

1. 從 [版本發佈 (Releases)](https://github.com/hermes-webui/hermes-swift-mac/releases) 下載最新的 `Hermes-Agent-vX.X.X.dmg`。
2. 打開 DMG 檔案並將 **Hermes Agent** 拖移到您的「應用程式 (Applications)」資料夾。
3. 啟動 App —— 預設會連線到 `http://localhost:8787`，也就是您剛才啟動 Hermes Web UI 的位置。

無需額外設定，開箱即用。

---

### 路徑 2 — 已經在本地運行 Hermes Web UI

如果您的 Mac 已經在運行 Hermes Web UI：

1. 從 [Releases](https://github.com/hermes-webui/hermes-swift-mac/releases) 下載最新的 DMG。
2. 將 **Hermes Agent** 拖移到應用程式資料夾並啟動。

預設的目標 URL 是 `http://localhost:8787`。如果您在不同的埠號（Port）運行 Hermes，請打開 **偏好設定 (Preferences)** (⌘,)，更新目標 URL，然後點擊 **Save & Reconnect (儲存並重新連線)**。

您可以在儲存前透過 **Test Connection (測試連線)** 按鈕來驗證連線。

---

### 路徑 3 — Hermes Web UI 運行在遠端伺服器

如果您的 Hermes Web UI 運行在需要透過 SSH 訪問的伺服器上：

**開始之前：** 請確保該伺服器的 SSH 金鑰驗證運作正常 —— 在終端機輸入 `ssh user@your-server` 應該能直接連線而無需輸入密碼。

1. 從 [版本發佈 (Releases)](https://github.com/hermes-webui/hermes-swift-mac/releases) 下載最新的 DMG。
2. 將 **Hermes Agent** 拖移到「應用程式 (Applications)」資料夾並啟動。
3. 打開 **偏好設定 (Preferences)** (⌘,)。
4. 將 **連線模式 (Connection Mode)** 設定為 **SSH Tunnel (SSH 隧道)**。
5. 填寫以下資訊：
   - **Username** — 您在遠端伺服器上的 SSH 使用者名稱。
   - **Host** — 伺服器的主機名稱或 IP 位址。
   - **Local Port** — 您 Mac 上的本地埠號（預設：8787）。
   - **Remote Port** — 遠端伺服器上 Hermes 運行的埠號（預設：8787）。
6. 點擊 **Test Connection (測試連線)** 進行驗證，然後點擊 **Save & Reconnect (儲存並重新連線)**。

App 在啟動時會自動開啟 SSH 隧道，監控連線狀態，並在退出時乾淨地關閉它。視窗底部的狀態列會顯示隧道狀態，並提供一鍵「重新連線」按鈕。

---

## 安裝選項

### 下載 DMG 檔案 (推薦)

前往 [Releases](https://github.com/hermes-webui/hermes-swift-mac/releases)，下載最新的 DMG，打開後將 **Hermes Agent** 拖移到您的「應用程式 (Applications)」資料夾。此 App 已使用開發者 ID 憑證簽署並通過 Apple 公證 —— 啟動時不會出現 Gatekeeper 安全性警告。

### 從原始碼編譯

此方法需要安裝 Xcode 命令列工具 (Command Line Tools)：

```bash
xcode-select --install   # if not already installed

git clone https://github.com/hermes-webui/hermes-swift-mac.git
cd hermes-swift-mac
./build.sh
```

這會編譯 App、綁定圖標並將其安裝到 /Applications/Hermes Agent.app。

---

## 功能特色

- **原生 macOS 應用程式** — 擁有 Dock 圖示、標準選單列，使用體驗就像任何 Mac 內建 App。
- **WKWebView 視窗載入** — 在獨立視窗執行 Hermes Web UI（無需瀏覽器分頁，非 Electron 架構）。
- **靈活連線模式** — 支援本地 Hermes 實例的「直接模式」，以及遠端伺服器的「SSH 隧道模式」。
- **剪貼簿整合** — 支援直接將文字與圖片 (⌘V) 粘貼至對話框中。
- **文件上傳** — 可透過迴紋針按鈕直接上傳檔案。
- **偏好設定視窗 (⌘,)** — 附帶「測試連線」按鈕，儲存設定前可先驗證。
- **即時狀態列** — 顯示 SSH 隧道即時狀態，並支援一鍵重新連線。
- **macOS 系統通知** — 當視窗在背景運行且 AI 回覆完成時，會發送通知提醒。
- **語音輸入** — 首次使用會請求麥克風權限，支援語音對話。
- **外部連結處理** — 所有的外部網址都會由 Safari 開啟，不會在 App 內亂跑。
- **自動更新 (Sparkle)** — 啟動時自動檢查新版本，或透過選單手動「檢查更新」。
- **已簽署並公證** — 通過 Apple 安全認證，首次啟動不會出現安全性門禁警告。

---

## 系統設定

請開啟 **偏好設定 (Preferences)** (⌘,)：

| 設定項目 | 描述 |
| :--- | :--- |
| **Connection Mode** | Direct (本地) 或 SSH Tunnel (遠端) |
| **Target URL** | 要載入的網址 (預設: `http://localhost:8787`) |
| **Username** | SSH 使用者名稱 (僅限 SSH 模式) |
| **Host** | SSH 伺服器主機名稱或 IP (僅限 SSH 模式) |
| **Local Port** | Mac 上用於隧道的本地埠號 (僅限 SSH 模式) |
| **Remote Port** | 伺服器上 Hermes 監聽的埠號 (僅限 SSH 模式) |

設定會跨啟動持久保存。

---

## 快捷鍵清單

| 快捷鍵 | 動作 |
| :--- | :--- |
| **⌘ ,** | 開啟偏好設定 |
| **⌘ R** | 重新整理 WebUI 頁面 |
| **⌘ W** | 隱藏視窗 (App 仍在背景運行) |
| **⌘ ⇧ H** | **全域快捷鍵**：從任何 App 將 Hermes 喚至最前層 |
| **⌘ + / ⌘ −** | 放大 / 縮小 |
| **⌘ 0** | 重設縮放倍率為實際大小 |

全域快捷鍵 **⌘ ⇧ H** 非常強大 —— 無論您在執行什麼程式，按下它就能立刻找回您的 Hermes。

---

## SSH 安全性

- `StrictHostKeyChecking=accept-new` — 首次連線到新主機時，金鑰會自動加入 `~/.ssh/known_hosts`。之後連線若主機金鑰有變動將被拒絕，防止中間人攻擊。
- `ExitOnForwardFailure=yes` — 如果埠號轉發失敗，隧道會立即關閉，避免在連線不完全的情況下運行。
- **必須使用 SSH 金鑰驗證** — 本 App 不支援密碼輸入方式。

---

## 故障排除

**啟動時出現連線錯誤或空白頁面**
代表 Hermes Web UI 未啟動。如果您是使用本地模式，請先啟動它：
```bash
cd ~/hermes-webui-public && bash start.sh
```
然後開啟偏好設定點擊 **Save & Reconnect**，或直接重新啟動 App。

**測試連線顯示「Unreachable」（無法連線）**
- 直接模式：Hermes Web UI 未在指定的網址執行。請檢查網址與埠號 (Port)。
- SSH 模式：遠端伺服器未執行 Hermes，或 SSH 金鑰驗證未設定。請先在終端機嘗試使用 `ssh user@your-server` 測試。

**SSH 隧道啟動後立即顯示「Disconnected」（已斷開）**
- 檢查在終端機執行 `ssh user@your-server` 是否能在不輸入密碼的情況下連線。如果仍需密碼，請先設定 SSH 金鑰驗證。
- 遠端埠號必須與 Hermes Web UI 實際監聽的埠號一致（預設：8787）。

**語音輸入無法運作**
macOS 需要明確授權。若您在首次啟動時拒絕了權限：
1. 前往 **系統設定 → 隱私權與安全性 → 麥克風**。
2. 啟用 **Hermes Agent** 的權限。
3. 重新啟動 App。

**Gatekeeper 門禁擋下應用程式**
您可能正在使用 v1.0.4 之前的版本。請下載最新的發佈版本 —— v1.0.4 之後的版本均已公證，啟動時不會有任何警示。

**從原始碼編譯後圖示看起來很模糊**
請執行 `killall Dock` 來刷新 macOS 的圖示快取。

---

## 專案架構說明 (Architecture)

```
Sources/HermesAgent/
├── main.swift                        — 程式進入點，訊號處理
├── AppDelegate.swift                 — App 生命週期、選單、Sparkle 更新器
├── BrowserWindowController.swift     — WKWebView 視窗、剪貼簿、通知、錯誤頁面
├── TunnelManager.swift               — SSH 程序管理、埠號偵測、連線監控
├── PreferencesWindowController.swift — 設定介面、連線測試
└── SplashWindowController.swift      — 啟動畫面
```

| 檔案 | 用途 |
| :--- | :--- |
| `Package.swift` | Swift 套件管理器 (SPM) 清單 (支援 macOS 12+, Swift 5.9+) |
| `build.sh` | 編譯腳本 — 負責編譯、打包 .app、轉換圖示並安裝至 /Applications |
| `scripts/release.sh` | 發佈助手 — 分別推送 main 分支與標籤以確保觸發 CI 程序 |
| `Tests/HermesAgentTests/` | 單元測試 — 使用 `swift test` 執行 |

---

## 版本發佈 (Releasing)

若要從乾淨的 `main` 分支發佈一個經過簽署與公證 (Notarized) 的新版本：

```bash
scripts/release.sh v1.0.9
```

該腳本會先推送 main 分支，接著再推送標籤 (Tag)。這非常重要：如果您同時推送提交與標籤（例如使用 git push --follow-tags），GitHub 有時會漏掉其中一個推送事件，導致「編譯與發佈」的工作流 (Workflow) 無法觸發。分開推送可以避免這個問題。
如果推送標籤後兩分鐘內工作流未啟動，請手動觸發：Actions → Build and Release macOS App → Run workflow → 輸入標籤名稱。

## 致謝 (Credits)
本應用程式基於 @redsparklabs 在 hermes-webui PR #544 中所貢獻的原生 macOS 應用程式版本。

## 授權條款 (License)
採用與 hermes-webui 相同的授權條款。
