CLAUDE.md — hermes-swift-mac

> 在修改任何程式碼前請先閱讀此文件。這是一個使用 WKWebView 封裝 hermes-webui 的 macOS 原生應用程式。

---

專案簡介

這是一款 macOS 系統選單列原生應用程式，透過 WKWebView 載入 hermes-webui。無沙盒（臨時簽署）。使用 Sparkle 2 進行自動更新。可選擇透過 TunnelManager.swift 支援 SSH 隧道。目標 macOS 12+，以通用 arm64+x86_64 DMG 發布。

語言： Swift 5.9  
建置： bash build.sh（本地）或建立 v* 標籤觸發 CI 產生 DMG  
測試： swift test（執行 Tests/HermesAgentTests/ValidationTests.swift）  
CI： .github/workflows/build-release.yml（發布）、test.yml（PR 測試）  
最新版本： git tag --sort=-v:refname | head -1

---

專案結構

Sources/HermesAgent/
  AppDelegate.swift               # App 生命週期、選單列初始化、Sparkle 更新
  BrowserWindowController.swift   # 主要 WKWebView 視窗 + WKNavigationDelegate
  PreferencesWindowController.swift  # 目標 URL 與設定
  SplashWindowController.swift    # 載入畫面
  TunnelManager.swift             # 可選 SSH 隧道（基於 Process）
  main.swift                      # 入口

Tests/HermesAgentTests/
  ValidationTests.swift           # URL 驗證與偏好設定解析測試

build.sh          # 本地建置 — 產生 Info.plist、臨時簽署
Package.swift     # SPM — 僅依賴 Sparkle 2
Entitlements.plist  # 無沙盒 — 臨時簽署
appcast.xml       # Sparkle 更新來源（GitHub Pages）
CHANGELOG.md      # 每版一條紀錄 — 標籤前更新

---

規則

不要直接推送到 main
所有修改需透過命名分支 + PR。測試必須通過。必須更新 CHANGELOG。

強制使用 SSH 推送
eval $(ssh-agent -s) && ssh-add ~/.ssh/id_ed25519
git push origin <branch>
# 或推送標籤：
git push origin v1.x.y
HTTPS token 推送會失敗，必須使用 ssh-agent。

Plist 鍵值一致性 — build.sh 與 CI workflow 必須相符
每個 Info.plist 鍵值必須同時存在於：
build.sh heredoc（約第 60-100 行）
.github/workflows/build-release.yml 的 PlistBuddy 區塊

若僅更新其中一處，可能導致本地建置可用但 CI DMG 缺少鍵值，或反之。務必每次同步檢查。

標籤前必須更新 CHANGELOG
在推送發布標籤前更新 CHANGELOG.md。CI 會從標籤自動建立 GitHub Release，若紀錄過期，發布說明會錯誤。

---

WKWebView 規則 — 修改 BrowserWindowController.swift 前必讀

ATS（App Transport Security）
http://localhost 自動免 ATS 限制。其他 http:// URL（Tailscale IP 100.x.x.x、區域網路 IP、主機名）預設會被阻擋。

修正方式（已在 issue #25）：
<!-- 需同時在 build.sh 與 CI workflow 的 Info.plist heredoc 中加入 -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <false/>
    <key>NSAllowsArbitraryLoadsInWebContent</key>
    <true/>
</dict>

Navigation delegate — 兩個失敗回呼都必須實作
// 必須同時實作，缺一會導致無提示失敗
func webView(_ webView: WKWebView, didFailProvisionalNavigation navigation: WKNavigation!, withError error: Error) {
    handleNavigationFailure(error)
}
func webView(_ webView: WKWebView, didFail navigation: WKNavigation!, withError error: Error) {
    handleNavigationFailure(error)
}

private func handleNavigationFailure(_ error: Error) {
    let code = (error as NSError).code
    if code == NSURLErrorCancelled { return }  // -999: 過濾取消導覽
    // 根據錯誤碼顯示提示訊息：
    // -1022: ATS 阻擋 HTTP → 建議使用 SSH 隧道或 NSAllowsArbitraryLoadsInWebContent
    // -1004: 無法連線 → 伺服器未運行或 URL 錯誤
    // -1003: DNS 解析失敗
}

禁止導覽到 file:// URL
僅允許 http:// 與 https://。WKWebView 在無沙盒下可廣泛存取檔案，因此需限制僅允許網路協議。

---

Sparkle 2 — 修改自動更新前必讀

Info.plist 的 SUPublicEDKey 必須與簽署 appcast.xml 的私鑰匹配
Info.plist 的 SUFeedURL：https://hermes-webui.github.io/hermes-swift-mac/appcast.xml
CI 簽署順序必須由內而外：XPC 服務 → Sparkle.framework → App Bundle
重新簽署 XPC 服務時需使用 --preserve-metadata=entitlements
DMG 不可先簽署再公證，必須先公證 App Bundle，再打包 DMG
本地建置會移除 XPC 服務（build.sh 會刪除）

---

SSH 隧道 — 修改 TunnelManager.swift 前必讀

// 正確：陣列參數（避免 Shell 注入）
process.arguments = ["-N", "-L", "8787:127.0.0.1:8787", "user@host"]

// 錯誤：字串指令（存在注入風險）
process.arguments = ["-c", "ssh -N -L 8787:127.0.0.1:8787 \(host)"]

必須在隧道參數中使用 StrictHostKeyChecking=yes 與 ExitOnForwardFailure=yes。
使用 DispatchSource 處理 Process 訊號，而非輪詢。

---

已知問題／待辦事項

#25 — 非 localhost URL（如 Tailscale IP）因 ATS + 伺服器綁定而靜默失敗。
  修正方法：在 Info.plist（build.sh 與 CI）加入 NSAllowsArbitraryLoadsInWebContent，
  並於 BrowserWindowController 實作 didFailProvisionalNavigation。
  伺服器端修正：.env 中設置 HERMES_WEBUI_HOST=0.0.0.0

---

Opus 導師 — 第二意見顧問

當不確定 Swift API、WKWebView 行為、ATS 設定或 Sparkle 簽署時，請先詢問 opus：

# 輸入實際程式碼與 issue/PR 內文可獲得最高品質建議
{ cat Sources/HermesAgent/BrowserWindowController.swift; cat build.sh; } \
  | claude --model claude-opus-4-7 --thinking enabled \
  --print 'Senior Swift/macOS engineer. [DESCRIBE SITUATION].
Review for: ATS config, WKNavigationDelegate completeness, Info.plist key parity, security.
Provide exact Swift code fixes.'

---

常見陷阱

gh pr view 已失效 — 請使用 gh api repos/hermes-webui/hermes-swift-mac/pulls/NNN
逐條閱讀所有 PR 評論 — 第一條評論可能為「仍在進行中」，第二條才是完成訊號
Swift 無法在 Linux 使用 — 所有建置必須在 macOS runner 執行
swift test 在 Linux 會失敗 — 請在 CI（macOS 14 runner）或 Mac 執行
CI 產物的 Sparkle 路徑 — 打包前使用 find 確認框架路徑
通用二進位 — 必須產生 arm64+x86_64。使用 swift build -c release
  （SPM 在 Apple Silicon 且工具鏈正確時會自動處理通用建置）
