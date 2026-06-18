---
name: excalidraw-diagram
description: Generate Excalidraw diagrams from text content. Supports three output modes - Obsidian (.md), Standard (.excalidraw), and Animated (.excalidraw with animation order). Triggers on "Excalidraw", "diagram", "flowchart", "mind map", "visualize", "standard excalidraw", "animate".
metadata:
  version: 1.3.0
---

# Excalidraw Diagram Generator

Create Excalidraw diagrams from text content with multiple output formats.

## Output Modes

Select the output mode based on the user's trigger words:

| Trigger Words | Output Mode | File Format | Purpose |
|---------------|-------------|-------------|---------|
| `Excalidraw`, `diagram`, `flowchart`, `mind map` | **Obsidian** (default) | `.md` | Open directly in Obsidian |
| `standard excalidraw` | **Standard** | `.excalidraw` | Open/edit/share on excalidraw.com |
| `animate` | **Animated** | `.excalidraw` | Drag into excalidraw-animate to generate animations |

## Workflow

1. **Detect output mode** from trigger words (see Output Modes table above)
2. Analyze content - identify concepts, relationships, hierarchy
3. Choose diagram type (see Diagram Types below)
4. Generate Excalidraw JSON (add animation order if Animated mode)
5. Output in correct format based on mode
6. **Automatically save to current working directory**
7. Notify user with file path and usage instructions

## Output Formats

### Mode 1: Obsidian Format (Default)

**Output strictly in the following structure, with no modifications:**

```markdown
---
excalidraw-plugin: parsed
tags: [excalidraw]
---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠== You can decompress Drawing data with the command palette: 'Decompress current Excalidraw file'. For more info check in plugin settings under 'Saving'

# Excalidraw Data

## Text Elements
%%
## Drawing
\`\`\`json
{complete JSON data}
\`\`\`
%%
```

**Key points:**
- Frontmatter must include `tags: [excalidraw]`
- The warning message must be included in full
- JSON must be wrapped in `%%` markers
- Do not use any frontmatter settings other than `excalidraw-plugin: parsed`
- **File extension**: `.md`

### Mode 2: Standard Excalidraw Format

Output a plain JSON file that can be opened on excalidraw.com:

```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [...],
  "appState": {
    "gridSize": null,
    "viewBackgroundColor": "#ffffff"
  },
  "files": {}
}
```

**Key points:**
- `source` uses `https://excalidraw.com` (not the Obsidian plugin)
- Pure JSON, no Markdown wrapping
- **File extension**: `.excalidraw`

### Mode 3: Animated Excalidraw Format

Same as Standard format, but each element includes a `customData.animate` field to control animation order:

```json
{
  "id": "element-1",
  "type": "rectangle",
  "customData": {
    "animate": {
      "order": 1,
      "duration": 500
    }
  },
  ...other standard fields
}
```

**Animation order rules:**
- `order`: Playback sequence (1, 2, 3...) — lower numbers appear first
- `duration`: Drawing duration for the element in milliseconds, default 500
- Elements with the same `order` appear simultaneously
- Recommended sequence: title -> main framework -> connectors -> detail text

**Usage:**
1. Generate the `.excalidraw` file
2. Drag it into https://dai-shi.github.io/excalidraw-animate/
3. Click Animate to preview, then export as SVG or WebM

**File extension**: `.excalidraw`

---

## Diagram Types & Selection Guide

Choose the appropriate diagram type to maximize clarity and visual appeal.

| Type | Use Case | Approach |
|------|----------|----------|
| **Flowchart** | Step-by-step instructions, workflows, task sequences | Connect steps with arrows to show process flow |
| **Mind Map** | Concept brainstorming, topic categorization, idea capture | Radiate outward from a central node |
| **Hierarchy** | Org structures, content tiers, system decomposition | Build top-down or left-to-right node levels |
| **Relationship** | Dependencies, influences, interactions between elements | Use connectors with labels to show associations |
| **Comparison** | Side-by-side analysis of two or more options/viewpoints | Two-column or table layout with comparison dimensions |
| **Timeline** | Event progression, project milestones, model evolution | Plot key dates and events along a time axis |
| **Matrix** | Two-dimensional classification, priority mapping, positioning | Place items on an X-Y coordinate plane |
| **Freeform** | Loose content, brainstorming notes, early-stage collection | No structural constraints — freely place blocks and arrows |

## Design Rules

