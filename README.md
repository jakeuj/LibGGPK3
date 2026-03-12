[![NuGet](https://img.shields.io/nuget/v/LibGGPK3.LibBundledGGPK3)](https://www.nuget.org/packages?q=LibGGPK3)

## 簡介
這是一套跨平台函式庫，專門處理《流亡黯道》（Path of Exile）的 `Content.ggpk`。  
原始版本重寫自：https://github.com/aianlinb/LibGGPK2

繁體中文化入口網站（PoeChinese）：https://poechinese.jakeuj.com/

### 贊助 / Sponsorship
This project is a fork of [aianlinb/LibGGPK3](https://github.com/aianlinb/LibGGPK3)。  
如果你想支持原作者（aianlinb），請透過 PayPal 贊助：<https://paypal.me/aianlinb>。  
If you want to support the original author, please sponsor them here: <https://paypal.me/aianlinb>.

如果你想支持本 Fork（jakeuj）的長期維護，歡迎透過 PayPal 贊助：<https://paypal.me/jakeuj>。  
If you want to support this fork's maintenance, you can sponsor me here: <https://paypal.me/jakeuj>.

## 關於此 Fork（jakeuj）
本庫定期以 `git fetch upstream && git rebase upstream/main` 方式追蹤 [aianlinb/LibGGPK3](https://github.com/aianlinb/LibGGPK3)。這個 fork 著重於台灣玩家與 macOS ARM 使用者，與上游的主要差異如下：
- **Target Framework 與版本**：預設 `net10.0`，並採 `2.7.5-fork.1`（`AssemblyVersion/FileVersion 2.7.5.1`）版號，以便在保持與上游 2.7.5 資源同步的同時，輸出 macOS ARM NativeAOT 執行檔。
- **在地化強化**：PoeChinese3 內建思源黑體（Source Han Sans TW），更新版權資訊，並提升 macOS .app 內自動偵測 Content.ggpk 的行為；上游需要自行把任意中文字型改名為 `Font.ttf` 放在執行檔旁，本 fork 不再需要手動拷貝字型。
- **NativeAOT 發佈管線**：`Directory.Build.props` 內含 AOT 最佳化設定（停用 debug/doc 檔、Strip 符號、連結 Oodle 靜態庫），同時僅在非 AOT 執行檔才複製動態庫，縮小包體。
- **macOS 體驗修復**：`VisualGGPK3`／`VPatchGGPK3` 會在建置時自動複製 `Icon.icns`，避免 Eto.Mac 的 “Icon.icns does not exist” 警告並產生帶有圖示的 `.app`。
- **PoeChinese3 互動升級**：CLI 與 `.app` 會記住上次成功中文化的 GGPK 路徑（儲存於 `%APPDATA%/PoeChinese3/lastpath.txt`，macOS 為 `~/Library/Application Support/PoeChinese3/lastpath.txt`），並新增 `--use-default`/`-d` 旗標，一鍵載入記憶路徑或預設 Windows 安裝路徑。
- **CI/CD 與釋出**：Fork 專屬 GitHub Actions、release drafter 與 README/系統需求說明（可參考 2025 年 12 月的提交紀錄）都針對台灣社群發佈簽署二進位檔。

### Fork 環境初始化與建置
1. 取得原始碼並同步子模組（OodleUE 的 SDK/Bundles 必須存在，否則無法建置/發佈）：
   ```bash
   git clone https://github.com/jakeuj/LibGGPK3.git
   cd LibGGPK3
   git submodule update --init --recursive External/OodleUE
   ```
2. 請安裝 .NET 8/9/10 SDK；本 fork 預設 `TargetFramework=net10.0`，但其他專案仍維持上游多目標設定。
3. 編譯所有函式庫與工具：
   ```bash
   dotnet build LibGGPK3.sln
   ```
4. 若要產生 NativeAOT 範例（PoeChinese3、GGPKFastCompact3 等），執行：
   ```bash
   dotnet publish -c Release -r <RID> /p:PublishAot=true
   ```
5. 遇到 macOS bundle 缺圖示警告時，確認 `Examples/Icon.icns` 是否已被複製到 `bin/Debug/net10.0/<App>.app/Contents/Resources/`。

> 想了解本 repo 如何維持同步？可參考 `.agent/skills/libggpk3-maintenance/SKILL.md`，裡面整理了 rebase、版本調整與 Icon 修補的標準流程。

### PoeChinese3 路徑記憶與 `--use-default`
自 2026-03-11 的 `b92df07e75dcd11de836346443457a3e0a1bfa74` 提交起，PoeChinese3 的 CLI/.app 具備以下行為，減少手動輸入 GGPK 路徑的頻率：

- 每次成功中文化後，會把實際使用的 GGPK 路徑寫入 `%APPDATA%/PoeChinese3/lastpath.txt`（macOS 為 `~/Library/Application Support/PoeChinese3/lastpath.txt`）。下次啟動時會優先使用此路徑並驗證檔案存在。
- 新增 `--use-default`（或縮寫 `-d`）旗標，可直接強制使用「記憶路徑 → 自動偵測 → Windows 預設 `C:\Program Files (x86)\Grinding Gear Games\Path of Exile\Content.ggpk`」的順序，跳過互動式輸入。
- 沒有旗標時，互動流程仍會嘗試以下預設：執行檔所在資料夾、`.app` 同層的 `Content.ggpk`、`PoE.app`（Wineskin/CrossOver）的 bundle、目前工作目錄，再退回到記憶路徑與提示輸入。

範例：直接沿用上次路徑或預設值。

```bash
./PoeChinese3 --use-default
# 或
./PoeChinese3 -d
```

> 既然 CLI/.app 會記住上次成功的 GGPK 路徑並可用 `--use-default` 套用預設，就不再建議透過桌面腳本硬寫固定路徑；請直接使用旗標或回憶檔案即可。

### 內建字型（免改 `Font.ttf`）

上游說明文件會要求 macOS 使用者先找任意中文字型，改名為 `Font.ttf` 後放在 `PoeChinese_osx-x64` 或 CLI 執行檔旁。本 fork 直接把 Source Han Sans TW（思源黑體台灣版）打包進 PoeChinese3，因此：

- 完整下載這個 fork 的 `.app` / CLI 可立即顯示繁體中文，不需再手動複製或改名字型檔。
- 若想換成自備字型，只要把 `Font.ttc`／`Font.ttf`／`Font.otf` 放在執行檔同層即可覆蓋，詳細流程見 `Examples/PoeChinese3/README.md` 的「自訂字型」章節。

也因此 README 與 GitHub Pages 皆標示出「fork 版內建字型，上游仍需自行準備」的差異，方便玩家判斷。

## 注意事項
- 未經授權禁止修改、再散佈或商業使用；如需合作請先聯絡作者。
- 此儲存庫中的專案可能非完全 thread-safe，若多執行緒同時操作同一個 ggpk，請格外小心（建議自行加鎖或分批處理）。
- 版本更新不保證與舊版完全相容；升級前請先檢視 commit 歷史並驗證自己的專案是否仍可正常運作。

# LibGGPK3
核心函式庫，負責讀寫 `Content.ggpk`。

# LibBundle3
處理 `Bundles2` 目錄下的 `*.bundle.bin`，主要給 Steam/Epic 版本使用。

# LibBundledGGPK3
整合 LibGGPK3 與 LibBundle3，可同時處理 `Content.ggpk` 以及其中內嵌的 bundle，適合 Standalone Client。

# Examples
示範如何運用上述函式庫完成常見任務。  
***VisualGGPK3 仍在開發中，僅供測試。***
