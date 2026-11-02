# Changelog

woowtech smart desktop releases. The app's What's new sheet reads this file; each release gets a `## <version> - <date>` heading.

## 0.9.0 - 2026-11-02

網頁版上線：配對連結和 QR Code 現在打開 smart.woowtech.io，在瀏覽器裡就能連到你的電腦。The web app is live: pairing links and QR codes now open smart.woowtech.io, so you can reach your computer from a browser.

### 新功能 / New

- 配對連結和 QR Code 現在打開網頁版 `https://smart.woowtech.io`，在瀏覽器裡就能連到你的電腦。原本是叫起 App；App 內掃描、貼上，以及舊的 QR Code 照樣能用。
  Pairing links and QR codes now open the web app at `https://smart.woowtech.io`, so you can reach your computer from a browser. They used to launch the app; scanning or pasting in the app, and old QR codes, still work.
- 網頁版只有分頁開著時能用瀏覽器通知；推播通知要用手機 App。
  The web app shows browser notifications only while its tab is open; push notifications need the mobile app.

### 安全 / Security

- 配對連結就是密碼：拿到連結的人就能連上你的 daemon。網頁版在加入主機前會顯示確認畫面，請和桌面版「配對裝置」顯示的指紋比對。
  A pairing link is a password: anyone who has it can connect to your daemon. Before the web app saves a host it shows a confirmation screen; compare the fingerprint with the one under "Pair device" in the desktop app.

### 變更 / Changes

