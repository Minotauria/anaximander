# Anaximander

A single-file HTML mind-mapping app. Botanical aesthetic, deterministic layouts, no build step, no server, no account.

Open the HTML file in a browser. That's it.

---

## What it does

- **Home screen** — launch state with map cards (live mini-preview, node count, edited time, colour chips). Begin a blank map, generate one from a prompt, or press `↵` to reopen the most recent. The settings gear here owns all AI config.
- **Spawn nodes** by dragging from the `+` button on a hovered node, or click for a shape picker.
- **Edit text** inline with bold / italic / underline / strikethrough / colour / size / alignment.
- **Connect & branch** edges between any two nodes; bend edges with mid-point keyframes; branch a new subtree off any keyframe.
- **Style** nodes with shape, fill colour (pastel / vivid / muted palettes), outline, font, image-as-node.
- **Notes** — attach long-form rich text to any node via a draggable, resizable side panel.
- **Links** — attach URLs (Google Docs, anything) to a node; double-click the link badge to open.
- **Layouts** — *untangle* (radial fan from root) and *tidy up* (left-to-right hierarchical tree). Both deterministic, smoothly animated.
- **Multi-select** — drag a marquee (or ⌘-click) to select several nodes; move and restyle them together.
- **AI assist** (optional, bring-your-own Anthropic API key) — `Tab` suggests children for the selected node; generate a starter map from a prompt; a per-map legend teaches the AI what your shapes and colours mean. Nothing is sent anywhere unless you turn it on.
- **Multiple maps** in the sidebar, organisable into folders.
- **History** — full undo/redo per map, with named snapshots.
- **Import .mm** files (Coggle / Freemind XML).
- **Export** to PNG, JPG, JSON, Markdown; share via URL.
- **Persistent** — autosaves to localStorage. Nothing leaves your browser.

## Getting started

Download the latest `anaximander_vN.html` (highest version number in the repo). Open it in any modern browser. Done.

To host: drop it on GitHub Pages, Netlify, or any static host. No build, no dependencies, no backend.

## Keyboard shortcuts

A reference is built into the app — open the ⋯ menu and click **shortcuts**. Highlights:

| | |
|--|--|
| `enter` | commit edit |
| `esc` | cancel edit / close panel |
| `⌘Z / ⌘⇧Z` | undo / redo |
| `⌘K` | search nodes |

## Tech

- Single HTML file, no dependencies.
- Canvas rendering.
- LocalStorage for persistence.
- DOMParser for `.mm` import.

For architecture and contribution notes, see [ARCHITECTURE.md](ARCHITECTURE.md).

## Name

Anaximander of Miletus drew the first known map of the world (~6th c. BCE) — a flat disc bordered by ocean, with regions and rivers laid out from a central point. This is software for that kind of thinking: a way to draw the shape of an idea, with branches and rivers and a centre.

## License

Personal project. Use it, fork it, modify it. No warranty.
