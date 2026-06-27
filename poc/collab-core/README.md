# collab-core PoC

Proves the hardest part of the Boostnote modernization architecture: **real-time
collaborative markdown editing that stays local-first and keeps "the file is the
source of truth"**, with a **self-hosted** sync server and **device-pairing** auth.

This is a standalone modern-Node (22) project. It is **not** part of the legacy
Electron app build (the legacy lint/CI ignore `poc/`).

## What it validates

| Concern | How |
|---|---|
| Character-level concurrent editing (no lost writes) | Yjs CRDT (`Y.Text` per note) |
| Local-first / offline | Yjs (+ `y-indexeddb` in the browser) |
| "File is the source of truth" | server writes a derived `.cson` snapshot in the **legacy note shape** on every change, and seeds a freshly-loaded doc back from it |
| Self-hosted, flat-cost sync | Hocuspocus (`@hocuspocus/server`) |
| Access control (Yjs has none) | device-pairing token verified in `onAuthenticate` |
| Editor | CodeMirror 6 + `y-codemirror.next` (`yCollab`) — same pipeline as the legacy app (source markdown + markdown-it preview) |

## Layout

- `server/hocuspocus-server.mjs` — sync server: `onAuthenticate` (pairing gate),
  `onLoadDocument` (seed from `.cson`), `onStoreDocument` (write `.cson` snapshot).
- `test/convergence.test.mjs` — **headless proof**: two Yjs clients, auth
  rejection, A→B convergence, concurrent-edit survival, `.cson` write-back, and a
  late device re-seeded from the snapshot.
- `client/` — HackMD-style split editor (CM6 left, live markdown-it+DOMPurify
  preview right) with collaborative cursors. Prototype of the modern app's editor pane.

## Run

```bash
# Node 22 (the repo pins Node 14 for the legacy app; this needs modern Node)
npm install

# 1) headless proof — no browser needed
npm test
#    -> "All 6 checks passed — collab core architecture validated."

# 2) live demo
npm run server          # terminal A: Hocuspocus on ws://127.0.0.1:1234
npm run dev             # terminal B: Vite; open the URL in TWO windows and type
```

## Notes / next

- The PoC persists only a `.cson` **snapshot**; production should also persist the
  Yjs update log (e.g. Hocuspocus `Database`/SQLite) so history/undo survive restarts.
- Auth here is a single shared token; the real app uses per-device pairing
  (single-user, multi-device — no multi-user auth provider needed).
- License: inherits the repository's **GPL-3.0**.
