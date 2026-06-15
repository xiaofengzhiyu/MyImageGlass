<div align="center">

# 🖼️ ImageGlass

### A Fast, Seamless Photo Viewer

ImageGlass is a fast, modern, open-source image viewer built for Windows, macOS, and Linux. Designed for speed and efficiency, it delivers a smooth, immersive viewing experience by combining high-performance rendering with powerful tools for both everyday users and designers. ImageGlass provides seamless, quick navigation across 90+ image formats, including `WEBP`, `JXL`, `SVG`, `HEIC`, `AVIF`, `HDR`, and raw images.

<br/>

[![Total downloads](https://img.shields.io/github/downloads/d2phap/imageglass/total?color=%23d60068&label=Total%20downloads&)](https://imageglass.org/download)
[![Latest version downloads](https://img.shields.io/github/downloads/d2phap/imageglass/latest/total?color=%23e66700&label=Latest%20version&)](https://imageglass.org/download)

[![Discord](https://img.shields.io/discord/818852544859209748?label=chat&logo=discord&color=%233097B8&style=social)](https://discord.gg/tWjbynH2X8)
[![Twitter Follow](https://img.shields.io/twitter/follow/duongdieuphap?style=social)](https://twitter.com/duongdieuphap)
[![Crowdin](https://d322cqt584bo4o.cloudfront.net/imageglass/localized.svg)](https://crowdin.com/project/imageglass)

<br/>

[**🌐 Website**](https://imageglass.org) &nbsp;•&nbsp;
[**📥 Download**](https://imageglass.org/download) &nbsp;•&nbsp;
[**📚 Docs**](https://imageglass.org/docs) &nbsp;•&nbsp;
[**💬 Discord**](https://discord.gg/tWjbynH2X8) &nbsp;•&nbsp;
[**💖 Donate**](https://imageglass.org/donate)

<br/>

[![ImageGlass 10 beta](https://github.com/user-attachments/assets/053f7f92-8679-449e-9b93-d44e349e0046)](https://imageglass.org/news/announcing-imageglass-10-beta-2-101)

</div>

<br/>

<div align="center">

<a href="https://apps.microsoft.com/detail/9N33VZK3C7TH?launch=true&cid=GitHubRelease&mode=full">
  <img height="58" src="https://github.com/d2phap/ImageGlass/assets/3154213/08a071bb-a6ae-420c-b53b-2317004570d4" alt="Download ImageGlass from the Microsoft Store" />
</a>

<br/>

Prefer the classic installer? Grab it from **[imageglass.org/download](https://imageglass.org/download)**.

</div>


## Download
### Why the Microsoft Store?

- Support the development of ImageGlass directly by purchasing it from the Microsoft Store.
- The Store version offers fast, easy installation across all your Windows devices, with fully automatic, behind-the-scenes updates that deliver the newest features, improvements, and fixes.

### Classic vs. Store

|  | ImageGlass Classic | [ImageGlass Store](https://apps.microsoft.com/detail/9N33VZK3C7TH?launch=true&cid=GitHubRelease&mode=full) |
| -- | -- | -- |
| [All features](https://imageglass.org/docs/features), including Explorer sort order | ✅ | ✅ |
| [Advanced configs for power users](https://imageglass.org/docs/app-configs) | ✅ | ✅ |
| Distribution | 🌐 [ImageGlass.org](https://imageglass.org) & various sources | 🛍️ [Microsoft Store](https://apps.microsoft.com/detail/9N33VZK3C7TH?launch=true&cid=GitHubRelease&mode=full) only |
| Price | 🆓 Free | 🪙 Fee, with a 7-day trial |
| Commercial use | ✅ Recommended to [register](https://imageglass.org/license) | ✅ |
| Auto-update | ❌ User-managed | ✅ Seamless auto-updates |
| Hotfix update | ❌ Official releases only | ✅ As soon as fixes land |

<br/>


## System Requirements
**Version 10**
- Windows 10/11 64-bit, version 1809 (build 17763) or later
- macOS Apple Silicon, version 12 or later
- Linux distros with Flatpak installer

**Version 9**
- Windows 10/11 64-bit, version 1809 (build 17763) or later
- Optional: [WebView2 Runtime 64-bit v119.0.2151 or later](https://go.microsoft.com/fwlink/p/?LinkId=2124703)

<br/>


## Development
The `develop` branch contains the latest commits, while the `prod` branch holds the final stable release.

**Version 10**: Located in `source` folder
- Visual Studio 2026 for Windows build
- VS Code for macOS, Linux build

**Version 9**: Located in `v9` folder
- Visual Studio 2026 on Windows 11
- VS Code for `WebUI`

<br/>


## Build & Deploy (v9 / WinForms)

This section describes how to compile ImageGlass **v9** (`v9/` folder) locally on Windows and produce a runnable, self-contained build. v9 is Windows-only (WinForms, `net9.0-windows10.0.17763.0`).

> The same steps are performed by the GitHub Actions workflow in [`.github/workflows/build.yml`](.github/workflows/build.yml); the commands below are the local equivalent.

### Prerequisites

| Tool | Version | Why |
| -- | -- | -- |
| [.NET SDK](https://dotnet.microsoft.com/download) | **9.0.x** | Compiles the `net9.0-windows` projects. (Do **not** use the 10.0 SDK here — all csproj target `net9.0`.) |
| [Visual Studio 2022](https://visualstudio.microsoft.com/) (or Build Tools) | 17.x | Provides the **full-framework MSBuild** required by the `<COMReference>` (`IWshRuntimeLibrary`) in `ImageGlass.Base`. The `dotnet` CLI uses .NET Core MSBuild, which **cannot** resolve COM references (error `MSB4803`). |
| [Node.js](https://nodejs.org/) | 20+ (tested with 24) | Builds the WebView2 front-end (`WebUI/dist`) used by Quick Setup, Settings, About and Update dialogs. |
| WebView2 Runtime (x64) | v119.0.2151+ | **Only needed at runtime** for the app to render its HTML-based dialogs. Optional for the build itself. |

> No `dotnet workload` packages are required — the Windows desktop (WinForms) stack ships with the Windows SDK.

### Why two output projects?

Under the `Publish_Release` configuration, `ImageGlass.csproj` **deliberately excludes** the `igcmd` project reference:

```xml
<ProjectReference Include="..\igcmd\igcmd.csproj"
                  Condition="'$(Configuration)'!='Publish_Release'" />
```

This means publishing `ImageGlass` alone does **not** produce `igcmd.exe`. You must publish **both** projects into the **same** output folder, otherwise the app exits silently at startup (Quick Setup launches `igcmd.exe`, which is missing). See [Troubleshooting](#troubleshooting) below.

### Step 1 — Build the WebUI front-end (required, once)

The WebView2-based dialogs (Quick Setup, Settings, About, Update) load HTML/JS from `WebUI/dist/`. That folder is **git-ignored** and must be generated with npm before the first build, and again whenever the TypeScript/SCSS sources under `WebUI/src/` change.

```bash
cd v9/Components/ImageGlass.Settings/WebUI
npm install
npm run build
```

If you skip this step, the PostBuild copy target finds no `dist/` files, the published folder is missing `WebUI/*.html`, and the app appears to "do nothing" when launched (the Quick Setup window cannot render and closes immediately).

### Step 2 — Publish both projects to a shared folder

From the repository root, using full-framework MSBuild (not `dotnet`):

```cmd
:: 1. Publish the main app (this also copies Themes / Language / WebUI into the folder)
msbuild v9\ImageGlass\ImageGlass.csproj -t:Publish ^
  -p:Configuration=Publish_Release ^
  -p:Platform=x64 ^
  -p:RuntimeIdentifier=win-x64 ^
  -p:PublishDir=bin\x64\Publish_Release\net9.0-windows10.0.17763.0\

:: 2. Publish igcmd into the SAME folder
msbuild v9\igcmd\igcmd.csproj -t:Publish ^
  -p:Configuration=Publish_Release ^
  -p:Platform=x64 ^
  -p:RuntimeIdentifier=win-x64 ^
  -p:PublishDir=..\ImageGlass\bin\x64\Publish_Release\net9.0-windows10.0.17763.0\
```

To build for ARM64, replace `-p:Platform=x64 -p:RuntimeIdentifier=win-x64` with `-p:Platform=ARM64 -p:RuntimeIdentifier=win-arm64` (and adjust the `PublishDir` path accordingly).

For a quick, non-packaged smoke build (no self-contained packaging), build the whole solution instead:

```cmd
msbuild v9\ImageGlass.slnx -t:Build -p:Configuration=Release -p:Platform=x64 -p:RuntimeIdentifier=win-x64
```

### Step 3 — Run

The runnable app lives in the shared publish folder:

```
v9\ImageGlass\bin\x64\Publish_Release\net9.0-windows10.0.17763.0\ImageGlass.exe
```

Double-click `ImageGlass.exe`, or launch it from a terminal. On first run the **Quick Setup** wizard appears (rendered via WebView2); after completing/skipping it, the main window opens.

### What the PostBuild target copies

`ImageGlass.csproj` defines a `PostBuild` target that copies runtime resources into both `$(OutputPath)` (regular build) and `$(PublishDir)` (when publishing):

| Source | Destination | Contents |
| -- | -- | -- |
| `_Setup/Assets/Themes/**` | `Themes/` | Built-in theme packs (`Kobe`, `Kobe-Light`, …) including `igtheme.json` + SVG icons |
| `_Setup/Assets/Language/**` | `Language/` | Built-in language packs (`*.iglang.json`) |
| `Components/ImageGlass.Settings/WebUI/dist/**` | `WebUI/` | Compiled WebView2 front-end (needs Step 1) |

If any of these are missing from the output folder, the app fails at startup with a misleading error (e.g. *Unable to load 'Kobe-Light' theme pack*) or exits silently.

### Publishing with Visual Studio

In the IDE you must **publish each project separately**, pointing both `PublishDir`/target folders at the **same** directory:

1. Right-click **ImageGlass** → **Publish** → use the `Properties/PublishProfiles/x64.pubxml` profile.
2. Right-click **igcmd** → **Publish** → use `Properties/PublishProfiles/x64.pubxml` (same output folder).
3. Run `npm run build` in `WebUI/` first (Visual Studio does **not** build the front-end for you).

### Troubleshooting

| Symptom | Cause | Fix |
| -- | -- | -- |
| Double-click `ImageGlass.exe` → nothing happens, process exits with code 0 | `igcmd.exe` missing from the output folder (only ImageGlass was published) | Publish **both** projects to the same folder (Step 2) |
| Same silent exit, even with `igcmd.exe` present | `WebUI/dist` was never built, so Quick Setup's HTML is missing | Run `npm install && npm run build` in `WebUI/` (Step 1), then re-publish |
| `Unable to load 'Kobe-Light' theme pack` / `igtheme.json appears corrupted` | `Themes/` folder empty in the output (PostBuild didn't copy, or wrong `PublishDir`) | Confirm the PostBuild target ran; verify `Themes/Kobe-Light/igtheme.json` exists in the publish folder |
| `error MSB4803: The "ResolveComReference" task was not found` | Built with `dotnet` CLI instead of full-framework MSBuild | Use `msbuild` from Visual Studio / Build Tools |
| `NETSDK1138` / target framework not found | Wrong .NET SDK installed | Install the **9.0.x** SDK |

<br/>



## Roadmap 2026

[![ImageGlass 2026 roadmap](https://github.com/user-attachments/assets/9cf2a8a4-18f4-4852-ad83-5df79239d93f)](https://github.com/d2phap/ImageGlass/discussions/2287)

<br/>


## License

ImageGlass is free for both personal and commercial use, except for the Store version. If you intend to use ImageGlass at your place of business or for commercial purposes, registering at [imageglass.org/license](https://imageglass.org/license) is recommended but not enforced.

<br/>


## This project needs your help!

If you find ImageGlass useful and would like to support its ongoing development, please consider making a donation. Your support — whether financial or simply sharing ImageGlass with others — means the world to me. Every bit helps keep the project alive and free for everyone.

#### 👉 Explore the ways to support at [imageglass.org/donate](https://imageglass.org/donate).

