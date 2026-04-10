---
name: excalidraw
description: Draw professional diagrams on a live Excalidraw canvas via MCP. Use when asked to create architecture diagrams, flowcharts, system maps, comparison visuals, or any visual explanation. Covers setup, 10 visual techniques, layout best practices, and the screenshot-iterate loop.
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

## 10 Visual Techniques

These are the building blocks for every diagram. Combine them to create professional visuals.

### 1. Layered Glow Effect

Stack 2-3 rectangles at decreasing opacity behind a shape to create depth.

```json
{"id": "glow-outer", "type": "rectangle", "x": 95, "y": 95, "width": 210, "height": 70,
 "backgroundColor": "#a5d8ff", "opacity": 20, "strokeColor": "transparent"},
{"id": "glow-inner", "type": "rectangle", "x": 98, "y": 98, "width": 204, "height": 64,
 "backgroundColor": "#a5d8ff", "opacity": 40, "strokeColor": "transparent"},
{"id": "main-box", "type": "rectangle", "x": 100, "y": 100, "width": 200, "height": 60,
 "text": "Core Service", "backgroundColor": "#a5d8ff"}
```

### 2. Color-Coded Zones

Low-opacity background rectangles group related elements. Use a free-standing text label at the top corner (never use `text` on the zone rectangle itself -- it centers and overlaps children).

```json
{"id": "zone-bg", "type": "rectangle", "x": 50, "y": 50, "width": 500, "height": 300,
 "backgroundColor": "#e3f2fd", "opacity": 30, "strokeColor": "#90caf9"},
{"id": "zone-label", "type": "text", "x": 70, "y": 60, "width": 200, "height": 30,
 "text": "Backend Services", "fontSize": 18}
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

### 7. Font Size Hierarchy

Create visual hierarchy with consistent sizing:
- 28px = diagram title
- 20px = section headers
- 16px = element labels (default)
- 14px = annotations, notes

### 8. Semantic Color Palette

Use color to encode meaning consistently:

| Color | Hex | Use for |
|-------|-----|---------|
| Blue | `#a5d8ff` | Primary / user-facing |
| Green | `#b2f2bb` | Success / approved / open |
| Red | `#ffc9c9` | Error / blocked / locked |
| Orange | `#ffd8a8` | Warning / async / feedback |
| Purple | `#d0bfff` | Integration / middleware |
| Yellow | `#ffec99` | Data / storage |
| Gray | `#dee2e6` | Infrastructure / background |

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
- **Between shapes:** 40-60px horizontal, 80-120px vertical (enough room for arrow labels)
- **Shape width:** `max(160, labelCharCount * 9)` to prevent text truncation
- **Shape height:** 60px single-line, 80px two-line labels
- **Zone padding:** 50px on all sides around contained elements

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

## Quality Checklist

Run this after every `batch_create_elements`. Take a screenshot and check:

1. **Text truncation** -- All label text fully visible? If not, increase shape width/height
2. **Overlap** -- Any shapes sharing space? Background zones must contain children with padding
3. **Arrow crossing** -- Do arrows cut through unrelated shapes? Route around with waypoints
4. **Arrow-label overlap** -- Labels at midpoint overlapping shapes? Shorten label or adjust path
5. **Spacing** -- At least 40px gap between elements
6. **Readability** -- Font size 16+ for body, 20+ for titles
7. **Zone labels** -- Not centered in background zones (use free-standing text instead)

If anything fails: stop, fix it, re-screenshot, then continue.

---

## What Works Well

- Rectangles, ellipses, diamonds -- all shape types
- Bound arrows with auto-routing
- Arrow labels
- Line styles (solid, dashed, dotted)
- Full color palette with fills and strokes
- Opacity for glow/shadow/zone effects
- Font sizes from 14px to 32px+
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
2. Plan your coordinate grid before writing any JSON (sketch tiers and x-positions)
3. Call `read_diagram_guide` for the built-in design reference
4. `batch_create_elements` -- create all shapes and arrows in one call
5. `set_viewport` with `scrollToContent: true` to auto-fit
6. `get_canvas_screenshot` -- run the Quality Checklist
7. Fix issues with `update_element`, re-screenshot
8. Export when clean: `export_to_image` (PNG/SVG) or `export_scene` (.excalidraw)

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
