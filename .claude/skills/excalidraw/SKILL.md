---
name: excalidraw
description: Draw professional diagrams on a live Excalidraw canvas via MCP. Use when asked to create architecture diagrams, flowcharts, system maps, comparison visuals, or any visual explanation. Covers setup, 10 visual techniques, design preferences, layout best practices, and the screenshot-iterate loop.
---

# Excalidraw Diagramming Skill

Turn text descriptions into professional diagrams on a live Excalidraw canvas. Your agent draws, screenshots its own work, and iterates until it looks right.

## Prerequisites: MCP Setup

Before using this skill, the Excalidraw MCP server must be installed and running.

### One-time install

```bash
git clone https://github.com/yctimlin/mcp_excalidraw && cd mcp_excalidraw
npm ci && npm run build
```

### Add to Claude Code

```bash
claude mcp add excalidraw -s user \
  -e EXPRESS_SERVER_URL=http://localhost:3000 \
  -- node /absolute/path/to/mcp_excalidraw/dist/index.js
```

Replace `/absolute/path/to/` with wherever you cloned the repo.

### Start the canvas (every session)

```bash
cd /path/to/mcp_excalidraw
PORT=3000 npm run canvas
```

Then open `http://localhost:3000` in your browser to see the live canvas.

### Verify it works

Once Claude Code starts, you should see `excalidraw/*` tools in your tool list. Try: "Draw a simple flowchart with three boxes."

---

## How It Works

```
You describe a diagram
  -> Agent plans the layout (coordinates, spacing, colors)
  -> batch_create_elements (draws everything in one call)
  -> get_canvas_screenshot (agent sees what it drew)
  -> Fixes any issues (overlap, truncation, bad arrows)
  -> Repeats until clean
  -> Exports (PNG, SVG, .excalidraw, or shareable URL)
```

The key insight: the agent can **see its own canvas** via screenshots. This creates a self-correcting feedback loop that produces clean diagrams without manual tweaking.

---

## Workflow (ALWAYS follow this order)

1. **Read the design guide** -- call `read_diagram_guide` first for color palette and sizing rules
2. **Plan the layout** -- decide on structure based on content (vertical cards, horizontal flow, grid)
3. **Create 3 variations** -- ALWAYS produce 3 different layout options before committing. Save each as a snapshot. Show a screenshot of each.
4. **Let the user choose** -- present all 3 with a short description of each
5. **Refine the chosen version** -- clean up based on feedback
6. **Final screenshot** -- verify everything looks right before exporting

### Variation Strategies

When creating 3 options, vary these dimensions:
1. **Layout direction** -- vertical stack vs horizontal flow vs grid
2. **Shape variety** -- all rectangles vs mixed (circles, diamonds, rectangles)
3. **Information density** -- minimal (just labels) vs detailed (labels + descriptions + examples)
4. **Visual personality** -- clean/professional vs playful/sketchy (roughness: 0 vs 1)

---

## Design Preferences

### Language
- **Always use plain language** -- no technical jargon unless explicitly asked
- Write labels as if explaining to someone who's never coded
- Descriptions should be conversational, not clinical

### Color Palette

Use orange as the dominant color. Gray for supporting text and backgrounds. Green for success/completion states. Only bring in purple/blue if the diagram has 4+ distinct categories.

| Role | Stroke | Fill |
|------|--------|------|
| **Primary** (Orange) | `#e8590c` | `#ffd8a8` |
| **Secondary** (Gray) | `#868e96` | `#e9ecef` |
| **Tertiary** (Green) | `#2f9e44` | `#b2f2bb` |
| **Accent** (Purple) | `#9c36b5` | `#eebefa` |
| **Accent** (Blue) | `#1971c2` | `#a5d8ff` |
| **Error/Blocked** (Red) | `#e03131` | `#ffc9c9` |
| **Data/Storage** (Yellow) | `#e8590c` | `#ffec99` |

