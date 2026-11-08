# scoop-epubcheck

A [Scoop](https://scoop.sh) bucket for 64-bit Windows providing self-contained packages for [EPUBCheck](https://github.com/w3c/epubcheck), [OpenReadest](https://github.com/luyishui/OpenReadest), and the [WinApp CLI](https://github.com/microsoft/winappCli). 32-bit Windows is not supported.

## EPUBCheck

Every release bundles everything needed to run EPUBCheck — no system Java, no configuration:

- `epubcheck.cmd` — launcher (shim-friendly)
- `epubcheck.jar` + `lib/` — EPUBCheck itself
- `jre/` — private Eclipse Temurin JRE (Windows x64 only)
- `licenses/`, `LICENSE.txt`, `THIRD-PARTY.txt` — upstream license files

## OpenReadest

[OpenReadest](https://github.com/luyishui/OpenReadest) is an unofficial [Readest](https://github.com/readest/readest) fork focused on local reading with WebDAV sync (Tauri/WebView2 GUI app, AGPL-3.0).

The bucket ships a **portable build repackaged from the official Windows installer** (7-Zip extraction in CI):

- `openreadest.exe` — the app itself
- `openreadest_thumbnail.dll` — Explorer thumbnail provider library (not registered by the portable package)
- `LICENSE.txt`, `NOTICE.md` — upstream license and notices

Unlike the stock installer, the portable package runs without UAC and makes **no registry changes and no file associations** (no epub/pdf association takeover, no thumbnail handler registration, no `openreadest://` deep link). App data lives in `%APPDATA%\com.openreadest.app`, outside of Scoop's directory. The app requires the Microsoft Edge WebView2 Runtime (preinstalled on Windows 11 and up-to-date Windows 10).

## WinApp CLI

[WinApp CLI](https://github.com/microsoft/winappCli) is Microsoft's command-line tool for Windows SDK setup, MSIX packaging, package identity, manifests, certificates, and build tools (MIT).

The bucket publishes a **portable x64 package** (`winappcli-x64.zip`) repackaged from the upstream release:

- `winapp.exe` — the CLI
- `libSkiaSharp.dll`, `libHarfBuzzSharp.dll`, `WinApp.DevTools.Managed.dll`, `WinApp.DevTools.Native.dll` — native and DevTools dependencies
- `winapp-devtools-schema.json`, `LICENSE.txt` — DevTools schema and upstream MIT license
- Debug symbols (`*.pdb`) are removed to keep the download small

Unlike the WinGet/MSIX distribution, this build runs unpackaged, so `winapp run` can register and unregister development packages normally (the MSIX build runs in a package container where deployment calls fail).

## Install

```powershell
scoop bucket add scoop-epubcheck https://github.com/linkcccp/scoop-epubcheck
scoop install epubcheck
scoop install openreadest
scoop install winappcli
```

## Usage

```powershell
epubcheck my-book.epub
epubcheck --version
```

OpenReadest is a GUI app — start it from the Start Menu shortcut, through the `openreadest` shim, or pass files to open:

```powershell
openreadest my-book.epub
```

WinApp CLI is a command-line tool:

```powershell
winapp --version
winapp run .
```

## How updates work

A scheduled GitHub Action runs daily per app:

1. [update-epubcheck.yml](.github/workflows/update-epubcheck.yml) detects the latest upstream EPUBCheck release and the latest Temurin 21 JRE, downloads both, verifies checksums, assembles and smoke-tests a bundled release archive, publishes a release tagged `v<epubcheck>-jre<jre>` (e.g. `v5.4.0-jre21.0.12.1`) and updates [bucket/epubcheck.json](bucket/epubcheck.json).
2. [update-openreadest.yml](.github/workflows/update-openreadest.yml) detects the latest upstream OpenReadest release, verifies the installer against the upstream digest, extracts it with 7-Zip into a portable package (static smoke test, license files included), publishes a release tagged `openreadest-v<version>` (e.g. `openreadest-v0.9.100`) and updates [bucket/openreadest.json](bucket/openreadest.json).
3. [update-winappcli.yml](.github/workflows/update-winappcli.yml) detects the latest upstream WinApp CLI release, verifies `winappcli-x64.zip`, strips debug symbols, adds the upstream LICENSE, smoke-tests `winapp.exe --version`, publishes a release tagged `winappcli-v<version>` (e.g. `winappcli-v0.8.0`) and updates [bucket/winappcli.json](bucket/winappcli.json).

OpenReadest and WinApp CLI releases are published with `--latest=false` so the EPUBCheck release keeps the repository's *latest* badge.

The manifests can also be refreshed manually with `scoop update` / `scoop status` after a release is published.

## Development

- `bin\checkver.ps1 epubcheck` — compare the epubcheck manifest version with published releases (add `-Update` to rewrite it)
- `bin\checkver.ps1 openreadest` — same for the openreadest manifest
- `bin\checkver.ps1 winappcli` — same for the winappcli manifest
- `bin\test.ps1` — run the official Scoop bucket test suite: manifest schema validation plus repository style checks (requires PowerShell Pester ≥ 5.2 and BuildHelpers)

## Disclaimer

Unofficial packaging. EPUBCheck is developed by the [W3C](https://www.w3.org/); this bucket is not affiliated with or endorsed by the W3C or the Eclipse Adoptium project.

OpenReadest is an unofficial fork of Readest maintained by [luyishui](https://github.com/luyishui/OpenReadest) and licensed [AGPL-3.0](https://github.com/luyishui/OpenReadest/blob/master/LICENSE); this bucket is not affiliated with the Readest project or the OpenReadest author. Repackaging notes for each release are published in the [release notes](https://github.com/linkcccp/scoop-epubcheck/releases).

WinApp CLI is developed by [Microsoft](https://github.com/microsoft/winappCli) and licensed [MIT](https://github.com/microsoft/winappCli/blob/main/LICENSE); this bucket is not affiliated with or endorsed by Microsoft.
