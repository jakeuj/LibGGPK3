---
layout: default
title: LibGGPK3 Fork / Github Pages
---

# LibGGPK3 Fork（jakeuj）

> 針對台灣玩家與 macOS ARM NativeAOT 發佈優化的 `LibGGPK3` 分支。這裡整理 fork 與上游 [aianlinb/LibGGPK3](https://github.com/aianlinb/LibGGPK3) 的差異、建置步驟，以及維護節奏，方便直接部署到 GitHub Pages。

## Fork ✦ 上游差異總覽

| 項目 | jakeuj Fork | 上游 (aianlinb) |
| --- | --- | --- |
| Target Framework | 預設 `net10.0`，持續追蹤 .NET 最新長期支援版本 | 多目標（`net8.0`/`net9.0`）為主 |
| 版本策略 | `2.7.5-fork.1`（`AssemblyVersion 2.7.5.1`），確保 macOS ARM NativeAOT 二進位可辨識 | `2.7.5` | 
| 發佈管線 | 針對 macOS ARM，Strip 符號、動態複製 Icon、OodleUE 子模組預載 | 以 Windows / x64 為主 |
| 使用者體驗 | PoeChinese3 內建 Source Han Sans TW（免手動改 `Font.ttf`）、macOS 自動偵測 `Content.ggpk` | 需自行準備中文字型並改名為 `Font.ttf` |
| CI/CD | Fork 專屬 GitHub Actions、Release Drafter、簽署 macOS bundle | 預設 CI |

## 最新穩定版

- 版本：`2.7.5-fork.1`
- 更新日期：2025 年 12 月（請於 release drafter 或 tag 查詢實際字串）
- 主要更新：
  - NativeAOT build pipeline 更新，macOS `.app` 自動複製 `Icon.icns`，避免 Eto.Mac 警告。
  - README、系統需求與中文說明同步。
  - `PoeChinese3` 記住上次 GGPK 路徑並新增 `--use-default`/`-d` 旗標（commit `b92df07e75dcd11de836346443457a3e0a1bfa74`，2026-03-11）。
  - 長期追蹤上游 `LibGGPK3` commit，確保資料結構維持最新。

## 快速安裝與同步

```bash
# 取得 fork 並同步上游
 git clone https://github.com/jakeuj/LibGGPK3.git
 cd LibGGPK3
 git remote add upstream https://github.com/aianlinb/LibGGPK3.git
 git fetch upstream
 git rebase upstream/main

# 初始化 OodleUE 子模組（NativeAOT 需要）
 git submodule update --init --recursive External/OodleUE
```

> **注意**：若 rebase 後版本號需要提升，請更新 `Directory.Build.props` 與 `LibGGPK3.csproj` 中的 `VersionPrefix/AssemblyVersion`。維護流程詳見 `.agent/skills/libggpk3-maintenance/SKILL.md`。

## 建置與發佈

1. 安裝 .NET 8/9/10 SDK（建議使用 `dotnet --list-sdks` 確認）。
2. 執行建置：
   ```bash
   dotnet build LibGGPK3.sln
   ```
3. NativeAOT 發佈（macOS ARM 範例）：
   ```bash
   dotnet publish Examples/PoeChinese3/PoeChinese3.csproj \
     -c Release -r osx-arm64 \
     /p:PublishAot=true /p:StripSymbols=true
   ```
4. 驗證 `.app` 內的 `Icon.icns` 是否複製到 `Contents/Resources/`，若無可手動執行：
   ```bash
   install -m 0644 Examples/Icon.icns \
     bin/Release/net10.0/osx-arm64/publish/PoeChinese3.app/Contents/Resources/Icon.icns
   ```

## PoeChinese3 路徑記憶與 `--use-default`

- 自 2026-03-11 的 `b92df07e75dcd11de836346443457a3e0a1bfa74` 起，`PoeChinese3` 會把成功中文化使用的 GGPK 路徑寫入 `%APPDATA%/PoeChinese3/lastpath.txt`（macOS 為 `~/Library/Application Support/PoeChinese3/lastpath.txt`），下次啟動自動帶入。
- `--use-default` / `-d` 旗標會套用「記憶路徑 → 自動偵測（.app 同層、`PoE.app`、目前目錄）→ Windows 預設 `C:\Program Files (x86)\Grinding Gear Games\Path of Exile\Content.ggpk`」，完全跳過互動輸入。
- 未加旗標但資料存在時，CLI 會直接顯示「使用預設路徑」，僅在檔案不存在時退回互動式輸入。

```bash
./PoeChinese3 --use-default
# 或
./PoeChinese3 -d
```

> 由於 CLI 會自動記憶 GGPK 路徑並可用 `--use-default`/`-d` 套用，不再提供過去那種固定路徑的 macOS 腳本範例。

## 內建字型（免 `Font.ttf` 手動複製）

- 上游的 macOS 指引會要求先找到一份中文字型，改名為 `Font.ttf` 並放在 `PoeChinese_osx-x64` 旁邊才有字形。
- 本 fork 把 Source Han Sans TW 打包在 `.app` 及 CLI 中，不需額外操作即可顯示繁中。
- 如需替換，只要把 `Font.ttc`／`Font.ttf`／`Font.otf` 放在執行檔旁，會優先使用自訂字型。

## 維護排程

- 每週檢查上游 `main`，執行 `git fetch upstream && git rebase upstream/main`。
- Rebase 後跑 `dotnet test`、`dotnet publish`（osx-arm64/native）確認。
- 更新 release drafter 草稿，並依情況產出 `2.7.x-fork.y` tag。

## 贊助 / Sponsor

- 支持原作者 [aianlinb](https://paypal.me/aianlinb)
- 支持 fork 維護者 [jakeuj](https://paypal.me/jakeuj)

---

如需更多細節，請查看 repo 內的 `README.md` 與 `.agent/skills/libggpk3-maintenance/SKILL.md`。此頁面可直接透過 GitHub Pages → `Settings > Pages > Build and deployment > Source = GitHub Actions` 或 `Source = Deploy from a branch (docs/)` 啟用。
