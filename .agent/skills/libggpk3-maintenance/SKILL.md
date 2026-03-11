---
name: libggpk3-maintenance
description: 保持 /Users/jakeuj/RiderProjects/LibGGPK3 fork 與 aianlinb/LibGGPK3 同步，並在 rebase 後完成 2.7.5-fork.1 版本號、macOS Icon 與 build 驗證。當使用者提到同步 LibGGPK3、版本提升、Icon 警告或 .NET 10.0 相關調整時務必使用此技能。
---

# LibGGPK3 維護流程
> 目標：確保本地 fork (`main`) 追上上游 `aianlinb/LibGGPK3`，並維持 2.7.5-fork.1 版本資訊與 macOS Icon 設定無警告。

## 1. 環境與位置
- 作業系統：macOS（Apple Silicon）。
- 專案根目錄：`/Users/jakeuj/RiderProjects/LibGGPK3`。
- 主要 .NET 版本：`net10.0`。（上游仍以 `net8.0` 為基準，但 fork 保持 net10.0 相容。）

## 2. 同步上游並 rebase
1. 進入專案根目錄。
2. 確認 remote：`git remote -v`。若沒有 `upstream`，新增：
   ```bash
   git remote add upstream https://github.com/aianlinb/LibGGPK3.git
   ```
3. 抓上游：`git fetch upstream`。
4. 在 `main` 執行：`git rebase upstream/main`。
5. 若 rebase 遇到衝突，依下方章節處理，再用 `git rebase --continue` 繼續。
6. 完成後以 `git status -sb` 確認乾淨，必要時 `git push --force-with-lease origin main` 更新 fork。

## 3. 常見衝突與必修內容
### 3.1 Directory.Build.props
保持以下內容（簡化重點段落）：
```xml
<Project>
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Version>2.7.5-fork.1</Version>
    <AssemblyVersion>2.7.5.1</AssemblyVersion>
    <FileVersion>2.7.5.1</FileVersion>
    <Nullable>enable</Nullable>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
    <CheckForOverflowUnderflow Condition="$(Configuration) == 'Debug'">true</CheckForOverflowUnderflow>
    <InvariantGlobalization>true</InvariantGlobalization>
    <PredefinedCulturesOnly>false</PredefinedCulturesOnly>
    <IsAotCompatible>true</IsAotCompatible>
    <RollForward>LatestMajor</RollForward>
    <BaseOutputPath>$(SolutionDir)bin/</BaseOutputPath>
    <OutDir>$(BaseOutputPath)$(Configuration)/</OutDir>
    <NoWarn>1573;1591</NoWarn>
    <PathMap>$(MSBuildProjectDirectory)=$(MSBuildProjectName)</PathMap>
    <OodleBasePath>$(MSBuildThisFileDirectory)External/OodleUE/Engine/Source/Runtime/OodleDataCompression/Sdks/2.9.14/lib</OodleBasePath>
    <OodleDistribPath>$(MSBuildThisFileDirectory)External/OodleUE/Engine/Source/Programs/Shared/EpicGames.Oodle/Sdk/2.9.10</OodleDistribPath>
    <DebugSymbols Condition="'$(PublishAOT)' == 'true'">false</DebugSymbols>
    <DebugType Condition="'$(PublishAOT)' == 'true'">none</DebugType>
    <GenerateDocumentationFile Condition="'$(PublishAOT)' == 'true'">false</GenerateDocumentationFile>
    <StripSymbols Condition="'$(PublishAOT)' == 'true'">true</StripSymbols>
    <NativeDebugSymbols Condition="'$(PublishAOT)' == 'true'">false</NativeDebugSymbols>
  </PropertyGroup>

  <!-- Oodle 動態庫僅在非 NativeAOT 時複製 -->
  <ItemGroup Condition="$([MSBuild]::IsOSPlatform('OSX')) AND '$(OutputType)' == 'Exe' AND '$(PublishAOT)' != 'true'">
    <None Include="$(OodleBasePath)/Mac/liboo2coremac64.2.9.14.dylib" Link="liboo2core.dylib">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
      <Visible>false</Visible>
    </None>
  </ItemGroup>
  <!-- Linux/Windows ItemGroup 同上，保持與目前 repo 一致 -->
</Project>
```
> 若 rebase 把 TargetFramework 或 Version 改回上游值（net8.0 / 2.7.5），務必改回上述 fork 設定。

### 3.2 PoeChinese3 字型/版權段落
- `Examples/PoeChinese3/Program.cs` 應：
  - 使用 `var assembly = Assembly.GetExecutingAssembly();`
  - 依 `version.Revision` 判斷輸出格式，版權行含 `aianlinb, jakeuj`。
  - `Console.WriteLine("流亡黯道 - 啟用繁體中文語系  By aianlinb, jakeuj");`