### Visual Elements
- **Emojis as icons** -- use relevant emojis in labels (e.g., "🧠 Claude thinks", "🎨 Drawing appears")
- **Numbered badge circles** -- solid-filled circles with white numbers for step sequences
  - Use `roughness: 0` for clean badge circles
  - Badge size: 50x50px, font size 24, white text on colored fill
- **Punchy footer tagline** -- include a one-liner at the bottom that captures the "so what" of the diagram

### Font Rules
- **Helvetica** (`"helvetica"`) -- titles, headings, and labels (clean and professional)
- **Excalifont** (`"excalifont"`) -- descriptions, examples, secondary text (friendly hand-drawn feel)
- Do NOT use Lilita One (too cartoony) or Comic Shanns

| Element | Font | Size |
|---------|------|------|
| Diagram title | Helvetica | 36px |
| Section heading | Helvetica | 22px |
| Element label | Helvetica | 16px |
| Description text | Excalifont | 14px |
| Footer tagline | Excalifont | 18px |

### Text Handling (CRITICAL -- prevent overflow)
- **Minimum box width: 600px** for any box containing a heading + description
- **Always pre-wrap text** with manual `\n` line breaks -- never rely on auto-wrap
- **Max ~40 characters per line** for description text at 14px font
- **Max ~35 characters per line** for heading text at 22px font
- **20px+ padding** between text and box edges on all sides

---

## 10 Visual Techniques

These are the building blocks for every diagram. Combine them to create professional visuals.

### 1. Layered Glow Effect

Stack 2-3 rectangles at decreasing opacity behind a shape to create depth.

```json
{"id": "glow-outer", "type": "rectangle", "x": 95, "y": 95, "width": 210, "height": 70,
 "backgroundColor": "#ffd8a8", "opacity": 20, "strokeColor": "transparent"},
{"id": "glow-inner", "type": "rectangle", "x": 98, "y": 98, "width": 204, "height": 64,
 "backgroundColor": "#ffd8a8", "opacity": 40, "strokeColor": "transparent"},
{"id": "main-box", "type": "rectangle", "x": 100, "y": 100, "width": 200, "height": 60,
 "text": "Core Service", "backgroundColor": "#ffd8a8", "strokeColor": "#e8590c"}
```

### 2. Color-Coded Zones

Low-opacity background rectangles group related elements. Use a free-standing text label at the top corner (never use `text` on the zone rectangle itself -- it centers and overlaps children).

```json
{"id": "zone-bg", "type": "rectangle", "x": 50, "y": 50, "width": 500, "height": 300,
 "backgroundColor": "#e9ecef", "opacity": 30, "strokeColor": "#868e96"},
{"id": "zone-label", "type": "text", "x": 70, "y": 60, "width": 200, "height": 30,
 "text": "Backend Services", "fontSize": 18, "fontFamily": "helvetica"}
```

### 3. Bound Arrows with Labels

Arrows snap to shapes using element IDs. Labels describe the relationship.

```json
{"id": "svc-a", "type": "rectangle", "x": 100, "y": 100, "width": 160, "height": 60, "text": "API Gateway"},
{"id": "svc-b", "type": "rectangle", "x": 400, "y": 100, "width": 160, "height": 60, "text": "Database"},
{"type": "arrow", "x": 0, "y": 0, "startElementId": "svc-a", "endElementId": "svc-b", "text": "SQL"}
```

### 4. Line Styles as Meaning

- **Solid** = synchronous / primary flow
- **Dashed** (`strokeStyle: "dashed"`) = asynchronous / secondary
- **Dotted** (`strokeStyle: "dotted"`) = optional / planned

### 5. Diamond Decision Nodes

Classic flowchart branching with Yes/No paths.

