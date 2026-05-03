# CLAUDE.md

Project-specific context for Claude Code sessions. Read this first.

## Working with Peter

How Peter likes to work — match this register and these defaults.

- **Conversational and direct.** No unnecessary preamble or padding. Don't pitch additional features before a build. Don't over-explain reasoning unless asked.
- **Precision in language.** Peter notices and cares about word choice.
- **Output first, questions after.** When iterating on something, give the working output, then the questions.
- **Match register.** Relaxed and informal in conversation. Precise in outputs.
- **Structured outputs follow the agreed format exactly.** Don't improvise structure once a format is settled.
- **Flag uncertainty briefly rather than guessing silently.**
- **Keep bullet points for structured outputs; prose for conversation.**
- **Always request confirmation before building or creating anything.** State the plan briefly, ask `y` to proceed, then build.
- **Just flag genuine blockers** when about to build — don't pitch additional features.

**Shortcuts Peter uses:**
- `y` = yes / confirm / proceed
- `n` = no / reject / different approach

## The build philosophy

This project has been built iteratively over many sessions. The workflow that works:

- **Isolate before folding.** When something is visually or behaviourally complex (spawn animations, layout algorithms, custom rendering tweaks), build a minimal standalone mockup file first — a small HTML file that demonstrates just the new piece. Peter tests it in isolation, confirms it works, then we fold the working version into the main file. Don't fold first and debug in the main file — too many moving parts.
- **One feature at a time.** Don't bundle. Each fold is its own commit-worthy unit.
- **Confirm before building.** Even small changes. State the plan, get `y`, then act.

## The project

**Anaximander** is a single-file HTML mind-mapping app. Canvas-based, vanilla JS, no build step, no server, no dependencies. Read `README.md` and `ARCHITECTURE.md` for the full picture before making changes. Architecture doc has data model, render loop, layout algorithms, persistence, file structure pointers.

**Working file:** Always the highest-versioned `anaximander_vN.html` in the repo root. Check with `ls anaximander_v*.html | sort -V | tail -1` if unsure.

**Repo:** github.com/Minotauria/anaximander. Live at minotauria.github.io/anaximander/.

## Quirks worth knowing

- **localStorage key is `anax_v7`** even though the file is on a higher version. Renaming would break existing user data. Keep it.
- **Spawn animation doesn't visibly fire on text-shape nodes.** Side-effect of where the text-shape early-return sits in `drawNode`. Accepted as-is; not a bug to fix.
- **Imported `.mm` files don't lay out beautifully.** The Coggle position model doesn't fully translate. Run *untangle* afterwards. Acceptable, not a bug.
- **Default colours and styling are botanical** — chrome `#1a201a`, borders `#2a3028` / `#303624`, text `#a09e8a` / `#909078` / `#c8c898` / `#dac888`, default node fill `#6a9a6a`, all UI in Lora serif. Don't drift from this palette without checking.
- **Bevel gradient on nodes is capped at 14px from top.** Intentional — looks wrong if it scales with node size. The gradient spans exactly `bevelEnd` (not `bevelEnd*2`) so it's fully gone by 14px.
- **Rhombus and diamond are aliases.** Old saves use `rhombus`, new saves use `diamond`. Both render identically. Don't remove the alias.
- **Single-click selects a node; double-click opens the editor.** This is intentional — single-click needs to leave canvas focus intact so Tab can trigger AI suggestions.
- **AI features require `state.ai` (global) and `map.legend` (per-map).** Both are defaulted in `loadState`. `state.ai = {enabled, apiKey, model}`. The localStorage key stays `anax_v7`.

## Workflow when changing the app

1. Find the working file: highest-versioned `anaximander_vN.html` in the repo root.
2. Open it in a browser to test (`open anaximander_vN.html` in terminal).
3. Make changes via the Code tab (or terminal).
4. Hard refresh the browser (⌘⇧R) to test.
5. When the change is solid, commit + push:
   - `git add -A && git commit -m "short message" && git push`
   - Live URL updates within ~30 seconds.
6. Don't bump the version filename until we cut a real new release.
7. **Keep the shortcuts menu current.** Any new keyboard shortcut or mouse gesture goes into the `SHORTCUTS` constant (near the bottom of the `<script>` block). Do this as part of the same change, not as a follow-up.

## The bank — features discussed but not built

These have been thought about, sometimes deeply. None are committed-to. Treat as starting points, not specs.

**Generate map from prompt** — Same code path as AI expansion, different entry point. Give Claude a topic / book / paper, get a starter map. Pairs naturally with AI expansion.

**Ollama + GPT providers** — Code skeleton for Ollama is already in v10 but parked. Revisit alongside a GPT/OpenAI option — do both at the same time so the provider tab pattern is complete in one go.

**Per-map AI toggle** — A switch on each map disabling AI features for that map specifically. For sensitive content. Requires the AI features to exist first.

**Nested map rooms** — A node contains its own sub-map. Dive in to expand it. Rough idea, no design work done.

**Temporal layers** — Versions of a map over time, like document chapters or timeline scenes. Rough idea, no design work done.

**Profiles** — Multiple users / contexts in one app instance. Rough idea, no design work done.

**Mobile / touch interface** — Pinch zoom, larger hit areas, mobile sidebar. Peter unsure he'd actually use it. Deferred indefinitely.

**Genogram symbols** — Standard genogram notation as a node shape set (circle = female, square = male, diamond = unknown/other, double-border = index person, X overlay = deceased, horizontal line = union, vertical drop = child). Rough idea, no design work done.

**Node groups** — Select multiple nodes by click-dragging a marquee (or ⌘-click to add to selection). Grouped nodes can be given a title and moved together. Rough idea, no design work done.

## What's deliberately not in this codebase

- No build step, no bundler, no transpilation.
- No tests. Manual testing in a browser.
- No classes. Free functions, global state in one `state` object.
- No modules. Single HTML file, single `<script>` block.
- Naming is short (`cmap`, `nrx`, `w2s`) because the file is long.

This is a personal tool that grew. It is not idiomatic modern frontend code. It is fast, hackable, and dependency-free. Keep it that way unless explicitly asked to change.

## Git rhythm

- Every meaningful change → commit with a short, specific message.
- Don't pile multiple unrelated changes into one commit.
- Push after each commit (the live site updates automatically).
- It's fine to ask "should I commit and push this?" before doing it — Peter sometimes wants to test more before committing.

## When in doubt

Ask. One question, briefly. Then proceed.