### 3.3 SpanExtensions → .NET Split API
- `LibBundle3/Index.cs` 及 `LibGGPK3/Records/DirectoryRecord.cs` 需改用 `.Split('/')` 枚舉器，而非自訂 `SpanExtensions.Split`。
- 確保 `MemoryExtensions.EndsWith(path, "/")` 用於檔名檢查。

### 3.4 macOS Icon 警告
- `Examples/Icon.icns` 已存在，需編入 Eto 專案：
  - `Examples/VisualGGPK3/VisualGGPK3.csproj` 與 `Examples/VPatchGGPK3/VPatchGGPK3.csproj` 各自加入：
    ```xml
    <Content Include="../Icon.icns">
      <Link>Icon.icns</Link>
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </Content>
    ```
  - 讓 `Eto.Platform.Mac64` 生成 .app 時自動帶入 Icon，避免 `Icon.icns does not exist` 警告。

## 4. 版本與釋出檢查
1. `grep -n "<Version" Directory.Build.props` 確認數值（2.7.5-fork.1 / 2.7.5.1）。
2. 若需要標記發行版號，更新 README 或 Release drafter。
3. 執行 `dotnet build LibGGPK3.sln`：
   - 預期 **0 warnings**（Icon 問題已解）；若有其他警告，再依專案需要補檔。
4. 驗證可選流程：
   - `dotnet publish` NativeAOT 範例（如 PoeChinese3）。
   - 執行核心工具（PatchGGPK3、VisualGGPK3 等）進行煙霧測試。

## 5. 提交與推送
1. 確認 `git status -sb` 僅包含預期檔案（props、csproj、程式碼調整）。
2. 撰寫 commit 訊息（例：`chore: sync upstream 2.7.5 & refresh fork config`）。
3. 推送：`git push --force-with-lease origin main`（rebase 後需要 force）。
4. 若準備 Release，可 tag `v2.7.5-fork.1`。

## 6. 故障排除
- **Rebase 衝突過多**：可暫停 (`git rebase --abort`) 改用 `git merge upstream/main` 取得 context，再重新規畫。
- **Icon 仍警告**：確認 `Icon.icns` 真有被複製到 `bin/Debug/net10.0/<App>.app/Contents/Resources/`。若無，手動 `cp Examples/Icon.icns ...` 檢查路徑。
- **Oodle 子模組**：`External/OodleUE` 為 submodule，若缺失需 `git submodule update --init`，或在本地提供相同路徑結構。

## 7. PoeChinese3 桌面腳本（固定路徑）
若玩家（含自己）已將發佈的 `PoeChinese3.app` 放在 `/Applications`，且 GGPK 放在 CrossOver Bottle（預設示範路徑：`/Users/jakeuj/Library/Application Support/CrossOver/Bottles/PoB/drive_c/Program Files (x86)/Grinding Gear Games/Path of Exile/Content.ggpk`），可提供以下桌面腳本便於一鍵中文化：

```bash
cat <<'EOF' > ~/Desktop/PoeChinese3TW.command
#!/bin/bash
set -euo pipefail
GGPK_PATH='/Users/jakeuj/Library/Application Support/CrossOver/Bottles/PoB/drive_c/Program Files (x86)/Grinding Gear Games/Path of Exile/Content.ggpk'
EXEC='/Applications/PoeChinese3.app/Contents/Resources/PoeChinese3'
if [[ ! -f "$GGPK_PATH" ]]; then
  osascript -e 'display alert "PoeChinese3" message "找不到 Content.ggpk\n請確認路徑是否正確"'
  exit 1
fi
if [[ ! -x "$EXEC" ]]; then
  osascript -e 'display alert "PoeChinese3" message "找不到 /Applications/PoeChinese3.app\n請先安裝中文版工具"'
  exit 1
fi
"$EXEC" "$GGPK_PATH"
EOF
chmod +x ~/Desktop/PoeChinese3TW.command
```

重點：
- `GGPK_PATH` 改成實際的媒體路徑即可。
- `EXEC` 指向 `.app` 內真實執行檔，避免程式再要求輸入路徑。
- 第一次在 Finder 執行 `.command` 檔需允許執行權限；若路徑變動，重新編輯腳本即可。

> 完成上述步驟後，LibGGPK3 fork 即可保持與上游同步、具備最新版本號並消除 macOS Icon 警告，同時也能提供終端腳本快速對指定 GGPK 套用中文化。