### Text & Format
- **All text elements must use** `fontFamily: 5` (Excalifont handwriting font)
- **Double quotes in text**: escape as `\"` in JSON text fields
- **Brackets must be ASCII**: use `(` `)` or `[` `]` only. Never use CJK/fullwidth bracket glyphs such as `「` `」`, `【` `】`, `（` `）`, `『` `』`, or `《` `》`. They render as foreign punctuation and look out of place in English diagrams.
- **Font size rules** (hard minimums — below these values text is unreadable at normal zoom):
  - Title: 20-28px (minimum 20px)
  - Subtitle: 18-20px
  - Body/labels: 16-18px (minimum 16px)
  - Minor annotations: 14px (only for unimportant supplementary notes, use sparingly)
  - **Absolutely no text below 14px**
- **Line height**: All text uses `lineHeight: 1.25`
- **Text centering estimation** (standalone text only): Standalone text elements (`containerId: null`) have no auto-centering; manually calculate the x coordinate:
  - Estimate text width: `estimatedWidth = text.length * fontSize * 0.5`
  - Centering formula: `x = centerX - estimatedWidth / 2`
  - Example: text "Hello" (5 chars, fontSize 20) centered at x=300 -> `estimatedWidth = 5 * 20 * 0.5 = 50` -> `x = 300 - 25 = 275`
  - **Bound text needs no centering math**: text with a `containerId` (a shape or arrow label) is auto-centered and auto-sized by Excalidraw. Do not apply this formula to bound text; just bind it (see Element Binding).

### Layout & Design
- **Canvas range**: 0-1200 x 0-800 is a starting default, not a hard cap. If content feels cramped, **grow the canvas** rather than compressing spacing -- a larger, well-spaced diagram reads far better than a tight one.
- **Minimum shape size**: Rectangles/ellipses containing text should be at least 120x60px
- **Spacing between connected nodes**: Leave at least 60px of clear space between two shapes joined by an arrow, so the arrow shaft has a visible run. Arrows squeezed into a 10-20px gap are nearly invisible.
- **Spacing between unconnected elements**: At least 40px gap between adjacent shapes that are not connected, so nothing overlaps or touches.
- **Never overlap**: Two shapes must never share screen space. When in doubt, add more gap, not less.
- **Clear hierarchy**: Use different colors and shapes to distinguish information levels
- **Graphic elements**: Use rectangles, circles, arrows, and other shapes to organize information
- **No Emoji**: Do not use any Emoji symbols in diagram text; use simple shapes (circles, squares, arrows) or color to create visual markers

### Color Palette

**Text colors (strokeColor for text):**

| Usage | Hex Value | Description |
|-------|-----------|-------------|
| Title | `#1e40af` | Deep blue |
| Subtitle/connectors | `#3b82f6` | Bright blue |
| Body text | `#374151` | Dark gray (no lighter than `#757575` on white background) |
| Emphasis/highlight | `#f59e0b` | Gold |

**Shape fill colors (backgroundColor, fillStyle: "solid"):**

| Hex Value | Semantic | Use Case |
|-----------|----------|----------|
| `#a5d8ff` | Light blue | Input, data source, primary nodes |
| `#b2f2bb` | Light green | Success, output, completed |
| `#ffd8a8` | Light orange | Warning, pending, external dependency |
| `#d0bfff` | Light purple | In progress, middleware, special items |
| `#ffc9c9` | Light red | Error, critical, alert |
| `#fff3bf` | Light yellow | Notes, decisions, planning |
| `#c3fae8` | Light teal | Storage, data, cache |
| `#eebefa` | Light pink | Analysis, metrics, statistics |

**Region background colors (large rectangle + opacity: 30, for layered diagrams):**

| Hex Value | Semantic |
|-----------|----------|
| `#dbe4ff` | Frontend/UI layer |
| `#e5dbff` | Logic/processing layer |
| `#d3f9d8` | Data/tooling layer |

**Contrast rules:**
- Text on white background must be no lighter than `#757575`, otherwise unreadable
- Use dark color variants on light fills (e.g., `#15803d` on light green, not `#22c55e`)
- Avoid light gray text (`#b0b0b0`, `#999`) on white backgrounds

Reference: [references/excalidraw-schema.md](references/excalidraw-schema.md)

## JSON Structure

**Obsidian mode:**
```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://github.com/zsviczian/obsidian-excalidraw-plugin",
  "elements": [...],
  "appState": { "gridSize": null, "viewBackgroundColor": "#ffffff" },
  "files": {}
}
```