```json
{"id": "decision", "type": "diamond", "x": 300, "y": 200, "width": 140, "height": 100, "text": "Auth OK?"},
{"id": "yes-path", "type": "rectangle", "x": 150, "y": 380, "width": 140, "height": 60, "text": "Proceed"},
{"id": "no-path", "type": "rectangle", "x": 450, "y": 380, "width": 140, "height": 60, "text": "Reject"},
{"type": "arrow", "x": 0, "y": 0, "startElementId": "decision", "endElementId": "yes-path", "text": "Yes"},
{"type": "arrow", "x": 0, "y": 0, "startElementId": "decision", "endElementId": "no-path", "text": "No"}
```

### 6. Mixed Shape Types

Use shape type to encode meaning:
- **Ellipse** = actors, users, external systems
- **Rectangle** = services, processes, components
- **Diamond** = decisions, conditions

### 7. Numbered Badge Circles

Solid-filled circles with white numbers for step sequences:

```json
{"id": "badge-1", "type": "ellipse", "x": 100, "y": 100, "width": 50, "height": 50,
 "backgroundColor": "#e8590c", "strokeColor": "#e8590c", "roughness": 0,
 "text": "1", "fontSize": 24, "textAlign": "center"}
```

### 8. Emoji Icons in Labels

Emojis render beautifully at any size. Use them to make labels scannable:

```json
{"type": "rectangle", "x": 100, "y": 100, "width": 200, "height": 60,
 "text": "🧠 Claude thinks", "backgroundColor": "#ffd8a8"}
```

### 9. Mermaid Conversion

Convert existing Mermaid diagrams to editable Excalidraw elements:

```
create_from_mermaid("graph TD\n  A[Start] --> B{Decision}\n  B -->|Yes| C[Do Thing]\n  B -->|No| D[Skip]")
```

After conversion, use `set_viewport` with `scrollToContent: true` to auto-fit the view.

### 10. Screenshot-Iterate Loop

The most important technique. After every batch of elements:

```
batch_create_elements -> get_canvas_screenshot -> evaluate -> fix -> re-screenshot
```

Never skip the screenshot step. Always verify before moving on.

---

## Layout Rules

These prevent the most common diagram problems.

### Spacing
- **Between shapes:** 30-40px gap between connected cards/zones
- **Vertical tiers:** 80-120px between rows (enough room for arrow labels)
- **Shape width:** `max(600, labelCharCount * 9)` for boxes with heading + description; `max(160, labelCharCount * 9)` for simple label boxes
- **Shape height:** 60px single-line, 80px two-line, 110px for step cards with descriptions
- **Zone padding:** 50px on all sides around contained elements

### Alignment
- All same-role elements should share the same x or y coordinate
- Center titles and footers relative to the card stack width
- Badges aligned at the same x offset within their cards

### Arrows
- Always use `startElementId` / `endElementId` to bind arrows to shapes (auto-routes to edges)
- Keep arrow labels under 12 characters
- If an arrow crosses through an unrelated shape, add a waypoint to route around it

**Curved arrows** (smooth arc over obstacles):
```json
{"type": "arrow", "x": 100, "y": 100, "points": [[0, 0], [50, -40], [200, 0]], "roundness": {"type": 2}}
```

**Elbowed arrows** (right-angle routing):
```json
{"type": "arrow", "x": 100, "y": 100, "points": [[0, 0], [0, -50], [200, -50], [200, 0]], "elbowed": true}
```

### Zone Labels (Critical)

Never put a `text` label on a large background rectangle. Excalidraw centers it in the middle of the zone, overlapping everything inside.

Instead, create a separate text element positioned at the top corner of the zone.

### Custom Element IDs

Always assign custom `id` values (e.g., `"id": "auth-svc"`) so arrows can reference them and you can update elements later by name.

---

## Sizing Defaults

| Element | Width | Height |
|---------|-------|--------|
| Step card (vertical) | 620px | 110px |
| Step card (horizontal) | 180px | 140px |
| Badge circle | 50px | 50px |

---

## Quality Checklist

Run this after every `batch_create_elements`. Take a screenshot and check:

