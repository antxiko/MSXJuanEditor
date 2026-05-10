# MSXJuanEditor — Manual de usuario

Editor nativo de gráficos para MSX1 / TMS9918A Screen 2: tiles, mapas de pantalla, sprites y un editor mixto de composición.

---

## Índice
1. [Instalación y arranque](#instalación-y-arranque)
2. [Visión general de la interfaz](#visión-general-de-la-interfaz)
3. [Pestaña Tiles & Map](#pestaña-tiles--map)
4. [Pestaña Sprites](#pestaña-sprites)
5. [Pestaña Mixed (editor mixto)](#pestaña-mixed-editor-mixto)
6. [Restricciones de hardware MSX1 (TMS9918A)](#restricciones-de-hardware-msx1-tms9918a)
7. [Formatos de exportación](#formatos-de-exportación)
8. [Atajos de teclado](#atajos-de-teclado)
9. [Resolución de problemas](#resolución-de-problemas)

---

## Instalación y arranque

**Requisitos:** Windows 10 o Windows 11.

**WebView2 Runtime**: el editor lo necesita para renderizar la UI.
- **Windows 11**: incluido de serie, no hace falta hacer nada.
- **Windows 10**: lo trae casi siempre Microsoft Edge (es la base del nuevo Edge). Si tu Win10 está actualizado en los últimos años, ya lo tienes.
- **Si no lo tienes**: el portable `.exe` no arrancará (verás un error "Failed to initialize WebView2"). Opciones:
  1. Instala "Microsoft Edge WebView2 Runtime" desde https://developer.microsoft.com/en-us/microsoft-edge/webview2/
  2. Usa el **instalador NSIS** (`MSXJuanEditor_..._x64-setup.exe`) en vez del portable: si detecta que falta WebView2, lo descarga e instala automáticamente.

**Versión portable:** descarga `MSXJuanEditor.exe` (~8 MB) desde la página de releases, ejecútalo. No requiere instalación, no escribe en el registro. Necesita WebView2 ya presente en el sistema.

**Versión instalable:** ejecuta `MSXJuanEditor_<version>_x64-setup.exe` (~1.8 MB, NSIS). Auto-descarga WebView2 si hace falta.

Al abrirse aparecen tres pestañas en la parte superior: **Tiles & Map**, **Sprites** y **Mixed**.

---

## Visión general de la interfaz

```
┌──────────────────────────────────────────────────────┬─────────────────┐
│ MSXJuanEditor                                         │                 │
│ [ Tiles & Map ] [ Sprites ] [ Mixed ]                 │  Editor de tile │
│                                                       │  (sólo visible  │
│   Contenido de la pestaña activa                      │   en la pestaña │
│                                                       │   Tiles & Map)  │
│                                                       │                 │
└──────────────────────────────────────────────────────┴─────────────────┘
```

El panel derecho con el editor pixel-a-pixel del tile sólo se muestra cuando estás en la pestaña Tiles & Map.

---

## Pestaña Tiles & Map

Esta pestaña tiene tres áreas: **tileset** (256 patrones), **editor de tile** (panel derecho) y **mapa de pantalla** (32×24).

### Tileset (256 tiles, 8×8 píxeles cada uno)

- **Click**: selecciona un tile como "tile activo" para pintar y para editar.
- **Doble click / click + edición**: el tile aparece en el panel derecho para editar pixel a pixel.
- **Hover**: muestra el número de tile bajo el cursor.

Botones de la barra:

| Botón | Función |
|---|---|
| `Fill All` | Rellena todo el mapa con el tile seleccionado |
| `Clear` | Vacía el mapa (todo a tile 0) |
| `Export Map (.json)` | Exporta el layout como JSON `{width, height, tiles[]}` |
| `Import Map (.json)` | Importa un mapa previamente guardado |
| `Export C Array` | Genera `static const u8 g_ScreenLayout[768]` listo para MSXgl |
| `Export .bin` | Descarga 768 bytes raw (carga directa con `VDP_WriteVRAM_16K`) |
| `Load PNG` | Carga un tileset PNG de 256×64 (32 columnas × 8 filas de 8×8) |
| `Import Image` | Importa una imagen entera (PNG / BMP / JPG) y la **trocea en tiles 8×8 quitando duplicados**: detecta los tiles únicos por hash de píxeles, mueve el más oscuro al índice 0, genera el tileset (extiende el tileset si se necesitan más de 8 filas) y rellena el mapa con los índices correspondientes. Las dimensiones de la imagen deben ser múltiplos de 8. Si hay más de 256 tiles únicos te avisa. Si la imagen es más grande que 256×192 el mapa se recorta a 32×24. |
| `Save Tileset PNG` | Guarda el tileset modificado como PNG |

### Editor de tile (panel derecho)

- **Click izq**: pinta con el color de **primer plano** (FG).
- **Click der**: pinta con el color de **fondo** (BG).
- **Paleta superior**: 16 colores fijos de la TMS9918A. Click izq selecciona FG, click der selecciona BG.
- **Operaciones del tile**: Flip H / Flip V / Rotate / Clear / Invert.
- **Aviso 2-color**: bajo el editor verás cada fila del tile con sus colores; si una fila tiene más de 2 colores aparece **WARNING!** porque infringe la restricción de hardware.

### Mapa de pantalla (32×24 = 256×192 px)

- **Click izq + arrastrar**: pinta el tile activo en cada celda por la que pasas.
- **Click der + arrastrar**: borra (escribe tile 0).
- **Hover**: tooltip con `Tile #N (x,y)`.

#### Selección y portapapeles de bloques

- **Shift + click+arrastrar**: dibuja un rectángulo de selección (línea amarilla discontinua).
- **Ctrl + C**: copia el bloque seleccionado al portapapeles (ancho × alto, en tiles).
- Al haber bloque copiado, el ratón sobre el mapa muestra un **rectángulo cian discontinuo** del tamaño exacto del bloque, anclado al tile que tienes debajo del cursor → te indica dónde y con qué tamaño se pegará.
- **Ctrl + V**: pega el bloque con la esquina superior-izquierda en el tile bajo el cursor. Si el bloque se sale del mapa, se recorta sin error.
- **Esc**: limpia la selección y vacía el portapapeles (desaparece el rectángulo cian).

---

## Pestaña Sprites

Edita los patrones de sprites del MSX1 con sus propios sets para gestionar frames de animación.

### Modos

- **8x8 mode**: 32 sprites individuales de 8×8 píxeles, con un color por sprite.
- **16x16 mode**: 8 sprites compuestos de 16×16, formados por 4 sub-sprites en el orden hardware MSX: **TL=0, BL=1, TR=2, BR=3** (top-left, bottom-left, top-right, bottom-right). Comparten un único color por composite.

### Sets

Un *set* de sprites es un bloque completo de 32 patrones + 32 colores (288 bytes en VRAM). Se usan para animación: preparas frames en sets distintos y haces swap en runtime.

- **Selector de set**: dropdown con los sets disponibles.
- `New`: crea un set nuevo en blanco.
- `Rename`: cambia el nombre del set actual.
- `Delete`: elimina el set actual (siempre queda al menos uno).

### Editor del sprite

- **Click izq**: pinta con el color del sprite (uno por sprite, MSX hardware).
- **Click der**: borra (pone el píxel a 0 = transparente).
- **Paleta**: 16 colores TMS9918A. El primer color (a cuadros gris) representa **transparente** y se usa como goma. Click en otros colores cambia el color del sprite. En modo 16×16, el color cambia en los 4 sub-sprites a la vez.
- **Color 0 = goma de borrar** (no es un color "negro pintado", es la ausencia de pixel).

### Operaciones

| Botón | 8×8 | 16×16 |
|---|---|---|
| `Flip H` | Voltea horizontalmente el sprite | Voltea cada sub-sprite y permuta TL↔TR, BL↔BR |
| `Flip V` | Voltea verticalmente el sprite | Voltea cada sub-sprite y permuta TL↔BL, TR↔BR |
| `Rotate` | Rota 90° el sprite | (no disponible en 16×16) |
| `Clear` | Limpia el sprite | Limpia los 4 sub-sprites |
| `Invert` | Invierte cada bit | Invierte todos los sub-sprites |
| `Copy` | Copia el sprite (patrón + color) al portapapeles | Copia los 4 sub-sprites + colores |
| `Paste` | Pega sobre el sprite actual | Pega los 4 sub-sprites sobre el composite actual |
| `Prev` / `Next` | Sprite anterior / siguiente |  |

**Nota**: el portapapeles guarda el modo en que se copió. Si copias en 8×8 y cambias a 16×16, te avisará al pegar.

### Vista previa y rejilla

- **Preview**: pequeño canvas que muestra el sprite tal y como se renderiza con su color real sobre fondo cuadriculado.
- **Rejilla "All sprites"**: muestra los 32 (8×8) o 8 (16×16) sprites del set, click para seleccionar.

### Atajos en la pestaña Sprites
- **Ctrl + C** copia el sprite actual.
- **Ctrl + V** pega sobre el sprite actual.

### Importar / exportar

| Botón | Formato |
|---|---|
| `Export (.json)` | JSON con todos los sets, patrones y colores |
| `Import (.json)` | Carga un JSON previamente exportado |
| `Export C` | C arrays `g_SpritePatterns[32][8]` + `g_SpriteColors[32]` |
| `Export .bin` | 288 bytes (256 patrones + 32 colores) listos para VRAM |

---

## Pestaña Mixed (editor mixto)

Pensada para componer escenas: pegas un bloque del mapa de fondo y pintas sprites encima a precisión de 8 píxeles, igual que verás en pantalla en el juego.

### Lienzo de composición

- **Tamaño**: 8×8 tiles (64×64 píxeles nativos), renderizados a escala ×8 → 512×512 px en pantalla.
- **Fondo**: bloque de tiles cargado del portapapeles del mapa.
- **Capa de sprites**: encima del bloque, posicionables.

### Cargar el bloque de fondo

1. En la pestaña **Tiles & Map**, selecciona un bloque exacto de **8×8 tiles** con Shift+drag.
2. Pulsa **Ctrl + C** para copiarlo.
3. Cambia a la pestaña **Mixed**.
4. Pulsa el botón **"Cargar bloque 8x8 del portapapeles"**.

Si el bloque copiado no es 8×8, el editor te avisa y no carga nada.

Botones:
- `Cargar bloque 8x8 del portapapeles`: ver arriba.
- `Quitar bloque`: borra los tiles de fondo (los sprites colocados se mantienen).
- `Quitar sprites`: borra todos los sprites colocados (el bloque se mantiene).

### Panel "Sprite a colocar" (a la derecha del lienzo)

- **Set**: dropdown que comparte los sets de la pestaña Sprites. Cambiarlo aquí no toca lo que tengas en Sprites.
- **Modo 8x8 / 16x16**: independiente del modo de la pestaña Sprites.
- **Picker**: rejilla con los sprites del set seleccionado en el modo elegido. Click selecciona (rectángulo amarillo).

### Colocar sprites en el lienzo

- **Click izq sobre el lienzo**: coloca el sprite seleccionado, con la **esquina superior-izquierda alineada al tile de 8×8** bajo el cursor (snap automático).
- **Hover sobre el lienzo**: ves un fantasma a 50 % de opacidad + outline cian discontinuo en el tile donde caería el sprite si pulsas. Sigue al cursor pero está siempre alineado a la rejilla 8×8.
- **Click der sobre un sprite colocado**: lo elimina (toma el sprite encima del cursor, el más reciente colocado si hay solapamiento).

Puedes colocar el **mismo patrón** múltiples veces; el hardware MSX permite hasta 32 sprites en pantalla, todas instancias de algún patrón.

### Editor del sprite (en la propia pestaña Mixed)

Bajo el picker hay un **editor de píxeles del sprite seleccionado**. **Edita el patrón compartido** (el mismo dato que la pestaña Sprites), así que cualquier cambio se ve:
- En todas las instancias colocadas en este lienzo.
- En la pestaña Sprites.
- En cualquier export que generes.

Controles:
- **Click izq**: pinta con el color del sprite.
- **Click der**: borra.
- **Paleta MSX1** debajo: click cambia el color del sprite. En 16×16 cambia el color de los 4 sub-sprites.
- En 16×16 verás líneas finas separando los 4 sub-sprites (8×8 cada uno) para orientarte.

### Lista "Sprites colocados"

- Cada fila: **#id · nombre del set · modo · número de sprite · (x, y) en píxeles**.
- **Click en la fila**: selecciona ese sprite (set + idx + modo) en el editor para retocarlo. Útil cuando has colocado uno y quieres modificar su patrón sin perder dónde lo pusiste.
- **Botón X**: elimina la instancia colocada (no toca el patrón).

---

## Restricciones de hardware MSX1 (TMS9918A)

El editor refleja las limitaciones reales del VDP:

| Constraint | Valor |
|---|---|
| Resolución de pantalla | 256 × 192 píxeles |
| Tamaño de tile | 8 × 8 píxeles |
| Tiles por pantalla | 32 × 24 = 768 |
| Tabla de patrones | 3 bancos × 256 patrones |
| Colores por fila de tile | 2 (FG + BG) |
| Paleta | 16 colores fijos (no reprogramable) |
| Sprites 8×8 | 32 máximo |
| Sprites 16×16 | 8 máximo (4 sub-sprites cada uno) |
| Colores por sprite | 1 |
| Sprites por scanline | 4 máximo |

**Aviso 2-color por fila**: el editor de tile lo destaca en rojo cuando una fila tiene más de 2 colores. En MSX1 real esa fila se renderizará incorrectamente.

**Sprites por scanline**: el editor *no* lo valida; queda como restricción que cumple el código del juego en runtime.

---

## Formatos de exportación

### Tileset
- **PNG** 256×64 (32 columnas × 8 filas de tiles 8×8). Compatible con MSXimg / nMSXtiles.

### Mapa de pantalla
- **JSON**: `{ "width": 32, "height": 24, "tiles": [768 enteros] }`. Para guardar / cargar proyecto.
- **C array**: `static const u8 g_ScreenLayout[768] = { ... };` listo para incluir en proyectos MSXgl.
- **Binary (.bin)**: 768 bytes de índices de tile, carga directa a VRAM con `VDP_WriteVRAM_16K`.

### Sprites
- **JSON**: todos los sets, cada uno con nombre, 32 patrones (8 bytes cada uno) y 32 bytes de color.
- **C array**: `g_SpritePatterns[32][8]` (256 bytes de patrón) + `g_SpriteColors[32]` (32 bytes de color) — del set seleccionado.
- **Binary (.bin)**: 288 bytes contiguos (256 patrón + 32 color), listos para VRAM.

### Mixed
La pestaña Mixed actualmente es una herramienta de previsualización/composición; no exporta un formato propio. Los datos viven donde corresponde:
- El bloque de fondo es un fragmento del mapa (exportable desde Tiles & Map).
- Los patrones de sprites son los del set (exportables desde Sprites).
- Las posiciones colocadas no se persisten todavía → si las necesitas para tu juego, anótalas a mano o usa tu propia tabla de posiciones en código.

---

## Atajos de teclado

### Pestaña Tiles & Map
| Atajo | Acción |
|---|---|
| `Shift + arrastrar` | Selección rectangular en el mapa |
| `Ctrl + C` | Copiar bloque seleccionado |
| `Ctrl + V` | Pegar bloque en la posición del cursor (con preview) |
| `Esc` | Limpiar selección y portapapeles |

### Pestaña Sprites
| Atajo | Acción |
|---|---|
| `Ctrl + C` | Copiar sprite actual al portapapeles |
| `Ctrl + V` | Pegar sobre el sprite actual |

### Pestaña Mixed
*Sin atajos específicos; uso con ratón.*

### Generales
| Atajo | Acción |
|---|---|
| `Ctrl + R` / `F5` | Recargar la WebView (modo dev) |

Los atajos se ignoran si el foco está en un campo de texto o textarea.

---

## Resolución de problemas

### Los acentos / caracteres especiales se ven mal
El HTML declara `<meta charset="UTF-8">`. Si ves caracteres como `Â` o `Ã³`, significa que algún tooling intermedio guardó el archivo en otra codificación. Asegúrate de servir el HTML como UTF-8.

### Pego un bloque y "no entra" en Mixed
El editor mixto exige exactamente **8×8 tiles**. Si tu selección era 12×7, el botón te avisará. Vuelve al mapa y selecciona exactamente 8×8.

### Modifico un sprite en Mixed y se cambia también en Sprites
Es lo esperado: el patrón es el mismo dato. Si quieres una variante, antes de editar copia el sprite (Sprites → Copy) y pégalo en otro slot, o crea un set nuevo y trabaja allí.

### El binario no encuentra WebView2
Windows 11 lo trae preinstalado. En Windows 10, instala "Microsoft Edge WebView2 Runtime" desde la web de Microsoft (https://developer.microsoft.com/en-us/microsoft-edge/webview2/), o usa el instalador NSIS que lo descarga automáticamente si falta.

### Tarda mucho en compilar desde fuente
La primera build de Tauri descarga ~385 crates y compila todo (~2 min). Las siguientes builds incrementales tardan unos segundos.

---

*MSXJuanEditor — herramienta de assets para [puyopuyoMSX1](https://github.com/antxiko/puyopuyoMSX1) y otros proyectos MSX1.*
