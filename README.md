# Branch

A JSON editor that works on the tree rather than on the text. One static page:
no build step, no framework, no bundler, no server. Your document never leaves
the browser.

## Deploy

Push this repo to GitHub, then in the Cloudflare dashboard:

- Workers & Pages → Create → Pages → Connect to Git → pick the repo
- Framework preset: **None**
- Build command: **leave empty**
- Build output directory: **`/`**

Save and deploy. Every push to your production branch redeploys; other branches
get preview URLs. Custom domain under the project's Custom domains tab.

Everything is served as-is from the repo root, so `index.html`, `sw.js`,
`manifest.webmanifest`, the icons and `_headers` all need to stay at the top
level — the service worker and the `.json` file association both depend on being
served from `/`.

To run it locally: `python3 -m http.server 8080`, then open
`http://localhost:8080`. Service workers only run on `localhost` or HTTPS.

## What it does

**Shape paths.** Every node has a structural address — `$.catalog[*].price`.
Press `m` on one value and you have selected that position in every sibling
branch at once. The batch panel then acts on all of them: set a value, run an
expression with a live preview (`Number(v)`, `v.trim()`, `v * 1.2`), find and
replace across keys or values, coerce types, rename the key everywhere, wrap,
unwrap, delete.

**Branch keys.** Put the cursor on an array and the second rail module lists the
union of keys across its children with how many carry each one. A key present in
7 of 9 rows shows amber; one click fills the two missing it, another renames it
in all nine. Mixed types (`number string`) are how you spot the inconsistency.

**Kinship highlighting.** Selecting a node tints every other node with the same
key: filled when it also sits at the same structural position, dotted underline
when the key merely repeats elsewhere. Hovering previews the same thing without
disturbing the selection.

**Four views over one document.** Indented tree; node-link chart you can pan,
zoom and fold by depth; spreadsheet table for any array of objects, where
renaming a column renames the key in every row; and raw text with a line gutter.
Folding and selection are shared across them.

`?` lists the keyboard map. Undo covers the last 80 edits.

**The look.** Hardware-instrument panel: putty or near-black chassis carrying
outlined modules with labelled header strips, keycap buttons that depress on
click, LED indicators for the active view and unsaved changes, one neon green
accent. Geist for UI, Geist Mono for data. Follows the system light/dark setting
and can be switched in the title bar.

## Installing it as an app

Open the deployed site in a Chromium browser and use the install button in the
address bar. You get a standalone window, plus:

- **Real files.** `Cmd`/`Ctrl+S` overwrites the file you opened (File System
  Access API). `Cmd+Shift+S` save as, `Cmd+O` open.
- **Double-click to open.** `manifest.webmanifest` registers Branch as a handler
  for `.json`, `.geojson`, `.jsonc`.
- **Drag and drop** a file onto the window.
- **Offline** after the first visit.

Safari and Firefox lack the File System Access API; there Open becomes an upload
box and Save a download. The title bar LED shows unsaved changes either way.

## Worth knowing

**Large files.** Every edit deep-clones the document onto the history stack.
Fine to a few megabytes, unpleasant beyond tens of megabytes — switch the
history to patches (path, old value, new value) if you need that. The tree stops
rendering past 4,000 visible rows and the chart past 2,200 nodes; fold by depth
or zoom into a branch.

**A draft is kept in `localStorage`** so a refresh doesn't lose work. File
handles are not persisted, so after a reload Save asks where to put the document.

**Fonts** come from Google Fonts, the only outbound request the page makes. To go
fully offline and drop the third party: put the woff2 files in `fonts/`, swap the
`<link>` for `@font-face`, add them to `SHELL` in `sw.js`, and remove the two
font hosts from the CSP in `_headers`.

**Caching.** `index.html` is fetched network-first, so a deploy reaches people as
soon as they're online. Bump `VERSION` in `sw.js` when you change any other file.