**Standard / Animated mode:**
```json
{
  "type": "excalidraw",
  "version": 2,
  "source": "https://excalidraw.com",
  "elements": [...],
  "appState": { "gridSize": null, "viewBackgroundColor": "#ffffff" },
  "files": {}
}
```

## Element Template

Each element requires these fields. Do NOT add these metadata fields: `frameId`, `index`, `versionNonce`, `rawText` -- they cause "invalid file" errors on excalidraw.com v0.17.0+. Set `updated: 1` (not timestamps).

**Binding fields are required when elements connect, not forbidden.** `boundElements` is `null` only when nothing is bound to the element; when a shape holds a text label or is touched by an arrow, it must be a populated array. A text element's `containerId` is `null` only for standalone text; text bound inside a shape (or an arrow) sets it to that container's id. See the **Element Binding** section below -- binding is what attaches labels to boxes and arrows to shapes.

```json
{
  "id": "unique-id",
  "type": "rectangle",
  "x": 100, "y": 100,
  "width": 200, "height": 50,
  "angle": 0,
  "strokeColor": "#1e1e1e",
  "backgroundColor": "transparent",
  "fillStyle": "solid",
  "strokeWidth": 2,
  "strokeStyle": "solid",
  "roughness": 1,
  "opacity": 100,
  "groupIds": [],
  "roundness": {"type": 3},
  "seed": 123456789,
  "version": 1,
  "isDeleted": false,
  "boundElements": null,
  "updated": 1,
  "link": null,
  "locked": false
}
```

`strokeStyle` values: `"solid"` (solid line, default) | `"dashed"` (dashed line) | `"dotted"` (dotted line). Dashed lines are suitable for optional paths, async flows, weak associations, etc.

Text elements add:
```json
{
  "text": "Display text",
  "fontSize": 20,
  "fontFamily": 5,
  "textAlign": "center",
  "verticalAlign": "middle",
  "containerId": null,
  "originalText": "Display text",
  "autoResize": true,
  "lineHeight": 1.25
}
```

**Animated mode additionally requires** a `customData` field:
```json
{
  "id": "title-1",
  "type": "text",
  "customData": {
    "animate": {
      "order": 1,
      "duration": 500
    }
  },
  ...other fields
}
```

See [references/excalidraw-schema.md](references/excalidraw-schema.md) for all element types.

---

## Element Binding

Binding is what turns scattered shapes, text, and arrows into a connected diagram. Without it, text just floats on top of a colored square and arrows are loose lines that detach the moment anything moves. **Always bind**: every label belongs inside its shape, and every connector binds to the shapes it joins. Bindings are **bidirectional** -- both sides must reference each other, or the link silently breaks.

### Text inside a shape (labels)

To put a label inside a rectangle, ellipse, or diamond, create the text as a separate element bound to the shape -- do not just position free text on top of it.

- The **shape** lists the text in `boundElements`:
  ```json
  { "id": "box1", "type": "rectangle", "boundElements": [{ "type": "text", "id": "box1-label" }] }
  ```
- The **text** points back via `containerId` and uses centered alignment:
  ```json
  {
    "id": "box1-label",
    "type": "text",
    "containerId": "box1",
    "text": "User Service",
    "textAlign": "center",
    "verticalAlign": "middle",
    "backgroundColor": "transparent",
    "fontSize": 18,
    "fontFamily": 5,
    "autoResize": true,
    "lineHeight": 1.25,
    "originalText": "User Service"
  }
  ```

Bound text is auto-centered and auto-sized inside the shape, so you do **not** apply the standalone-text centering formula to it. Give the text roughly the shape's position; Excalidraw snaps it to the center.

### Arrows between shapes (connectors)

An arrow that connects two shapes must bind to both, so it stays attached when shapes move. The arrow gets `startBinding` and `endBinding`; **both** connected shapes list the arrow in their `boundElements`.

- The **arrow**:
  ```json
  {
    "id": "arrow-1",
    "type": "arrow",
    "startBinding": { "elementId": "box1", "focus": 0, "gap": 4 },
    "endBinding": { "elementId": "box2", "focus": 0, "gap": 4 },
    "startArrowhead": null,
    "endArrowhead": "arrow",
    "boundElements": null
  }
  ```
- **Both shapes** include the arrow in `boundElements` (alongside any text label):
  ```json
  { "id": "box1", "boundElements": [{ "type": "text", "id": "box1-label" }, { "type": "arrow", "id": "arrow-1" }] }
  ```
  ```json
  { "id": "box2", "boundElements": [{ "type": "text", "id": "box2-label" }, { "type": "arrow", "id": "arrow-1" }] }
  ```

