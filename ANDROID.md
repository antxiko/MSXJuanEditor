# Android port — work in progress

This branch (`android`) is the prep ground for a Tauri-based Android build of MSXJuanEditor. The desktop builds (Windows/macOS/Linux) are unaffected: lib/main split is purely additive.

## Status

- [x] `src-tauri/src/lib.rs` + `main.rs` split with `tauri::mobile_entry_point` for Android entry.
- [x] `Cargo.toml` `[lib]` section so the crate produces a `cdylib` for the JNI side.
- [x] CI workflow `.github/workflows/android.yml` that builds a debug APK on push to `android` (or on demand via `workflow_dispatch`) and uploads it as an artifact.
- [ ] `cargo tauri android init` ran locally (needs Android SDK + NDK).
- [ ] Touch input adaptation (pointer events, no-hover, no-right-click, no-keyboard-shortcuts).
- [ ] Responsive layout for tablet/phone (current UI assumes ~1280px desktop).
- [ ] Signing config + Play Store listing (only relevant when shipping releases).

## Local prerequisites

The first local build needs:
1. **Android Studio** (installed via `winget install Google.AndroidStudio`).
2. Open Android Studio once → accept SDK license → install:
   - Android SDK Platform 34
   - Android SDK Build-Tools 34.0.0
   - **NDK (Side by side)** — required by Rust cross-compile.
3. Set environment variables (PowerShell, persistent):
   ```powershell
   [Environment]::SetEnvironmentVariable('ANDROID_HOME', "$env:LOCALAPPDATA\Android\Sdk", 'User')
   [Environment]::SetEnvironmentVariable('NDK_HOME', "$env:LOCALAPPDATA\Android\Sdk\ndk\<VERSION>", 'User')
   ```
   Replace `<VERSION>` with the installed NDK version, e.g. `26.1.10909125`.
4. Add Rust Android targets:
   ```bash
   rustup target add aarch64-linux-android armv7-linux-androideabi i686-linux-android x86_64-linux-android
   ```

## First-time scaffolding

After the prerequisites are in place:
```bash
npx tauri android init
```

This creates `src-tauri/gen/android/` with the Gradle project (committed to the branch).

## Building

```bash
# Debug APK (unsigned, sideloadable)
npx tauri android build --apk --debug

# Release APK / AAB (needs signing config in tauri.conf.json + keystore)
npx tauri android build
```

Output APKs land in `src-tauri/gen/android/app/build/outputs/apk/...`.

## Running on a device / emulator

```bash
# Connect a phone with USB debugging, or start an AVD from Android Studio
adb devices
npx tauri android dev
```

This installs the debug APK and opens it with live-reload pointing at the dev server.

## Known UI issues to fix before release

The current frontend (`src/index.html`) was designed for desktop. Once the build runs on a device, these will need touch adaptation (in priority order):

1. **Pointer events**: replace all `mousedown` / `mousemove` / `mouseup` with `pointerdown` / `pointermove` / `pointerup`. The Pointer Events API works on both desktop and touch — same code path covers both.
2. **No right-click**: every right-click action (erase tile, erase pixel in editor, delete placed sprite) needs a tool toggle button (Paint / Erase / Select).
3. **No keyboard**: `Shift+drag` for map block selection, `Ctrl+C/V` for copy/paste, `Esc` for clear — all need on-screen toolbar buttons. Already partly there as buttons in some places, but Shift+drag needs a "Select Block" mode.
4. **No hover**: ghost previews (paste rectangle, sprite ghost in mixed editor) need to be re-anchored to "last tap" rather than "hover position", or shown only after a placement-confirm tap.
5. **Tooltip on hover**: replace with a status bar that updates on tap.
6. **Layout**: rework with CSS for narrow viewports — at minimum, swap side-by-side panels for stacked / collapsible sections; consider a drawer for the palette and tile picker on phones.

## CI

The Android CI workflow (`android.yml`) currently:
- Triggers on push to the `android` branch and via manual dispatch.
- Builds an unsigned debug APK on `ubuntu-latest`.
- Uploads the APK as a workflow artifact (downloadable from the Actions run page).

To attach Android builds to GitHub Releases (alongside Windows/macOS/Linux), the artifact step would be replaced by an upload to the matching release tag — left for when the build first works end-to-end.
