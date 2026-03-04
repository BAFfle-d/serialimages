# CLAUDE.md — AI Assistant Guide for `serialimages`

## Project Overview

**serialimages** is a standalone, single-file browser application for extracting frames from a video and exporting them as a PDF or ZIP archive. It also supports extracting the audio track from a video.

- **No backend** — all processing happens client-side in the browser.
- **No build step** — open `index.html` directly in a browser, or serve it with any static file server.
- **No package manager** — external libraries are loaded from CDN at runtime.

---

## Repository Structure

```
serialimages/
├── CLAUDE.md         # This file
├── README.md         # One-line project description
└── index.html        # The entire application (HTML + CSS + JS, ~1030 lines)
```

The project is intentionally minimal. **All code lives in `index.html`.**

---

## Running the Application

```bash
# Option 1 — open directly (works for most features)
open index.html

# Option 2 — serve via HTTP (recommended; avoids cross-origin restrictions)
python3 -m http.server 8000
# then visit http://localhost:8000
```

There is no build, compile, or install step.

---

## Key Application Features

| Feature | Description |
|---|---|
| Serial extraction | Extract frames at fixed time intervals (seconds) or fixed frame intervals |
| Manual extraction | Capture a single frame at the current video playhead position |
| ZIP export | Download all extracted images as a ZIP file |
| PDF export | Download extracted images in a PDF with 1 / 2 / 3 / 4 / 6 images per page |
| Audio export | Extract the audio track and download as WebM |
| Dark mode | Toggle via settings panel |
| Timezone offset | Adjust the timestamps written into filenames |
| Image format | Choose JPEG or PNG output |
| Extended video mode | Raise the 5-minute default limit to 10 minutes |

---

## Architecture — Inside `index.html`

### Section breakdown (top to bottom)

1. **`<head>`** — CDN imports (Tailwind, Google Fonts, JSZip, jsPDF)
2. **`<body>` / HTML** — full page layout: header, upload panel, serial/manual extraction controls, gallery, settings modal, progress modal
3. **`<style>`** — small set of custom CSS rules not covered by Tailwind (dark-mode overrides, scrollbar styling, animations)
4. **`<script>`** — the entire application logic (~650 lines of vanilla JS)

### JavaScript structure (not modularised — all global scope)

| Area | Key identifiers |
|---|---|
| State variables | `images[]`, `videoFile`, `serialNumber`, `baseFilename` |
| Video handling | `extractFrame()`, the `<video>` element, canvas-based frame capture |
| Gallery rendering | `renderGallery()`, drag-and-drop reorder handlers |
| Export | `exportZip()`, `exportPdf()`, `exportAudio()` |
| Settings | `darkModeToggle`, `tenMinuteModeToggle`, `timezoneOffset` |
| Modals | `openModal()`, `closeModal()` |
| Utility | `formatTime()`, `formatUnixTime()` |

### External dependencies (CDN, no lock-file)

| Library | Version | Purpose |
|---|---|---|
| Tailwind CSS | latest (unversioned CDN) | Utility-first styling |
| Inter (Google Fonts) | latest | Typography |
| JSZip | 3.7.1 | ZIP archive creation |
| jsPDF | 2.5.1 | PDF generation |

Browser APIs used: Canvas, Video, Drag-and-Drop, MediaRecorder, Blob/URL.

---

## Code Conventions

- **Language:** vanilla JavaScript (ES2020+) with no transpilation.
- **Naming:**
  - JavaScript variables and functions — `camelCase`
  - HTML element `id` attributes — `kebab-case` (e.g. `modal-container`, `video-upload`)
  - Related UI groups prefixed consistently — `serial-*`, `manual-*`, `export-*`
- **DOM access:** `document.getElementById()` throughout; no framework.
- **Async:** `async/await` for frame-extraction promises; callbacks elsewhere.
- **CSS:** Tailwind utility classes in HTML; custom rules only where Tailwind cannot reach.
- **FPS assumption:** frame-based interval calculations hardcode 30 FPS.

---

## Constraints and Known Limitations

- Maximum video duration is 5 minutes (user can unlock 10-minute mode via settings).
- No persistence — extracted images are lost on page refresh.
- Audio export format is WebM (browser MediaRecorder default).
- Requires a modern browser with Canvas, Video, Drag-and-Drop, and MediaRecorder APIs.
- Requires internet connection for CDN assets (Tailwind, fonts, JSZip, jsPDF).
- Memory usage may be high with large videos; browser-dependent.

---

## Development Workflow

### Making changes

1. Edit `index.html` directly.
2. Reload the browser tab to see changes.
3. No build or compile step is needed.

### Adding a new feature

- Add any new HTML markup in the `<body>` near related controls.
- Add CSS in the `<style>` block only if Tailwind classes are insufficient.
- Add JavaScript at the bottom of the `<script>` block.
- Follow existing naming conventions (see above).
- Keep everything in `index.html`; do **not** split into multiple files without a compelling reason.

### CDN dependency updates

To update a CDN library, change the `src` URL in the `<head>`. There is no lock-file — document the chosen version in a comment if the URL is unversioned.

---

## Testing

There is no automated test suite. Testing is manual:

1. Open `index.html` in the browser (or via `python3 -m http.server`).
2. Upload a short MP4 or MOV file.
3. Exercise serial extraction, manual frame capture, ZIP export, PDF export, and audio export.
4. Verify dark mode, settings toggles, and gallery drag-and-drop reordering work correctly.
5. Check across at least one Chromium-based browser and Firefox.

---

## Git / Branch Conventions

- `master` — stable releases.
- `claude/*` — AI-assisted feature/fix branches; merge to `master` via PR.
- Commit messages should be short and imperative (e.g. `Add timezone offset to PDF filename`).

---

## What NOT to Do

- Do not introduce a build system, bundler, or package manager unless the user explicitly requests it.
- Do not split `index.html` into separate JS/CSS files unless explicitly requested.
- Do not add a backend; the application is intentionally client-side only.
- Do not add framework dependencies (React, Vue, etc.) without explicit instruction.
- Do not pin CDN URLs to arbitrary new versions without testing the change in a browser.