`focus` is roughly -1..1 (0 = aim at the shape center); `gap` is the px distance kept from the shape's edge. Use `startBinding`/`endBinding` -- the `start`/`end` shorthand is only valid in the higher-level skeleton API, not in raw `.excalidraw` JSON.

### Labels on arrows

An arrow label is text bound to the arrow, exactly like text inside a shape -- the label gets `containerId: "<arrow-id>"` and the arrow lists it in `boundElements`. This keeps the label centered on the arrow instead of floating beside it.

```json
{ "id": "arrow-1", "type": "arrow", "boundElements": [{ "type": "text", "id": "arrow-1-label" }] }
```
```json
{ "id": "arrow-1-label", "type": "text", "containerId": "arrow-1", "text": "calls", "textAlign": "center", "verticalAlign": "middle", "backgroundColor": "transparent" }
```

Keep arrow labels short (one or two words) so they fit on the arrow.

See [references/excalidraw-schema.md](references/excalidraw-schema.md) for the full binding reference.

### Reference Example

[assets/example-flowchart.excalidraw](assets/example-flowchart.excalidraw) is a verified Standard-mode diagram that demonstrates every rule above: text bound inside each shape, arrows bound to both endpoints with reciprocal `boundElements`, bound arrow labels, ASCII brackets in node text (`(auth)`, `[core]`), and generous spacing. Mirror its structure when generating diagrams.

---

## Additional Technical Requirements

### Text Elements Handling
- The `## Text Elements` section in the Markdown **must be left empty** — only use `%%` as a separator
- The Obsidian Excalidraw plugin **automatically populates text elements** from the JSON data
- No need to manually list text content

### Coordinates & Layout
- **Coordinate system**: Origin (0,0) is at the top-left corner
- **Recommended range**: Keep all elements within 0-1200 x 0-800 pixels
- **Element IDs**: Each element needs a unique `id` (can be a string such as "title", "box1", etc.)

### Required Fields for All Elements

**IMPORTANT**: Do NOT include the metadata fields `frameId`, `index`, `versionNonce`, or `rawText` -- they cause "invalid file" errors on excalidraw.com v0.17.0+. Set `updated: 1` (not timestamps). `boundElements` is `null` for unbound elements but a populated array for bound ones (see **Element Binding**); these are valid binding fields, not the metadata fields above.

```json
{
  "id": "unique-identifier",
  "type": "rectangle|text|arrow|ellipse|diamond",
  "x": 100, "y": 100,
  "width": 200, "height": 50,
  "angle": 0,
  "strokeColor": "#color-hex",
  "backgroundColor": "transparent|#color-hex",
  "fillStyle": "solid",
  "strokeWidth": 2,
  "strokeStyle": "solid|dashed|dotted",
  "roughness": 1,
  "opacity": 100,
  "groupIds": [],
  "roundness": {"type": 3},
  "seed": 123456789,
  "version": 1,
  "isDeleted": false,
  "boundElements": null,
  "updated": 1,
  "link": null,
  "locked": false
}
```

### Text-Specific Properties
Text elements (type: "text") require additional properties (do NOT include `rawText`):
```json
{
  "text": "Display text",
  "fontSize": 20,
  "fontFamily": 5,
  "textAlign": "center",
  "verticalAlign": "middle",
  "containerId": null,
  "originalText": "Display text",
  "autoResize": true,
  "lineHeight": 1.25
}
```

### appState Configuration
```json
"appState": {
  "gridSize": null,
  "viewBackgroundColor": "#ffffff"
}
```

### files Field
```json
"files": {}
```

## Common Mistakes to Avoid