- 新安裝的 daemon 預設放行 `https://smart.woowtech.io` 直連。直連要等你在桌面瀏覽器的歡迎畫面按「連到這台電腦」才會開始，這時 Chrome 和 Firefox 預期會問你要不要允許存取本機；Safari 與 iOS 的瀏覽器預期會擋掉連到本機的 `ws://`（沒有加密的直接連線），那就只能走中繼。（這些瀏覽器的行為是依瀏覽器的原始碼和說明文件整理的，我們只在桌面版 Chrome 實測過。）不想要直連的話：`woowtech-smart daemon config set daemon.cors.allowedOrigins '[]'`，再執行 `woowtech-smart daemon restart`。只改設定只擋新的連線，已經開著的直連要重啟才會斷。這只會關掉直連：已經用連結配對過的瀏覽器，仍然可以經中繼連上；要完全切斷它，要輪替配對金鑰（做法見下面「撤銷所有配對」）。
  A newly created daemon allows `https://smart.woowtech.io` to connect directly. A direct connection only starts after you press "Connect to this computer" on the welcome screen of a desktop browser; Chrome and Firefox are then expected to ask whether to allow access to your computer, while Safari and iOS browsers are expected to block `ws://` (an unencrypted direct connection) to your computer, so they can only use the relay. (This browser behaviour comes from the browsers' source code and documentation; we have only tested desktop Chrome.) To turn direct connections off, run `woowtech-smart daemon config set daemon.cors.allowedOrigins '[]'`, then `woowtech-smart daemon restart`. Changing the setting only blocks new connections; direct connections that are already open end after a restart. This turns off only the direct path: a browser you already paired through a link can still connect through the relay. To cut that off too, rotate the pairing key (see "Revoke every pairing" below).
- Hub（上游 Paseo 專案提供的選用線上服務）連線預設關閉。已經用 `woowtech-smart hub connect`（或 `hub init`）把 daemon 接上 Hub 的人，升級後 daemon 不會自動續連（daemon 的記錄檔 `daemon.log` 會有一行警告）；要繼續用，先執行 `woowtech-smart daemon config set daemon.hub.enabled true`，再執行 `woowtech-smart daemon restart`。
  Hub connections (Hub is an optional online service run by the upstream Paseo project) are now off by default. If you connected this daemon to a Hub with `woowtech-smart hub connect` (or `hub init`), it will not reconnect after the update (`daemon.log` carries one warning). To keep using it, run `woowtech-smart daemon config set daemon.hub.enabled true`, then `woowtech-smart daemon restart`.
- 上面的指令要先安裝才打得出來：在桌面版的「設定 → 整合 → 指令列」按「安裝」，再開一個新的終端機視窗。
  The commands above only work once the command-line tool is installed: in the desktop app open Settings → Integrations → Command line and press Install, then open a new terminal window.

### 修正 / Fixes

- Intel 處理器的 Mac：桌面版現在能編譯本機外掛了。0.8.1 的 Intel 版裝錯了一個元件（給 Apple 晶片用的 esbuild），外掛編譯會失敗。
  Intel Macs: the desktop app can now compile local plugins. 0.8.1's Intel build shipped a component built for Apple silicon (esbuild), so compiling a plugin failed.

### 已經安裝過的 daemon / Daemons you already installed

- 升級前就建立的 daemon，資料夾（`~/.woowtech-smart`）裡的設定不會自動更新，直連名單仍是空的，所以網頁版只能經中繼連上它（中繼要是開著的）；它的配對連結本來就會打開網頁版。要開直連，合併原本的清單，不要整份蓋掉（設定馬上生效，不用重啟）：先用 `woowtech-smart daemon config get daemon.cors.allowedOrigins` 看原本的清單，再執行 `woowtech-smart daemon config set daemon.cors.allowedOrigins '["https://smart.woowtech.io"]'`（原本有其他來源就一起列進去）。
  For a daemon created before this version, the settings in its data folder (`~/.woowtech-smart`) are not updated automatically: its direct-connection list stays empty, so the web app can reach it only through the relay (the relay must be on). Its pairing links already open the web app. To allow direct connections, merge with the current list instead of overwriting it (the change takes effect at once, no restart needed): check the list with `woowtech-smart daemon config get daemon.cors.allowedOrigins`, then run `woowtech-smart daemon config set daemon.cors.allowedOrigins '["https://smart.woowtech.io"]'` (keep any origins already listed).

### 撤銷所有配對 / Revoke every pairing

- 配對連結沒有期限，要讓所有連結和 QR Code 失效，只能輪替配對金鑰。在你的電腦上依序執行：`woowtech-smart daemon stop`（桌面版：結束 App）；`mv ~/.woowtech-smart/daemon-keypair.json ~/.woowtech-smart/daemon-keypair.json.revoked-$(date +%Y%m%d)`（把舊金鑰改名留著，不要刪）；`woowtech-smart daemon start`（桌面版：重新打開 App，會產生新的金鑰）；`woowtech-smart daemon pair`（或在桌面版按「配對裝置」）拿新的連結和 QR Code；有設直連密碼的話，用 `woowtech-smart daemon set-password` 一起換掉。
  Pairing links never expire; rotating the pairing key is the only way to make every link and QR code stop working. On your computer run, in order: `woowtech-smart daemon stop` (desktop app: quit the app); `mv ~/.woowtech-smart/daemon-keypair.json ~/.woowtech-smart/daemon-keypair.json.revoked-$(date +%Y%m%d)` (rename the old key and keep it; do not delete it); `woowtech-smart daemon start` (desktop app: open the app again; it creates a new key); `woowtech-smart daemon pair` (or press Pair device in the desktop app) for a new link and QR code; if you set a direct-connection password, change it too with `woowtech-smart daemon set-password`.
- 之後每支手機、每個瀏覽器都要先在主機設定裡按「移除主機」，再用新的連結或 QR Code 配對；沒有先移除的話，確認畫面會擋下新連結，因為金鑰不一樣。daemon 不在預設資料夾 `~/.woowtech-smart` 的話，每個指令都加 `--home "<資料夾路徑>"`。
  Afterwards, on every phone and browser, first press Remove host in the host's settings, then pair again with the new link or QR code; without removing it first, the confirmation screen blocks the new link because the key is different. If your daemon is not in the default folder `~/.woowtech-smart`, add `--home "<folder>"` to every command.

### 系統需求 / Requirements

- macOS 13 以上，Apple 晶片或 Intel。
  macOS 13 or later, Apple silicon or Intel.

## 0.8.1 - 2026-10-16

woowtech smart（渥屋智能）Mac 桌面版第一個公開版本。The first public release of the woowtech smart desktop app for Mac.

### 新功能 / New

- 在 Mac 上啟動和管理 AI 寫程式 agent：Claude Code、Codex、GitHub Copilot CLI、OpenCode、Pi。agent 要另外安裝，用你自己的帳號登入。
  Run and supervise AI coding agents on your Mac: Claude Code, Codex, GitHub Copilot CLI, OpenCode and Pi. Install the agents separately and sign in with your own accounts.
- 手機配對：用手機 App「渥屋智能」掃 QR code，在外面也能看進度、核准或拒絕修改、回覆 agent。連線端對端加密，程式碼留在你的電腦上。
  Pair your phone: scan the QR code with the 渥屋智能 (woowtech smart) mobile app to follow progress, approve or reject changes and answer your agents from anywhere. End-to-end encrypted; your code stays on your computer.
- 通知：agent 需要你回應或完成時通知你；電腦上的通知 3 分鐘沒處理，會改推到手機。通知不帶你的對話或程式內容。
  Notifications when an agent needs you or finishes; if a notification on the Mac is not handled within 3 minutes it goes to your phone. Notifications never include your conversations or code.
- 每則 AI 回覆都可以「回報」不當內容。
  Report any AI reply that looks wrong or harmful.
- 繁體中文與英文介面。
  Traditional Chinese and English interface.

### 系統需求 / Requirements

- macOS 13 以上，Apple 晶片或 Intel。
  macOS 13 or later, Apple silicon or Intel.
