# Excalidraw JSON Schema Reference

## Color Palette

### Primary Colors
| Purpose | Color | Hex |
|---------|-------|-----|
| Main Title | Deep Blue | `#1e40af` |
| Subtitle | Medium Blue | `#3b82f6` |
| Body Text | Dark Gray | `#374151` |
| Emphasis | Orange | `#f59e0b` |
| Success | Green | `#10b981` |
| Warning | Red | `#ef4444` |

### Background Colors
| Purpose | Color | Hex |
|---------|-------|-----|
| Light Blue | Background | `#dbeafe` |
| Light Gray | Neutral | `#f3f4f6` |
| Light Orange | Highlight | `#fef3c7` |
| Light Green | Success | `#d1fae5` |
| Light Purple | Accent | `#ede9fe` |

## Element Types

### Rectangle
```json
{
  "type": "rectangle",
  "id": "unique-id",
  "x": 100,
  "y": 100,
  "width": 200,
  "height": 80,
  "strokeColor": "#1e40af",
  "backgroundColor": "#dbeafe",
  "fillStyle": "solid",
  "strokeWidth": 2,
  "roughness": 1,
  "opacity": 100,
  "roundness": { "type": 3 }
}
```

### Text
```json
{
  "type": "text",
  "id": "unique-id",
  "x": 150,
  "y": 130,
  "text": "Content here",
  "fontSize": 20,
  "fontFamily": 5,
  "textAlign": "center",
  "verticalAlign": "middle",
  "strokeColor": "#1e40af",
  "backgroundColor": "transparent"
}
```

### Arrow
```json
{
  "type": "arrow",
  "id": "unique-id",
  "x": 300,
  "y": 140,
  "width": 100,
  "height": 0,
  "points": [[0, 0], [100, 0]],
  "strokeColor": "#374151",
  "strokeWidth": 2,
  "startArrowhead": null,
  "endArrowhead": "arrow"
}
```

### Ellipse
```json
{
  "type": "ellipse",
  "id": "unique-id",
  "x": 100,
  "y": 100,
  "width": 120,
  "height": 120,
  "strokeColor": "#10b981",
  "backgroundColor": "#d1fae5",
  "fillStyle": "solid"
}
```

### Diamond
```json
{
  "type": "diamond",
  "id": "unique-id",
  "x": 100,
  "y": 100,
  "width": 150,
  "height": 100,
  "strokeColor": "#f59e0b",
  "backgroundColor": "#fef3c7",
  "fillStyle": "solid"
}
```

### Line
```json
{
  "type": "line",
  "id": "unique-id",
  "x": 100,
  "y": 100,
  "points": [[0, 0], [200, 100]],
  "strokeColor": "#374151",
  "strokeWidth": 2
}
```

## Full JSON Structure

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [
    // Array of elements
  ],
  "appState": {
    "gridSize": null,
    "viewBackgroundColor": "#ffffff"
  },
  "files": {}
}
```

## Font Family Values

| Value | Font Name |
|-------|-----------|
| 1 | Virgil (hand-drawn) |
| 2 | Helvetica |
| 3 | Cascadia |
| 4 | Assistant |
| 5 | Excalifont (recommended) |

## Fill Styles

- `solid` - Solid fill
- `hachure` - Hatched lines
- `cross-hatch` - Cross-hatched
- `dots` - Dotted pattern

## Roundness Types

- `{ "type": 1 }` - Sharp corners
- `{ "type": 2 }` - Slight rounding
- `{ "type": 3 }` - Full rounding (recommended)

## Element Binding

Binding is **bidirectional** and **required** for connected diagrams: each side must reference the other, or the link breaks. `boundElements` is `null` (or omitted) only when an element has nothing bound to it; once a shape holds a label or is touched by an arrow, it must be a populated array. Do not draw free text on top of a shape and call it a label -- bind it.

A shape's `boundElements` may hold multiple entries at once -- one text label plus any number of arrows:

```json
{
  "id": "shape-1",
  "boundElements": [
    { "type": "text", "id": "shape-1-label" },
    { "type": "arrow", "id": "arrow-1" }
  ]
}
```

### Text inside a container (labels)

The container lists the text; the text points back via `containerId` and uses centered alignment with a transparent background. Bound text is auto-centered/auto-sized by Excalidraw, so it does not need manual x-centering.

```json
{
  "type": "rectangle",
  "id": "container-id",
  "boundElements": [{ "type": "text", "id": "text-id" }]
}
```

```json
{
  "type": "text",
  "id": "text-id",
  "containerId": "container-id",
  "text": "Label",
  "textAlign": "center",
  "verticalAlign": "middle",
  "backgroundColor": "transparent",
  "autoResize": true,
  "lineHeight": 1.25,
  "originalText": "Label"
}
```

Do **not** add metadata fields (`frameId`, `index`, `versionNonce`, `rawText`) to bound elements -- they cause "invalid file" errors on excalidraw.com v0.17.0+. `boundElements` arrays, `containerId`, `startBinding`, and `endBinding` are valid binding fields and are safe to use.

## Arrow Binding

An arrow connecting two shapes binds to both. The arrow carries `startBinding`/`endBinding`, and **both** shapes list the arrow in their `boundElements` (this is the reciprocal half -- without it the binding is broken).

The arrow:

```json
{
  "type": "arrow",
  "id": "arrow-1",
  "startBinding": { "elementId": "source-shape-id", "focus": 0, "gap": 4 },
  "endBinding": { "elementId": "target-shape-id", "focus": 0, "gap": 4 },
  "startArrowhead": null,
  "endArrowhead": "arrow",
  "boundElements": null
}
```

Both shapes reference the arrow back:

```json
{ "id": "source-shape-id", "boundElements": [{ "type": "arrow", "id": "arrow-1" }] }
```
```json
{ "id": "target-shape-id", "boundElements": [{ "type": "arrow", "id": "arrow-1" }] }
```

Field meanings:
- `elementId` -- id of the shape this endpoint attaches to
- `focus` -- roughly -1..1, where the arrow aims along the shape (0 = center)
- `gap` -- px distance kept between the arrow endpoint and the shape edge

Use `startBinding`/`endBinding` in raw `.excalidraw` JSON. The `start`/`end` shorthand exists only in the `convertToExcalidrawElements` skeleton API and is invalid in saved files.

## Arrow Labels

An arrow label is text bound to the arrow via the same container mechanism as text-in-a-shape: the label sets `containerId` to the arrow id, and the arrow lists the label in `boundElements`.

```json
{ "id": "arrow-1", "type": "arrow", "boundElements": [{ "type": "text", "id": "arrow-1-label" }] }
```
```json
{
  "id": "arrow-1-label",
  "type": "text",
  "containerId": "arrow-1",
  "text": "calls",
  "textAlign": "center",
  "verticalAlign": "middle",
  "backgroundColor": "transparent"
}
```

When an arrow both connects shapes and carries a label, its `boundElements` holds only the label (the arrow is the connector, not a connection target), while the two shapes' `boundElements` hold the arrow.
