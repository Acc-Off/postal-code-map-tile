# postal-code-map-tile

A repository for converting the minimap used in [postal-code-map](https://github.com/Acc-Off/postal-code-map) into Leaflet-compatible tiles.

> Original map by Virus_City —
> [Release Postal Code Map / Minimap | New & Improved v1.3](https://forum.cfx.re/t/release-postal-code-map-minimap-new-improved-v1-3/147458)

## Directory Structure

| Path | Description |
|------|-------------|
| `minimap/` | Source minimap images (`minimap_sea_row_col.png` format, 6 files in a 3-row × 2-col grid) |
| `public/tiles/` | Generated tiles (`{z}/{x}/{y}.png`) |
| `public/metadata.json` | Metadata produced during tile generation |
| `public/index.html` | Local Leaflet viewer for verification |

## Publishing Tiles

The generated output under `public/` is hosted on [**Cloudflare Pages**](https://postal-code-map-tile.pages.dev/).  
Tile URL format:

```
https://postal-code-map-tile.pages.dev/tiles/{z}/{x}/{y}.png
```

## Using with Leaflet

### CRS / World Bounds

These tiles use a custom CRS based on the in-game coordinate system (Los Santos), not geographic coordinates.

| Parameter | Value |
|-----------|-------|
| CRS | Custom transform based on `L.CRS.Simple` |
| topLeft | `(-4140, 8400)` |
| bottomRight | `(4860, -5100)` |
| tileSize | `256` |
| maxZoom | `6` |

### Example (JavaScript)

```js
const mapConfig = {
  image: [6144, 9216],
  topLeft: [-4140, 8400],
  bottomRight: [4860, -5100],
  tileSize: 256,
  maxZoom: 6,
};

const crs = Object.create(L.CRS.Simple);
const scaleFactor = Math.pow(2, mapConfig.maxZoom);
const u = mapConfig.image[0] / ((mapConfig.bottomRight[0] - mapConfig.topLeft[0]) * scaleFactor);
const d = mapConfig.image[1] / ((mapConfig.bottomRight[1] - mapConfig.topLeft[1]) * scaleFactor);
crs.transformation = new L.Transformation(u, -u * mapConfig.topLeft[0], d, -d * mapConfig.topLeft[1]);
crs.infinite = false;
crs.scale = (zoom) => Math.pow(2, zoom);
crs.zoom  = (scale) => Math.log(scale) / Math.LN2;

const map = L.map('map', { crs, maxZoom: mapConfig.maxZoom });

L.tileLayer('https://postal-code-map-tile.pages.dev/tiles/{z}/{x}/{y}.png', {
  tileSize: mapConfig.tileSize,
  maxNativeZoom: mapConfig.maxZoom,
  noWrap: true,
}).addTo(map);
```

> In Leaflet, `lat` maps to the Y axis and `lng` to the X axis.  
> Pass in-game coordinates `(x, y)` as `L.latLng(y, x)`.

## Using with lb-phone

Add an entry to `Config.CustomMaps` in `lb-phone/config/config.lua` to make this tile available in the lb-phone Maps app.

```lua
Config.CustomMaps = {
    {
        label       = "Postal Code Map",
        url         = "https://postal-code-map-tile.pages.dev/tiles/{z}/{x}/{y}.png",
        center      = { 1650, 450 },
        topLeft     = { -4140, 8400 },
        bottomRight = { 4860, -5100 },
        resolution  = { 6144, 9216 },
        zoom = { default = 3, max = 6, min = 0 },
        styles = {
            { name = "render", background = "#000000" },
        },
    },
    //-- Default Maps below for reference --
    {
        label       = "Los Santos",
        url         = "https://assets.loaf-scripts.com/map-tiles/gtav/main/{layer}/{z}/{x}/{y}.jpg",
        center      = { 1650, 450 },
        topLeft     = { -4140, 8400 },
        bottomRight = { 4860, -5100 },
        resolution  = { 16384, 24576 },
        zoom        = { default = 3, max = 6, min = 2 },
        styles = {
            { name = "render", background = "#0d2b4f" },
            { name = "game",   background = "#384950" },
            { name = "print",  background = "#4eb1d0" },
        },
    },
    {
        label       = "Cayo Perico",
        url         = "https://assets.loaf-scripts.com/map-tiles/gtav/cayo-perico/{layer}/{z}/{x}/{y}.jpg",
        center      = { -5150, 4700 },
        topLeft     = { 3700, -4150 },
        bottomRight = { 5700, -6150 },
        resolution  = { 10000, 10000 },
        zoom        = { default = 2, max = 6, min = 0 },
        styles = {
            { name = "render", background = "#0d2b4f" },
            { name = "game",   background = "#384950" },
            { name = "print",  background = "#4eb1d0" },
        },
    }
}
```

---

## Tile Generation Script

### `build-minimap-tiles.ps1`

Reads the 6 PNG files from `minimap/` and performs the following steps:

1. Parses row/column indices from filenames (`minimap_sea_row_col.png`)
2. Combines all 6 images into a single 2-col × 3-row composite at `public/combined/minimap_combined.png`
3. Slices 256 px tiles for zoom levels 0–6 and saves them to `public/tiles/{z}/{x}/{y}.png`
4. Writes metadata to `public/metadata.json`

```powershell
# Run with defaults
./build-minimap-tiles.ps1

# Specify parameters explicitly
./build-minimap-tiles.ps1 -SourceDir ./minimap -OutputDir ./public -TileSize 256
```

Requires only .NET's `System.Drawing` — no external dependencies (PowerShell 5.1+, Windows).
