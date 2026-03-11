[![NuGet](https://img.shields.io/nuget/v/LibGGPK3.LibBundledGGPK3)](https://www.nuget.org/packages?q=LibGGPK3)

## Overview
A cross-platfrom library for working with Content.ggpk from the game Path of Exile.  
Rewrite of: https://github.com/aianlinb/LibGGPK2

## About This Fork (jakeuj)
This repository tracks the upstream project at [aianlinb/LibGGPK3](https://github.com/aianlinb/LibGGPK3) and periodically rebases on top of it.  
Key differences in this fork:
- **Target Framework & Versioning** – Builds against `net10.0` with the `2.7.5-fork.1` line (`AssemblyVersion/FileVersion 2.7.5.1`) so the fork can ship macOS ARM NativeAOT binaries while staying aligned with upstream 2.7.5 assets.
- **Localization Enhancements** – PoeChinese3 embeds Source Han Sans TW, updates copyright strings, and improves automatic GGPK path detection for macOS bundles.
- **NativeAOT Pipeline** – `Directory.Build.props` includes tuned Publish settings (no debug/doc symbols, strip binaries, Oodle static libs) plus NativeAOT-friendly Oodle copying rules.
- **macOS UX Fixes** – `VisualGGPK3` 與 `VPatchGGPK3` now package `Icon.icns`, eliminating Eto.Mac build warnings and producing properly branded `.app` bundles.
- **CI/CD & Release artifacts** – Fork-specific GitHub Actions, release drafter, and README/sys-req additions (see history around Dec 2025) focus on distributing signed binaries for TW community users.

Upstream bug fixes continue to be merged via rebase; fork-only commits layer on top so you can always run `git fetch upstream && git rebase upstream/main`.

### Fork Setup & Build
1. Clone repo then initialize submodules (OodleUE assets are required for bundle/native builds):
   ```bash
   git clone https://github.com/jakeuj/LibGGPK3.git
   cd LibGGPK3
   git submodule update --init --recursive External/OodleUE
   ```
2. Ensure .NET 8/9/10 SDKs are available (fork defaults to `TargetFramework=net10.0` but multi-targeting still honors upstream projects).
3. Compile all libraries & tools:
   ```bash
   dotnet build LibGGPK3.sln
   ```
4. For NativeAOT samples (PoeChinese3, GGPKFastCompact3 etc.) run `dotnet publish -c Release -r <RID> /p:PublishAot=true`.
5. If you hit macOS bundle warnings, confirm `Examples/Icon.icns` exists in `bin/Debug/net10.0/<App>.app/Contents/Resources/`.

> Need guidelines for keeping the fork current? See `.agent/skills/libggpk3-maintenance/SKILL.md` for the scripted workflow used during this rebase.

## Notice
- Unauthorized modification, redistribution, or commercial use without open-source is prohibited. Please contact the author if needed.  
- Projects in this repository may not be fully thread-safe.  
Exercise caution when processing a single ggpk file from multiple threads.  
- Updates do not guarantee forward compatibility.  
Please review the commit history carefully and verify that your project continues to function as expected before upgrading to a new version.

# LibGGPK3
Handles the Content.ggpk

# LibBundle3
Handles the *.bundle.bin files in the Bundles2 directory  
For Steam/Epic users

# LibBundledGGPK3
Combination of LibGGPK3 and LibBundle3  
Handle both Content.ggpk and the bundle files in it  
For Standalone-Client users

# Examples
Sample programs to realize some simple features of the libraries  
***VisualGGPK3 is not yet completed***
