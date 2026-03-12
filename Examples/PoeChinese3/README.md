# PoeChinese3

流亡黯道 (Path of Exile) 繁體中文化工具 - macOS ARM64 版本

## 系統需求

- macOS（Apple Silicon）M 系列晶片，ARM64 架構

## 功能

- 自動套用繁體中文語系
- 內嵌思源黑體臺灣版 (Source Han Sans TW) 字型

## 安全性與簽章

- 發佈版（Release）以 Apple Developer ID Application 進行簽署，通常不會被 macOS Gatekeeper 擋下。
- 若仍遇到阻擋，可在 Finder 右鍵選擇「打開」或使用終端機執行以繞過首次驗證。

## 使用方式

### 方式一：放到遊戲目錄（推薦）

1. 將 `PoeChinese3` 執行檔放到 `Content.ggpk` 所在目錄
2. 雙擊執行即可自動中文化

### 方式二：指定路徑

```bash
./PoeChinese3 "/path/to/Content.ggpk"
```

### 方式三：沿用記憶路徑 (`--use-default`)

自 2026-03-11 的 `b92df07e75dcd11de836346443457a3e0a1bfa74` 開始，PoeChinese3 會記住最後一次成功中文化時的 GGPK 路徑，並提供 `--use-default`（或 `-d`）旗標快速使用：

- 路徑會寫入 `%APPDATA%/PoeChinese3/lastpath.txt`（macOS 在 `~/Library/Application Support/PoeChinese3/lastpath.txt`）。
- 啟動時會依序檢查：記憶路徑 → `.app`/執行檔同層的 `Content.ggpk` → `PoE.app` bundle（CrossOver/Wineskin）→ 目前工作目錄 → Windows 預設 `C:\Program Files (x86)\Grinding Gear Games\Path of Exile\Content.ggpk`。
- 若路徑失效才會回到互動式輸入。

```bash
./PoeChinese3 --use-default
# 或
./PoeChinese3 -d
```

### macOS 範例路徑

CrossOver/Wine 安裝的 PoE：
```bash
./PoeChinese3 "/Users/你的帳號/Applications/XXX/PoE.app/Contents/SharedSupport/prefix/drive_c/Program Files (x86)/Grinding Gear Games/Path of Exile/Content.ggpk"
```

## 自訂字型

程式會按以下順序尋找字型：
1. `Font.ttc`（使用者自訂）
2. `Font.ttf`（使用者自訂）
3. `Font.otf`（使用者自訂）
4. 內嵌的思源黑體（預設）

如需使用其他字型，將字型檔案重新命名後放在執行檔同目錄即可。

## 編譯

### 前置需求

- [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0)
- Git（用於下載 submodule）

### 下載原始碼

```bash
git clone --recursive https://github.com/jakeuj/LibGGPK3.git
cd LibGGPK3
```

如果已經 clone，需要初始化 submodule：
```bash
git submodule update --init --recursive
```

### 發布 NativeAOT 單檔執行檔

```bash
dotnet publish Examples/PoeChinese3 -c Release -r osx-arm64 --self-contained -p:PublishAOT=true -o bin/publish/osx-arm64
```

輸出檔案位於 `bin/publish/osx-arm64/PoeChinese3`

## Oodle 壓縮庫 (liboo2core)

Path of Exile 使用 Oodle 壓縮演算法來壓縮 `.bundle` 檔案。本專案透過 git submodule 整合了 [OodleUE](https://github.com/WorkingRobot/OodleUE)。

### 支援的平台

| 平台 | 庫檔案 |
|------|--------|
| Windows x64 | `oo2core_win64.dll` |
| Windows ARM64 | `oo2core_winarm64.dll` |
| Linux x64 | `liboo2corelinux64.so` |
| Linux ARM64 | `liboo2corelinuxarm64.so` |
| macOS x64 | `liboo2coremac64.dylib` |
| macOS ARM64 | `liboo2coremac64.dylib` |

### NativeAOT 發布

使用 NativeAOT 發布時，Oodle 庫會透過 `DirectPInvoke` 靜態鏈接到執行檔中，不需要額外的 `.dylib` 或 `.dll` 檔案。

庫檔案路徑：
```
External/OodleUE/Engine/Source/Runtime/OodleDataCompression/Sdks/2.9.14/lib/
├── Linux/
│   ├── liboo2corelinux64.a
│   └── liboo2corelinuxarm64.a
├── Mac/
│   └── liboo2coremac64.a
└── Win64/
    ├── oo2core_win64.lib
    └── oo2core_winarm64.lib
```

## 授權

本專案 fork 自 [aianlinb/LibGGPK3](https://github.com/aianlinb/LibGGPK3)，採用 AGPL-3.0 授權。

第三方元件授權請參閱 [THIRD_PARTY_LICENSES.md](../../THIRD_PARTY_LICENSES.md)。

## 致謝

- [aianlinb/LibGGPK3](https://github.com/aianlinb/LibGGPK3) - 原始專案
- [Adobe Source Han Sans](https://github.com/adobe-fonts/source-han-sans) - 思源黑體字型
- [WorkingRobot/OodleUE](https://github.com/WorkingRobot/OodleUE) - Oodle 壓縮庫
