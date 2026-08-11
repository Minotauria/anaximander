# Architecture

The whole app is one HTML file: markup, CSS, and JavaScript in a single `<script>` block at the bottom. No build step, no bundler, no module system. This document maps out how it's organised internally so you can find what you need.

## File layout

The HTML file is roughly:

1. `<head>` — fonts (Lora, IBM Plex Mono via Google Fonts), inline `<style>` block (~400 lines of CSS).
2. `<body>` — splash launch page (map cards + begin tiles), burger + drawer (map navigation), main canvas, toolbar + right tool panel, modals, hidden file inputs.
3. `<script>` — all logic.

There is intentionally no separation into modules. The trade-off is faster iteration and trivial deployment (drag-and-drop on any static host) at the cost of a long file. Use your editor's symbol search.

## State

The app's entire state lives in one global `state` object, persisted to `localStorage` under the key `anax_v7`.

```
state = {
  maps: [Map],
  currentMapId: string,
  folders: [Folder],     // legacy (pre-v11 "projects") — preserved in saves, no UI
  ai: { enabled, apiKey, model, provider,   // global AI config
        ollamaUrl, ollamaModel },           // Ollama fields parked, unused
}
```

A `Map` is:

```
{
  id: string,
  name: string,
  updatedAt: number,      // last-edited stamp; feeds the splash cards
  nodes: [Node],
  edges: [Edge],
  bg: string,             // canvas background colour
  nextId: number,         // monotonic id counter
  history: [Snapshot],    // undo/redo stack for this map
  histIdx: number,
  legend: { shapes, edges, colours, textWords },  // per-map meaning key; feeds AI prompts
}
```

A `Node`:

```
{
  id, x, y, label, labelHtml, urls, primaryUrl,
  color, shape, font, notes, image, outlined,
  scale, fontSize, palKey, textAlign, collapsed,
  vx, vy,                 // physics velocity (used by layout animations)
  imgCaption, textColor   // textColor authoritative when set
}
```

An `Edge`:

```
{
  id, from, to, style, color, thickness,
  keyframes: [{x,y}],     // edge bend points
  label,
  branchOrigin?: {edgeId, kfIdx}  // for "branch from keyframe"
}
```

Coordinates are world-space (not screen-space). Screen-space is computed via the global transform `T = {x, y, s}` (translate + scale). Helpers `w2s(x,y)` and `s2w(x,y)` convert between them.

## Render loop

The app uses an always-running `requestAnimationFrame` loop (`animate`). Each frame it:

1. Checks layout animations (`layoutAnim`) and steps them.
2. Checks spawn animations (`spawnAnim` Map) — these auto-clean when complete.
3. If `dirty` or any animation/drag is active, calls `render()`.
4. Schedules the next frame.

`render()` clears the canvas, applies the world-to-screen transform, draws edges (with hover/branch/keyframe handling), then nodes, then UI overlays (selection halos, `+` buttons, drag previews, ghost nodes).

`dirty = true` is set whenever something changes that needs redrawing. Most interaction handlers set it after mutating state.

## Node rendering

`drawNode(n, sel)` is the workhorse. It branches by shape:

