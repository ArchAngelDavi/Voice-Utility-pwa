# CHEW Voice Chart

Offline-first speech-to-text patient documentation app for Community Health
Extension Workers. Dictate a consultation, watch the chart auto-fill, review
and correct it, save it offline, and export to CSV / NHMIS / DHIS2 formats.

Built as a single-page React app (React 18 via CDN, no build step required)
so the whole thing runs from static files — no server-side code, no
database server, no bundler.

## Requirements

- **Google Chrome or Microsoft Edge** on desktop or Android, for live voice
  recognition (the Web Speech API these use isn't supported in Firefox and
  is only partially supported in Safari). The chart, saving, editing and
  exporting all work in any modern browser even without speech support —
  you'd just type instead of dictate.
- A **local web server** to serve the folder (see below). Opening
  `index.html` directly by double-clicking it (a `file://` URL) will run the
  app, but Chrome blocks microphone access and service workers on `file://`
  pages, so voice recording and offline installability won't work — serving
  it, even just from your own machine, fixes that.

## Run it

Pick whichever you have available. All of these serve the current folder
on `http://localhost` and work fully offline once the page has loaded once.

**Option A — Node (no install needed beyond Node itself):**
```bash
npm start
```

**Option B — Python (already on most machines):**
```bash
python3 -m http.server 5173
```

**Option C — any other static file server** (`live-server`, VS Code's "Live
Server" extension, `caddy file-server`, etc.) pointed at this folder works
identically — there's nothing to build or compile.

Then open **http://localhost:5173** (or whichever port you used) in Chrome
or Edge.

## Using the app

1. **Create an account** on first launch — this is a local, on-device login
   (username + password, hashed and salted) so multiple CHEWs can share one
   device. No account data leaves the browser.
2. **Tap the big record button** on the home screen and start dictating a
   consultation naturally, e.g.:
   > "Patient name is Amina Bello, age 34, gender female, temperature is
   > 38.5, blood pressure is 120 over 80, pulse rate is 92, complains of
   > fever and headache, diagnosis is malaria, treatment is paracetamol and
   > ACT, follow up in 3 days."
3. The chart fields fill in live as you talk. Tap **Stop**, then **review
   and correct** anything the parser missed or mis-heard.
4. Tap **Save offline** — the record is stored in the browser's local
   storage immediately, connection or not.
5. **History** lists everything you've saved. **Export** generates a plain
   CSV, an NHMIS-compatible CSV, or a DHIS2 event-style JSON file you can
   download.
6. **Settings** lets you change the dictation language and, if you're
   running this inside a host that grants it, turn on optional cloud
   backup.

## What's real here, and what's a stand-in

This is a working prototype, not a production clinical system. A few things
are deliberately simplified:

- **Voice engine**: uses the browser's built-in Web Speech API rather than
  an embedded offline model (e.g. Vosk). This means live dictation itself
  needs an internet connection in most browsers, even though saving,
  editing, and exporting data all work fully offline. Swapping in a true
  offline engine is possible but needs a real model file hosted alongside
  the app (tens of MB) and a WASM-capable runtime — out of scope for this
  static demo.
- **Language support**: English is the most reliable option. Hausa, Yoruba
  and Igbo are offered in Settings, but browser-level speech recognition
  support for them is inconsistent across devices — the form can always be
  filled in by typing regardless of dictation language.
- **Auto-fill parsing**: uses cue-phrase pattern matching ("temperature
  is...", "diagnosis is...") rather than a trained clinical NLP model. It
  works well with structured dictation like the example above; free-form
  speech will need more manual correction in the review step.
- **DHIS2 export**: produces a realistically-shaped event payload, but the
  `dataElement` values (`DE_DIAGNOSIS`, `DE_TEMPERATURE`, etc.) are
  placeholders. Swap them for your DHIS2 instance's real metadata UIDs
  before importing.
- **Authentication**: client-side only (SHA-256 + per-user salt, stored in
  `localStorage`). Fine for a single shared device in a demo/pilot; not a
  substitute for real server-side auth in a multi-device deployment.
- **Cloud backup**: only appears/works when the app is opened inside a host
  environment that grants a `db`/`user` capability (for example, a Claude
  artifact). Run standalone like this, cloud backup will show as
  unavailable and the app works entirely offline/local, which is the
  expected default.

## Folder contents

```
chew-voice-chart-app/
├── index.html          the entire app: markup, styles, and React code
├── manifest.json        PWA metadata (name, icons, theme color)
├── service-worker.js    caches the app shell for offline page loads
├── icons/                app icons used by the manifest / home screen
├── package.json          convenience "npm start" local server script
└── README.md             this file
```

There's no `src/` folder or build pipeline — everything the browser needs
is already in `index.html`. This keeps the project runnable by just
serving static files, with no `npm install` of a framework required.

## Installing as an app

Once served over `http://localhost` (or deployed to real HTTPS hosting),
Chrome/Edge will offer an install icon in the address bar (or "Add to Home
Screen" on Android). Installing uses the manifest and service worker in
this folder to give the app its own window and an offline-loading icon on
your device.
