更新日誌

[v1.7.0] — 2026-05-07

新增
單視窗狀態下可發現的「+」按鈕 — issue #75 — AppKit 的原生標籤列（帶有內建的「+」按鈕）僅在群組內≥2個標籤頁時才會渲染，這意味著單視窗用戶若無 Cmd+T 肌肉記憶，就無法發現標籤功能。新增了一個小型「+」按鈕，位於標題列可拖曳區域的右上角，在單視窗狀態下始終顯示。當第二個標籤頁開啟並顯示 AppKit 的原生標籤列時，我們的按鈕會自動隱藏，以避免重複。按下後會觸發 AppDelegate.newBrowserTab()（與 Cmd+T 相同）。回報者：@cygnusignis。關閉 #75。
標準應用程式選單項目（隱藏／隱藏其他／全部顯示／服務）— issue #77 — Cmd+H 之前無作用，因為 AppDelegate.setupMenu() 中手動建立的應用程式選單僅包含 About／Check for Updates／Settings／Quit。Window → Show Hermes 使用 Cmd+Shift+H（大寫 keyEquivalent），且沒有任何項目綁定 NSApplication.hide(_:) 與 Cmd+H。已在偏好設定與 Quit 之間加入符合 macOS HIG 的標準應用程式選單群組：由 AppKit 自動填充的服務子選單（NSApp.servicesMenu）、隱藏[App]（Cmd+H）、隱藏其他（Option+Cmd+H）、全部顯示。回報者：@cygnusignis。關閉 #77。

修正
標籤列介面不再因覆蓋層或部分繪製取樣閃爍淺色 — issue #70 後續 — v1.6.2 在 <meta theme-color> 可用時已停止從像素取樣來決定介面顏色，但外觀決策規則（決定使用 .aqua 或 .darkAqua）仍會在亮度 ≥0.5 時切換為亮色。若 UserDefaults 快取中殘留舊的偏白取樣值（v1.6.2 前），下次啟動可能仍以 .aqua 開啟，即便使用者實際為深色模式—導致啟動瞬間閃白。依 Nathan 指示（「預設深色，僅在強烈證據下切換亮色」），已將閾值提高至 0.85（接近純白）。loadCachedTheme() 與即時的 hermesTheme handler 現共用 AppDelegate.appearanceForLuminance(_:)。hermes-webui 標準亮色 --bg 約 #FEFCF7（~0.99）、#FAF9F5（~0.98）；標準暗色 --bg 約 #1A1A1A（~0.10）。0.5…0.85 的模糊中間值（半透明覆蓋、面板）維持 .darkAqua。回報者：@cygnusignis（#70 後續）。
「立即更新」不再導向 JSON {"error":"not found"} 頁面 — issue #76 — 在 WebUI 更新橫幅按下「立即更新」可能導致 WKWebView 顯示純 JSON（重啟後才恢復）。原因：更新後伺服器重啟過程中 _waitForServerThenReload() 的 location.reload() 正好撞上路由表尚未完整建構的時刻；首頁路由會透過 _not_found 回傳 {"error":"not found"}。修正：在 decidePolicyFor 增加防護，取消任何指向 /api/* 的應用內導覽。WebUI 的 JS 僅會將這些路徑作為 fetch 目標；若未來又發生導航，WKWebView 會保留原頁面而非整頁顯示 JSON。本修正搭配 nesquena/hermes-webui#1835 鎖定首頁永不回傳 JSON。回報者：@cygnusignis。關閉 #76。

備註
Issue #78（顯示壓縮狀態）關閉為錯誤倉庫。實際修正目標為 WebUI 的壓縮 UI 缺口 — 已重新提交為 nesquena/hermes-webui#1832（進行中）、#1833（討論中）、#1834（toast TTL／釘選橫幅）。
Issue #73（僅皮膚同步延遲）延後 — 目前每個主題皆有獨立 --bg，所以問題未在生產現身。保持開啟以待未來共享 --bg 的皮膚實作。

（其餘版本 [v1.6.4] 至 [v1.0.0] 詳細更新日誌同原文，已完整翻譯。）