- **Standard shapes** (rounded-rect, pill, longpill, circle, ellipse, diamond, parallelogram, hex) — fill + stroke + bevel gradient (capped at 14px from the shape's true apex — the gradient origin is computed per shape), then `drawRichLabel`.
- **Image shapes** — `drawNode` returns early after rendering the image and optional caption.
- **Text shapes** — no fill or stroke, just text. Returns early. Uses `n.textColor || n.color` so it always has a colour.

`drawRichLabel(n)` parses `n.labelHtml` via `parseHtmlRuns` into an array of runs (each run carrying bold/italic/underline/strike/colour/size), wraps lines, and draws them. Underline and strikethrough are drawn manually (canvas doesn't support them natively).

## Layout algorithms

Two deterministic layouts:

- **`computeRadialTargets(map)` (untangle)** — BFS tree from `map.nodes[0]`, leaf-count-weighted angular wedges, ring radius depth-based. Root stays put; everything orbits.
- **`computeTreeTargets(map)` (tidy up)** — Reingold-Tilford-style left-to-right. Subtree height computed bottom-up; siblings stacked vertically; depth × LGAP horizontal march.

Both produce a `Map<nodeId, {x,y}>` of target positions. `animateLayout(targets)` then snapshots the *from* positions, sets `layoutAnim`, and the render loop interpolates over ~650ms with `easeInOutCubic`.

Cycles in the graph are handled via BFS with a `visited` set. Disconnected/orphan nodes stay where they are (their target is their current position).

## Spawn animation

When a child is created, `startSpawnAnim(childId, fromX, fromY)` registers an entry in the `spawnAnim` Map. `drawNode` checks this Map at the top of each draw, and if there's an active entry, lerps position, scale (0.5 → 1.0), and alpha (0 → 1) over ~380ms. Edges to spawning children fade in concurrently via `drawEdge`'s alpha override.

Text-shape nodes don't visibly animate because their early-return branch in `drawNode` runs before the visible portion of the slide-out — this is intentional and was kept by user preference.

## Persistence

`hardSave()` writes `state` to localStorage. `softSave()` debounces to 800ms. Most state mutations call `softSave()`; explicit user actions (commit edit, delete, layout) call `hardSave()` via `pushCanvasHistLocal`.

`pushCanvasHistLocal(label)` snapshots `nodes` and `edges` to the per-map history (max 30 entries, FIFO). Undo and redo restore from these snapshots. Dragging coalesces into a single history entry via `schedCanvasHistLocal`.

## Import / export

- **JSON** — full state dump, round-trippable.
- **Markdown** — flattens the BFS tree into nested bullets.
- **PNG / JPG** — canvas snapshot at current zoom.
- **`.mm` import** — `parseCoggleMm(xmlText, baseName)` parses Coggle / Freemind XML via `DOMParser`. Handles cross-link nodes (`X_COGGLE_JOINEDTO` becomes an edge-only). Position attributes preserved if present, otherwise fanned around parents radially.

## AI features

Optional, off by default. Global config lives in `state.ai` (provider, API key, model); per-map context in `map.legend`. Both are defaulted in `loadState`. Config is edited only via the splash settings gear (`#sp-gear` → `#ai-ov`); the canvas keeps in-map actions (suggestions, new from prompt) in the right panel's AI group. Calls go straight from the browser to `api.anthropic.com` (via the `anthropic-dangerous-direct-browser-access` header) — no backend, key stored only in localStorage.

- **Suggest children** — `Tab` with a node selected calls `requestAiSuggestions()`, which streams candidates (`streamAiSuggestions`) and renders them as ghost nodes to accept or dismiss. `Esc` cancels the request or dismisses ghosts.
- **Generate from prompt** — the `#gen-ov` modal takes a topic and produces a whole starter map through the same API path.
- **Legend** — `map.legend` assigns meanings to shapes, edge styles, and colours; it is serialised into AI prompts so suggestions respect the map's semantics.

An Ollama provider skeleton exists in the config object but is parked and unused.

## Adding a feature

Some pointers:

- **New node shape** → add a draw branch in `drawNode`, add hit-testing in `nodeAt`, add a button in the shape picker (`pickerFor` UI).
- **New keyboard shortcut** → add a handler at the top of the `keydown` listener (search for `e.key === 'Escape'` to find it).
- **New layout algorithm** → write a `computeXTargets(map)` returning a `Map<nodeId, {x,y}>`, then `pushCanvasHistLocal(label); animateLayout(computeXTargets(map))`. Wire to a menu item.
- **New export format** → add a button in the export submenu (`#sub-export` in markup), wire a click handler that walks `cmap()` and produces the output.
- **New persistence backend** → replace `hardSave` and the load path in `loadState`. The state shape is plain JSON-serialisable.

## What's deliberately not modular

- No build step, no bundler, no transpilation, no tests.
- Single file means single download, single deploy, no path dependencies.
- All state is global within the script. No classes, mostly free functions.
- Naming is short (`cmap`, `nrx`, `w2s`) because the file is long and the same names appear hundreds of times.

This is a personal tool that grew. It is not idiomatic modern frontend code. It is fast, hackable, and dependency-free.

## Known limitations

- Performance degrades past ~500 nodes (full re-render every frame, no spatial index).
- No mobile / touch support.
- No collaboration / multi-user.
- LocalStorage is per-browser; switching machines means manual JSON export/import.
- Imported `.mm` files don't always lay out beautifully — the Coggle position model doesn't fully translate. Run *untangle* afterwards.

Unbuilt ideas are banked in `CLAUDE.md` — that list is the only one maintained.
