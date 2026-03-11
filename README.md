[![NuGet](https://img.shields.io/nuget/v/LibGGPK3.LibBundledGGPK3)](https://www.nuget.org/packages?q=LibGGPK3)

## 簡介
這是一套跨平台函式庫，專門處理《流亡黯道》（Path of Exile）的 `Content.ggpk`。  
原始版本重寫自：https://github.com/aianlinb/LibGGPK2

## 關於此 Fork（jakeuj）
本庫定期以 `git fetch upstream && git rebase upstream/main` 方式追蹤 [aianlinb/LibGGPK3](https://github.com/aianlinb/LibGGPK3)。這個 fork 著重於台灣玩家與 macOS ARM 使用者，與上游的主要差異如下：
- **Target Framework 與版本**：預設 `net10.0`，並採 `2.7.5-fork.1`（`AssemblyVersion/FileVersion 2.7.5.1`）版號，以便在保持與上游 2.7.5 資源同步的同時，輸出 macOS ARM NativeAOT 執行檔。
- **在地化強化**：PoeChinese3 內建思源黑體（Source Han Sans TW），更新版權資訊，並提升 macOS .app 內自動偵測 Content.ggpk 的行為。
- **NativeAOT 發佈管線**：`Directory.Build.props` 內含 AOT 最佳化設定（停用 debug/doc 檔、Strip 符號、連結 Oodle 靜態庫），同時僅在非 AOT 執行檔才複製動態庫，縮小包體。
- **macOS 體驗修復**：`VisualGGPK3`／`VPatchGGPK3` 會在建置時自動複製 `Icon.icns`，避免 Eto.Mac 的 “Icon.icns does not exist” 警告並產生帶有圖示的 `.app`。
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
