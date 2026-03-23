# MSXJuanEditor

**The ultimate MSX1 graphics editor. Native. Fast. Pixel-perfect.**

A native Windows desktop application for creating and editing MSX1 Screen 2 graphics — tiles, screen maps, and sprites — with the exact TMS9918A constraints built in.

Built with Tauri (Rust + WebView2). No Electron bloat. 8MB. One click.

---

## Features

### Tile Editor
- Edit all 256 tiles (8x8 pixels each) with pixel-perfect precision
- MSX1 TMS9918A fixed palette (16 colors)
- Per-row 2-color validation with real-time warnings
- Flip, rotate, invert, clear operations
- Load any tileset PNG (32x8 tile grid = 256x64px)
- Save modified tileset as PNG

### Screen Map Editor
- Full 32x24 tile grid (256x192 pixels — native MSX Screen 2 resolution)
- Paint tiles with click & drag, erase with right-click
- Tile tooltip showing tile number and position on hover
- Fill, clear operations
- Export/Import as JSON (for project continuity)
- Export as C array (ready for MSXgl)
- Export as raw .bin (768 bytes, direct VRAM load)

### Sprite Editor
- **8x8 mode**: Edit all 32 individual sprites
- **16x16 mode**: Edit 8 composite sprites (4 sub-sprites each, MSX layout: TL-BL-TR-BR)
- Single color per sprite (hardware constraint)
- Color 0 = eraser (transparent)
- Multiple sprite sets — prepare different animation frames and swap them at runtime
- Create, rename, delete sets
- Flip H/V, rotate, invert, clear
- Export/Import JSON, C arrays, binary (.bin)
- Binary format: 256 bytes patterns + 32 bytes colors = 288 bytes per set

---

## Screenshots

*Coming soon*

---

## Building

### Prerequisites
- [Rust](https://rustup.rs/) (stable toolchain)
- [Node.js](https://nodejs.org/) (for Tauri CLI)
- Windows 10/11 with WebView2 (included by default)

### Build
```bash
cd tools/msx-editor
npm install
cargo tauri build
```

Output:
- `src-tauri/target/release/msx-editor.exe` — Portable executable (8MB)
- `src-tauri/target/release/bundle/nsis/MSX Editor_1.0.0_x64-setup.exe` — Installer (1.8MB)

### Development
```bash
cargo tauri dev
```

---

## MSX1 Screen 2 Constraints

This editor enforces the real hardware limitations of the TMS9918A VDP:

| Constraint | Value |
|---|---|
| Screen resolution | 256 x 192 pixels |
| Tile size | 8 x 8 pixels |
| Tiles per screen | 32 x 24 = 768 |
| Pattern table | 3 banks x 256 patterns |
| Colors per tile row | 2 (foreground + background) |
| Palette | 16 fixed colors (not reprogrammable) |
| Sprites (8x8) | 32 max |
| Sprites (16x16) | 8 max (4 sub-sprites each) |
| Sprite colors | 1 color per sprite |
| Sprites per scanline | 4 max |

---

## Export Formats

### Tileset
- **PNG** (256x64, 32 columns x 8 rows of 8x8 tiles)

### Screen Map
- **JSON** — `{ width, height, tiles[] }` for project save/load
- **C Array** — `static const u8 g_ScreenLayout[768]` ready for MSXgl
- **Binary** — 768 bytes raw tile indices, direct `VDP_WriteVRAM_16K` load

### Sprites
- **JSON** — Multiple sets with patterns and colors
- **C Array** — `g_SpritePatterns[32][8]` + `g_SpriteColors[32]`
- **Binary** — 288 bytes (256 pattern + 32 color), ready for VRAM

---

## Tech Stack

- **Tauri 2.x** — Native window, minimal footprint
- **Rust** — Backend (currently thin wrapper, extensible)
- **HTML/CSS/JS** — Frontend (single self-contained page)
- **WebView2** — Windows native renderer (no bundled browser)

---

## License

MIT

---

*Made with pixels and passion for the MSX platform.*