1. **Text truncation** -- All label text fully visible? If not, increase shape width/height
2. **Text overflow** -- Text staying within box edges with 20px+ padding on all sides?
3. **Overlap** -- Any shapes sharing space? Background zones must contain children with padding
4. **Arrow crossing** -- Do arrows cut through unrelated shapes? Route around with waypoints
5. **Arrow-label overlap** -- Labels at midpoint overlapping shapes? Shorten label or adjust path
6. **Spacing** -- At least 30px gap between elements
7. **Readability** -- Font size 14+ for body, 22+ for headings, 36 for title
8. **Zone labels** -- Not centered in background zones (use free-standing text instead)
9. **Alignment** -- Same-role elements sharing coordinates?
10. **Font consistency** -- Helvetica for headings, Excalifont for descriptions?

If anything fails: stop, fix it, re-screenshot, then continue.

---

## Anti-Patterns (DO NOT do these)

- Single-line text that overflows the box edge
- Technical jargon without being asked for it
- Skipping the 3-variation step
- Forgetting to take a screenshot after building
- Using only one font family throughout
- Tiny fonts below 14px
- Boxes narrower than 600px when they contain multi-line text
- Forgetting to save snapshots before moving to the next variation

---

## What Works Well

- Rectangles, ellipses, diamonds -- all shape types
- Bound arrows with auto-routing
- Arrow labels
- Line styles (solid, dashed, dotted)
- Full color palette with fills and strokes
- Opacity for glow/shadow/zone effects
- Font sizes from 14px to 36px+
- Emoji icons (render great at any size, inside or outside shapes)
- Mermaid-to-Excalidraw conversion
- Export to .excalidraw, PNG, SVG, and shareable URL
- Canvas screenshots for self-verification

## Known Limitations

- **Images** -- MCP tool doesn't expose file path or base64 params. Workaround: drag images into the browser at localhost:3000
- **Freedraw** -- Needs point arrays the MCP tool can't accept. Workaround: draw freehand in browser
- **SVG import** -- `import_scene` only accepts .excalidraw JSON, not raw SVGs. Workaround: paste SVGs in the browser (native Excalidraw feature)

---

## Workflow: New Diagram

1. `clear_canvas` to start fresh
2. Call `read_diagram_guide` for the built-in design reference
3. Plan your coordinate grid (sketch tiers and x-positions)
4. Create **3 variations** -- save each as a snapshot, screenshot each
5. Present all 3, let user choose
6. Refine the chosen version
7. `get_canvas_screenshot` -- run the Quality Checklist
8. Fix issues with `update_element`, re-screenshot
9. Export when clean: `export_to_image` (PNG/SVG) or `export_scene` (.excalidraw)

## Workflow: Refine Existing Diagram

1. `describe_scene` to understand what's on the canvas (IDs, positions, labels)
2. Identify elements by ID or label text (not coordinates)
3. `update_element` to resize, recolor, or move
4. `get_canvas_screenshot` to verify
5. If an element won't update, try `unlock_elements` first

## Workflow: Snapshots (Undo Safety)

1. `snapshot_scene` with a name before risky changes
2. Make changes, screenshot to evaluate
3. `restore_snapshot` to roll back if needed

## Workflow: Export

- `.excalidraw` JSON (re-editable): `export_scene`
- PNG image: `export_to_image` with `format: "png"`
- SVG image: `export_to_image` with `format: "svg"`
- Shareable URL: `export_to_excalidraw_url` (uploads to excalidraw.com)

---

## Error Recovery

- **Elements not appearing?** They may be off-screen. Use `set_viewport` with `scrollToContent: true`
- **Arrow not connecting?** Verify element IDs exist with `get_element`
- **Canvas in a bad state?** `snapshot_scene` first, then `clear_canvas` and rebuild
- **Element won't update?** May be locked -- call `unlock_elements` first
- **Duplicate text elements appearing?** The frontend auto-syncs and can re-inject bound texts. Fix: use `query_elements` to find text elements with a `containerId`, delete the extras
