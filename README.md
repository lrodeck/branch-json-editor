# Branch

A JSON editor that works on the tree rather than on the text. It exists because
reformatting a file in a textarea doesn't help when the edit you actually want is
"every `price` in this array is a string and should be a number."

Everything is one static page: no build step, no framework, no bundler, no
server, no account. The editor runs entirely in the browser and your document
never leaves the machine.

## What it does

**Shape paths.** Every node has a structural address — `$.catalog[*].price`.
Press `m` on one value and you have selected that position in every sibling
branch at once. The batch panel then acts on all of them: set a value, run an
expression with a live preview (`Number(v)`, `v.trim()`, `v * 1.2`), find and
replace across keys or values, coerce types, rename the key everywhere, wrap,
unwrap, delete.

**Branch keys.** Put the cursor on an array and the inspector lists the union of
keys across its children with how many children carry each one. A key present in
7 of 9 rows shows up amber; one click fills the two that are missing it, another
renames it in all nine. Mixed types (`number string`) are how you notice the
inconsistency in the first place.

**Kinship highlighting.** Selecting a node tints every other node carrying the
same key: solid rose when it also sits at the same structural position, dotted
when the key merely repeats somewhere else in the document. Hovering previews the
same thing without disturbing the selection. It's the warning you want before
renaming by key.

**Four views over one document.** An indented tree; a node-link chart you can pan
and zoom, with per-depth folding; a spreadsheet table for any array of objects,
where renaming a column renames the key in every row; and the raw text for paste
in and paste out. Folding and selection are shared, so work moves between them.

`?` in the app lists the keyboard map. Undo covers the last 80 edits.

**The look.** Hardware-instrument panel: a putty or near-black chassis carrying
outlined modules, each with a labelled header strip; physical keycap buttons with
a 2px hard shadow that depress on click; LED indicators for the active view and
the unsaved-changes pulse; one neon green accent (`#39ff14`) and nothing else
coloured except value types. Geist for UI, Geist Mono for all data. Light and
dark both follow the system and can be switched with the toggle in the title bar.

## Deploy from GitHub to Cloudflare

Push this repo to GitHub, then pick one of two routes. Both redeploy on every
push to `main`, permanently, with nothing to run by hand.

### Route A — GitHub Actions (the workflow in this repo)

`.github/workflows/deploy.yml` already does the deploy. You only need two
repository secrets.

1. **Get an API token.** Cloudflare dashboard → My Profile → API Tokens →
   Create Token → use the **Edit Cloudflare Workers** template. Copy the token;
   it is shown once.
2. **Get your account ID.** Workers & Pages → right-hand sidebar, or the hex
   string in the dashboard URL.
3. **Add both to GitHub.** Repo → Settings → Secrets and variables → Actions →
   New repository secret:
   - `CLOUDFLARE_API_TOKEN`
   - `CLOUDFLARE_ACCOUNT_ID`
4. **Push to `main`.** The Actions tab shows the run; the Worker appears at
   `branch-json-editor.<your-subdomain>.workers.dev`. Rename it by changing
   `name` in `wrangler.jsonc`.

You can also trigger a deploy by hand from the Actions tab (Run workflow), and
test the credentials locally with `npx wrangler whoami` then `npm run deploy`.

### Route B — Cloudflare builds it, no secrets, no workflow

Workers & Pages → Create → Workers → Connect to Git, choose the repo, and accept
the defaults. Cloudflare reads `wrangler.jsonc`, finds no build step, and uploads
`public/`. Every push to `main` deploys; other branches get preview URLs. If you
take this route, delete `.github/workflows/deploy.yml` so the two systems don't
both deploy the same commit.

Pages works identically if you prefer it: Connect to Git, empty build command,
output directory `public`. Cloudflare now recommends Workers for new projects,
which is why the config here targets Workers.

### A custom domain

Open the Worker → Settings → Domains & Routes → Add. If the domain's DNS is
already on Cloudflare, the record and certificate are handled for you. A
`workers.dev` subdomain works fine too, but installing the app and the `.json`
file association need HTTPS, which both give you.

### Locally

```sh
npm install
npm run dev     # wrangler dev, closest to production
npm run serve   # plain python http server on :8787
```

Service workers need `https` or `localhost`, so open `http://localhost:8787`
rather than a LAN IP if you want to exercise the offline behaviour.

## Installing it as an app

Open the deployed site in Chrome, Edge, or another Chromium browser and use the
install button in the address bar. You get a standalone window with no browser
chrome, an entry in your launcher, and:

- **Real files.** Open and Save use the File System Access API, so `Cmd`/`Ctrl+S`
  overwrites the file you opened rather than dropping another copy in Downloads.
  `Cmd+Shift+S` is Save as, `Cmd+O` is Open.
- **Double-click to open.** `manifest.webmanifest` registers Branch as a handler
  for `.json`, `.geojson`, and `.jsonc`. The OS may ask you to confirm the
  association the first time.
- **Drag and drop.** Drop a file on the window; if the browser hands over a
  writable handle, saving goes straight back to that file.
- **Offline.** The service worker caches the shell on first visit.

Safari and Firefox don't implement the File System Access API. There the app
still works, but Open becomes an upload box and Save becomes a download. The
title bar shows a dot when there are unsaved changes either way.

## Things worth knowing before you rely on it

**Large files.** Every edit deep-clones the document and pushes it onto the
history stack. That's comfortable to a few megabytes and unpleasant beyond tens
of megabytes. If you need to work on big files, change the history in
`index.html` to store patches (path, old value, new value) instead of whole
snapshots. The tree also stops rendering past 4,000 visible rows and the chart
past 2,200 nodes; fold by depth or zoom into a branch.

**A draft is kept in `localStorage`** so a refresh doesn't lose work. File
handles are not persisted, so after a reload Save asks where to put the document
again. Persisting handles in IndexedDB is the fix if that annoys you.

**Fonts.** The page loads Geist and Geist Mono from Google Fonts, which is the
only outbound request it makes. For genuine offline typography and one less
third party, download the woff2 files into `public/fonts/`, replace the `<link>`
in `index.html` with `@font-face` rules, add the files to `SHELL` in `sw.js`, and
drop the two font hosts from the CSP in `_headers`.

**Caching.** `index.html` is fetched network-first, so a deploy reaches people as
soon as they're online. Bump `VERSION` in `sw.js` when you change any other
cached file.

## Files

```
.github/workflows/deploy.yml   push to main, deploy to Cloudflare
public/
  index.html                   the entire editor
  sw.js                        offline shell
  manifest.webmanifest         app identity, .json file handlers
  _headers                     CSP and caching, read by Pages and Workers
  icon.svg, icon-*.png         launcher icons
wrangler.jsonc                 assets-only Worker
```

## Taking it further

The same `public/` directory works unchanged as the frontend of a Tauri app if
you later want a signed native binary with a proper menu bar: point
`frontendDist` at it and swap the File System Access calls for the `dialog` and
`fs` plugins. Other directions the editor is set up for: JSON Schema validation
in the inspector, diffing two documents with per-node accept and reject, and
recording a sequence of batch edits as a replayable macro.
