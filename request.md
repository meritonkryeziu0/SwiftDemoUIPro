# Demo playback improvements: request and work log

## What you wanted

You asked for improvements to CS2 Demo viewing:

1. Prevent mouse-wheel scrolling from cycling the camera mode.
2. Make the SwiftDemoUIPro overlay movable.
3. Add J/K/L playback keys: J to go backward, K to pause/resume, L to go forward, with a configurable skip duration.
4. Add a key (you chose G) to follow the utility thrown by the player you are spectating and follow it to its landing.

## What changed

The branch `feat/demo-playback-shortcuts` adds:

- **J** seeks backward and **L** seeks forward by the configured number of seconds.
- **K** toggles pause/resume through the native DemoUI playback handler.
- In-panel `-` and `+` controls adjust the skip interval. It starts at 15 seconds and is clamped to 1–60 seconds.
- English and Simplified Chinese labels/status strings and README instructions.
- Focused tests for shortcut registration, skip bounds, seek clamping, and pause toggling.

The shortcuts use Panorama's key-bind API and do not rewrite saved CS2 key bindings. The code changes were committed as `9ed1c5a` (`feat(menu): add JKL demo playback shortcuts`) and pushed to the fork.

## What remains

- **Movable overlay:** The inspected code already had a drag handle (the speaker icon at the left of the panel header), edge clamping/snapping, and a reset-position button. To move it, enable CS2 Demo mouse mode and drag that speaker icon. No drag code change was made because no reproducible failure steps were available.
- **Mouse wheel:** Not implemented. The requested behavior was to stop camera cycling but retain panel scrolling. A safe, verified way to intercept only the undesired camera-mode action was not established.
- **Utility chase:** Not implemented. G-based projectile tracking needs validated projectile ownership/trajectory and camera behavior. The bundled Demo parser currently indexes recorded voice data, and there was no representative Demo file available to verify this feature.

## Tests and verification

Passed:

```powershell
node .\tests\test_demo_voice_mask.js
git diff --check
```

The Panorama test covers the new shortcut logic. A live CS2 test of keyboard focus and the visual controls was not performed here. The Windows build/package was also not run by the assistant; the user was preparing the Windows dependencies.

## Build prerequisites

For building the updated in-game VPK and launcher on Windows, install/have available:

- CS2 and Workshop Tools, which provide `game\bin\win64\resourcecompiler.exe`.
- VPKEdit CLI (`vpkeditcli.exe`) to pack the static DemoUI VPK.
- Visual Studio 2022 with **Desktop development with C++**, CMake, Qt 6.5+ MSVC x64, Rust/Cargo, and Node.js.
- Qt `-QtRoot` must be the kit folder containing `lib\cmake\Qt6\Qt6Config.cmake` and `bin\windeployqt.exe`, e.g. `C:\Qt\6.8.3\msvc2022_64` (substitute your actual installed version/path).

An account-free Qt install with `aqtinstall` was discussed. The user reported that Workshop Tools were installed and `resourcecompiler.exe` was found, and that both Qt path checks returned `True`. CMake also needed to be installed. The assistant did not install software or run the Windows build on the user's PC.

## Build, package, and run

Run in PowerShell from the repository root. Replace the VPKEdit and Qt paths if yours differ. The process-level execution-policy bypass below is only for invoking the scripts in that PowerShell process; it does not permanently change policy.

### Build the DemoUI VPK

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\demo-menu.ps1 `
  -Action Build `
  -Cs2Root "C:\Program Files (x86)\Steam\steamapps\common\Counter-Strike Global Offensive" `
  -VpkEditCli "C:\Games\VPKEdit\vpkeditcli.exe"
```

### Build, test, and package the launcher

Run after the VPK build succeeds:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\launcher\build-launcher.ps1 `
  -QtRoot "C:\Qt\6.8.3\msvc2022_64" `
  -Configuration Release `
  -Package
```

The script tests/builds the Rust voice indexer, builds the launcher, runs CTest, and deploys Qt runtime files into the package.

### Launch the packaged app

```powershell
$version = (Get-Content .\VERSION -Raw).Trim()
& ".\launcher\package\SwiftDemoUIPro-v$version\SwiftDemoUIPro.exe"
```

Keep the package directory intact when running the executable. Select a `.dem` in the launcher and choose **Start playback**.

> **Safety:** Demo playback starts CS2 with `-insecure`. Do not use that session for matchmaking. When done, fully exit CS2 and choose **Stop and restore** in the launcher.