- **Text misalignment** — A standalone text element's `x` is the left edge, not the center. You must use the centering formula to calculate manually, or text will be off to one side
- **Free text floating on a shape** -- A label drawn as standalone text on top of a colored box is not connected to it. Bind it: set the text's `containerId` to the shape and add the text to the shape's `boundElements` (see Element Binding)
- **One-sided binding** -- Binding must exist on both ends. A text with `containerId` whose shape does not list it in `boundElements` (or vice versa) is a broken link and may render incorrectly
- **Element overlap**: Elements with similar coordinates can stack on top of each other. Keep at least 40px from unconnected neighbors and at least 60px between arrow-connected shapes; grow the canvas rather than compressing spacing
- **Insufficient canvas padding** — Do not place content flush against canvas edges. Leave 50-80px padding on all sides
- **Title not centered over diagram** — The title should be centered over the full width of the diagram below, not pinned at x=0
- **Unbound arrows** -- An arrow positioned near two boxes but with `startBinding`/`endBinding` set to `null` is not connected; it detaches the moment a box moves. Bind both ends and add the arrow to both shapes' `boundElements` (see Element Binding)
- **Floating arrow labels** -- Label text placed beside an arrow as standalone text is not attached. Bind it: set the label's `containerId` to the arrow and list it in the arrow's `boundElements`
- **Using `start`/`end` for arrow binding** -- Those keys are skeleton-API only and invalid in raw `.excalidraw` JSON. Use `startBinding`/`endBinding` with `elementId`/`focus`/`gap`
- **Arrow label overflow**: Long bound labels (e.g., "ATP + NADPH") can exceed short arrows. Keep labels short (one or two words) or increase arrow length
- **Insufficient contrast** — Light-colored text on a white background is nearly invisible. Text color must be no lighter than `#757575`; for colored text use dark variants
- **Font size too small** — Below 14px text is unreadable at normal zoom; body text minimum is 16px
- **Non-ASCII brackets**: Do not use `「」`, `【】`, `（）`, or other CJK/fullwidth bracket glyphs in text. Use ASCII `()` or `[]` only

## Implementation Notes

### Auto-save & File Generation Workflow

When generating Excalidraw diagrams, **the following steps must be performed automatically**:

#### 1. Choose the Appropriate Diagram Type
- Based on the characteristics of the user's content, refer to the "Diagram Types & Selection Guide" table above
- Analyze the core intent of the content and choose the most suitable visualization form

#### 2. Generate a Meaningful Filename

Choose the file extension based on the output mode:

| Mode | Filename Format | Example |
|------|----------------|---------|
| Obsidian | `[topic].[type].md` | `business-model.relationship.md` |
| Standard | `[topic].[type].excalidraw` | `business-model.relationship.excalidraw` |
| Animated | `[topic].[type].animate.excalidraw` | `business-model.relationship.animate.excalidraw` |

- Use concise English filenames with hyphens for readability

#### 3. Use the Write Tool to Auto-save the File
- **Save location**: Current working directory (auto-detect from environment)
- **Full path**: `{current_directory}/[filename].md`
- This allows flexible migration without hardcoded paths

#### 4. Ensure the Markdown Structure Is Exactly Correct
**Must generate in the following format** (no modifications allowed):

```markdown
---
excalidraw-plugin: parsed
tags: [excalidraw]
---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠== You can decompress Drawing data with the command palette: 'Decompress current Excalidraw file'. For more info check in plugin settings under 'Saving'

# Excalidraw Data

## Text Elements
%%
## Drawing
\`\`\`json
{complete JSON data}
\`\`\`
%%
```

#### 5. JSON Data Requirements
- Include the complete Excalidraw JSON structure
- All text elements use `fontFamily: 5`
- Escape double quotes as `\"` in JSON text fields
- JSON format must be valid and pass syntax checks
- All elements have a unique `id`
- Include `appState` and `files: {}` fields

#### 6. User Feedback and Confirmation
Report to the user:
- The diagram has been generated
- The exact save location
- How to view it in Obsidian
- Design choice rationale (what diagram type was chosen and why)
- Whether adjustments or modifications are needed

### Example Output Messages

**Obsidian mode:**
```
Excalidraw diagram generated!

Saved to: business-model.relationship.md

How to use:
1. Open this file in Obsidian
2. Click the MORE OPTIONS menu in the top-right corner
3. Select "Switch to EXCALIDRAW VIEW"
```

**Standard mode:**
```
Excalidraw diagram generated!

Saved to: business-model.relationship.excalidraw

How to use:
1. Go to https://excalidraw.com
2. Click the top-left menu -> Open -> select this file
3. Or drag and drop the file onto the excalidraw.com page
```

**Animated mode:**
```
Animated Excalidraw diagram generated!

Saved to: business-model.relationship.animate.excalidraw

Animation order: Title(1) -> Main framework(2-4) -> Connectors(5-7) -> Detail text(8-10)

To generate the animation:
1. Go to https://dai-shi.github.io/excalidraw-animate/
2. Click "Load File" and select this file
3. Preview the animation
4. Click "Export" to save as SVG or WebM
```
