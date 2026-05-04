# MSXJuanEditor

**El editor definitivo de gráficos MSX1. Nativo. Rápido. Pixel-perfect.**

Aplicación de escritorio Windows para crear y editar gráficos MSX1 Screen 2 — tiles, mapas de pantalla y sprites — con las restricciones reales del TMS9918A integradas.

Construido con Tauri (Rust + WebView2). Sin Electron. ~8 MB. Un click.

---

## Características

### Editor de tiles
- 256 tiles 8×8 con edición pixel a pixel.
- Paleta fija MSX1 (16 colores TMS9918A).
- Validación 2-colores-por-fila en tiempo real.
- Flip H/V, rotar, invertir, limpiar.
- Cargar tileset PNG (256×64 = 32×8 tiles), guardar PNG modificado.

### Editor de mapa
- Rejilla completa 32×24 (256×192, resolución nativa Screen 2).
- Pintar / borrar con click izquierdo / derecho.
- Tooltip con número de tile y coordenadas.
- **Selección de bloques con Shift+drag, Copy/Paste con Ctrl+C / Ctrl+V**, preview fantasma del tamaño del bloque mientras mueves el cursor, Esc para limpiar.
- Fill / Clear.
- Export / Import JSON, Export C array (MSXgl-ready), Export .bin (768 bytes raw VRAM).

### Editor de sprites
- **Modo 8×8**: 32 sprites individuales.
- **Modo 16×16**: 8 sprites compuestos (4 sub-sprites cada uno, layout MSX TL-BL-TR-BR).
- Un color por sprite (constraint hardware), color 0 = transparente.
- Múltiples sets — frames de animación intercambiables en runtime.
- Crear / renombrar / borrar sets.
- Flip H/V, rotar, invertir, limpiar, **copy/paste sprites entre slots con Ctrl+C / Ctrl+V**.
- Export / Import JSON, C arrays, binario (.bin).
- Formato binario: 256 bytes patrones + 32 bytes colores = 288 bytes por set.

### Editor mixto (composición)
- Lienzo 8×8 tiles (64×64 px) escalado a 512×512 en pantalla.
- Carga un bloque del mapa (Shift+drag → Ctrl+C → tab Mixed → Cargar).
- Pintar sprites encima con **snap a 8×8** y preview fantasma en el cursor.
- Click der elimina el sprite bajo el cursor.
- Editor de píxeles inline del sprite seleccionado (modifica el patrón compartido).
- Lista de sprites colocados con click-para-editar y X-para-borrar.
- Selectores de set y modo (8×8 / 16×16) independientes de la pestaña Sprites.

📖 **[Ver manual de usuario completo](USER_MANUAL.md)**

---

## Capturas

*Próximamente.*

---

## Compilación

### Requisitos para compilar
- [Rust](https://rustup.rs/) (toolchain estable)
- [Node.js](https://nodejs.org/) (para Tauri CLI)
- Windows 10/11 con WebView2 (Win11 lo incluye; en Win10 normalmente lo trae Microsoft Edge)

### Requisitos para ejecutar
- **Windows 11**: nada extra.
- **Windows 10**: WebView2 Runtime. Lo trae Edge en sistemas actualizados. Si falta:
  - Usa el **instalador NSIS** (descarga WebView2 automáticamente), o
  - Instálalo manualmente desde [WebView2 Runtime](https://developer.microsoft.com/en-us/microsoft-edge/webview2/).

### Build de release
```bash
npm install
npx tauri build
```

Salida:
- `src-tauri/target/release/msx-editor.exe` — Ejecutable portable (~8 MB)
- `src-tauri/target/release/bundle/nsis/MSXJuanEditor_<version>_x64-setup.exe` — Instalador NSIS (~1.8 MB)

### Desarrollo
```bash
npx tauri dev
```

---

## Restricciones MSX1 Screen 2

El editor refleja las limitaciones reales del VDP TMS9918A:

| Constraint | Valor |
|---|---|
| Resolución | 256 × 192 píxeles |
| Tile | 8 × 8 píxeles |
| Tiles por pantalla | 32 × 24 = 768 |
| Tabla de patrones | 3 bancos × 256 patrones |
| Colores por fila | 2 (FG + BG) |
| Paleta | 16 colores fijos |
| Sprites 8×8 | 32 máx. |
| Sprites 16×16 | 8 máx. (4 sub-sprites cada uno) |
| Color por sprite | 1 |
| Sprites por scanline | 4 máx. |

---

## Formatos de exportación

### Tileset
- **PNG** (256×64, 32 columnas × 8 filas de 8×8 tiles)

### Mapa de pantalla
- **JSON** — `{ width, height, tiles[] }` para guardar/cargar proyecto.
- **C array** — `static const u8 g_ScreenLayout[768]` para MSXgl.
- **Binary** — 768 bytes raw, carga directa con `VDP_WriteVRAM_16K`.

### Sprites
- **JSON** — múltiples sets con patrones y colores.
- **C array** — `g_SpritePatterns[32][8]` + `g_SpriteColors[32]`.
- **Binary** — 288 bytes (256 patrón + 32 color), listos para VRAM.

---

## Stack técnico

- **Tauri 2.x** — ventana nativa, footprint mínimo
- **Rust** — backend (thin wrapper, extensible)
- **HTML/CSS/JS** — frontend en una sola página
- **WebView2** — renderer nativo Windows (sin browser bundleado)

---

## Licencia

MIT

---

*Hecho con píxeles y pasión por la plataforma MSX. Herramienta de assets para [puyopuyoMSX1](https://github.com/antxiko/puyopuyoMSX1).*
