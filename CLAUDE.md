# unVault — CLAUDE.md

Project instructions for Claude Code. Read this fully before touching any file.

---

## What This Is

**unVault** is a free, single-file browser utility built by DarkLoud DigitAI (DLD).
It batch-unlocks password-protected PDFs entirely client-side — no server, no uploads, no accounts.
The brand tagline is: **"Because time is everything."**

---

## Hard Rules — Never Break These

### Security (non-negotiable)
- **Never log the password to the console** — not even during debugging
- **Never store the password** in `localStorage`, `sessionStorage`, `IndexedDB`, cookies, or any persistent browser storage
- **Never send any data** (passwords, filenames, file contents) via `fetch`, `XMLHttpRequest`, or any network call
- **Clear the password variable from memory** after all files have been processed — set it to `null` or `''` immediately after the batch completes
- These rules exist because users are processing real bank statements with real passwords. There are no exceptions.

### Architecture
- **Single-file HTML only.** Everything lives in `index.html` — all CSS in `<style>`, all JS in `<script>`
- **No build step, no framework, no bundler.** Vanilla HTML/CSS/JS
- **CDN only for dependencies** — load from unpkg.com or cdnjs.cloudflare.com:
  - pdf.js (Mozilla) — decrypt and render password-protected PDFs
  - pdf-lib — reconstruct clean unencrypted PDFs
  - JSZip — bundle output files into a .zip
  - FileSaver.js — trigger the browser download

---

## Visual Design Tokens

| Token | Value |
|---|---|
| Background | `#0a0a0a` |
| Surface | `#111111` |
| Border | `#222222` |
| Primary text | `#ffffff` |
| Secondary text | `#888888` |
| Accent (gold) | `#FFD700` |
| Error | `#ff4444` |
| Font | `'Inter', 'Segoe UI', system-ui, sans-serif` |
| Border radius | 8px cards/buttons, 12px drop zone |

No gradients. No decorative shadows. No rounded avatars. Dark, gold, professional.

---

## App Name & Copy Conventions

- App name is always written **unVault** — lowercase u, capital V, no space
- Button: **unVault All**
- Processing state: **unVaulting...**
- Success status per file: ✅ **unVaulted**
- Progress: **"unVaulting 7 of 12..."**
- Completion: **"Done — X of Y unVaulted successfully"**
- Download button: **"Download unVaulted PDFs"**
- Zip filename: `unvaulted-pdfs-YYYY-MM-DD.zip`
- Footer: `Built by DarkLoud DigitAI — Because time is everything.`

---

## Code Comment Standard

Include comments that explain **WHY**, not just WHAT. The developer (Brad) is learning
while building. For any non-obvious technical decision, add a short plain-English
comment explaining the reasoning. Example:

```js
// We process files sequentially rather than in parallel to avoid overwhelming
// the browser's memory — 36 PDFs processed simultaneously could cause a crash.
```

---

## Deploy Target

- GitHub Pages under `coder01128`
- Repo name: `unvault`
- Live URL will be: `coder01128.github.io/unvault`

---

## Out of Scope for v1.0

- Multi-group / multi-password support (v1.1 feature)
- Drag-to-reorder files
- Preview of unlocked PDFs before download
- Any backend, API, or server-side processing
