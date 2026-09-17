# unVault — Build Spec for Claude Code (Sonnet)

## Context

This app is built by **DarkLoud DigitAI (DLD)** — a Johannesburg-based web studio. It's a free utility tool that will live on the DLD showcase site as a brand goodwill piece and case study. The tagline for DLD is **"Because time is everything."** Every DLD app solves one tedious workflow and saves the user meaningful time.

**The problem:** During tax season, Brad downloads ~36 password-protected bank statement PDFs across multiple banks. Each bank uses a different password (usually ID number or account number). Currently he unlocks them one at a time on SmallPDF — download, unlock, re-download, repeat 36 times. It takes forever.

**The solution:** A single-page app where you drop a batch of PDFs, enter one password, and get them all back unlocked in a zip file. Everything runs client-side — no files ever leave the browser.

---

## Hard Constraints

- **Single-file HTML.** One `index.html` — no build step, no framework, no bundler. Vanilla HTML/CSS/JS.
- **Client-side only.** Zero server calls. All PDF processing happens in the browser via JavaScript libraries loaded from CDN. This is non-negotiable — these are bank statements.
- **CDN dependencies (load from cdnjs.cloudflare.com or unpkg.com):**
  - **pdf.js** (Mozilla) — for reading/decrypting password-protected PDFs
  - **pdf-lib** — for reconstructing decrypted pages into clean unencrypted PDFs
  - **JSZip** — for bundling output files into a single .zip download
  - **FileSaver.js** — for triggering the browser download of the zip
- **Deploy target:** GitHub Pages under `coder01128` account, repo name `unvault`
- **Mobile-friendly** but desktop is the primary use case (this is a tax-season-at-the-desk workflow)

---

## UX Flow (Step-by-Step)

### State 1: Empty — Ready for files
- Hero area with DLD branding (dark bg, gold accent `#FFD700`, clean sans-serif type)
- Tagline: **"unVault your PDFs. Because time is everything."**
- Large drop zone with dashed border: "Drop password-protected PDFs here, or click to browse"
- Drop zone accepts `.pdf` files only, multiple selection enabled
- Privacy badge below drop zone: 🔒 "Your files never leave your browser. Everything runs locally."

### State 2: Files loaded — Ready to configure
- File list appears below the drop zone showing:
  - File name
  - File size (human-readable, e.g. "1.2 MB")
  - Status indicator (pending/processing/done/failed)
  - Individual remove button (✕) per file
- "Add more files" link/button above or below the list (allows adding to the batch)
- Password input field (type="password" with show/hide toggle)
  - Placeholder: "Enter the password for these PDFs"
  - Single password field — this is v1.0 (one batch, one password)
- **"unVault All" button** — large, prominent, gold on dark
  - Disabled until: at least 1 file loaded AND password field is not empty

### State 3: Processing
- "unVault All" button changes to "unVaulting..." with a spinner, disabled
- Each file in the list shows its own status:
  - ⏳ Waiting
  - 🔄 Processing (with a subtle animation)
  - ✅ unVaulted
  - ❌ Failed — with a short reason (e.g. "Wrong password" or "Not encrypted" or "Corrupted file")
- Overall progress bar at the top of the file list: "unVaulting 7 of 12..."
- **Critical: a single failed file must NOT stop the batch.** Process all files. Failures are flagged but the rest continue.

### State 4: Complete
- Progress bar shows "Done — X of Y unVaulted successfully"
- If any files failed, show a summary: "3 files could not be unVaulted (wrong password or not encrypted)"
- **"Download unVaulted PDFs" button** — triggers download of a .zip file
  - Zip file name: `unvaulted-pdfs-YYYY-MM-DD.zip`
  - Contains only the successfully unlocked files
  - File names inside zip: original filename (unchanged)
  - Files that weren't encrypted in the first place: include them as-is in the zip (don't error on them)
- **"Start Over" button** — resets to State 1 (clears file list, clears password)

---

## Technical Implementation Notes

### PDF Decryption Flow (per file)

```
1. Read file as ArrayBuffer using FileReader API
2. Attempt to load with pdf.js: pdfjsLib.getDocument({ data: arrayBuffer, password: userPassword })
3. If load succeeds → PDF was encrypted and password was correct
4. If load fails with "PasswordException" → wrong password, flag as failed
5. If load succeeds WITHOUT needing the password → file wasn't encrypted, include as-is
6. For successfully decrypted files:
   a. Get page count from pdf.js document
   b. Create new PDFDocument with pdf-lib
   c. For each page:
      - Render page to canvas at 2x scale using pdf.js
      - Convert canvas to PNG data URL
      - Embed PNG into new pdf-lib document as a full page
   d. Save pdf-lib document as Uint8Array
   e. Add to JSZip instance
7. After all files processed, generate zip and trigger download
```

### Teaching Note for Brad (include as code comment)

The reason we render to canvas and re-embed as images rather than copying the PDF structure directly is that pdf.js can decrypt and render a PDF but doesn't expose the raw decrypted PDF bytes in a way we can just save. pdf-lib can create PDFs but can't decrypt them. So the workaround is: pdf.js decrypts → renders to canvas (an image) → pdf-lib wraps that image into a new clean PDF. The tradeoff is the output is image-based rather than text-selectable, but for bank statements you're filing for tax purposes, that's fine.

**UPDATE — better approach if possible:** Before resorting to canvas rendering, try this first:
```
1. Load encrypted PDF bytes with pdf.js (providing password)
2. Use pdf.js to get the raw page data
3. Try loading the same bytes into pdf-lib with: PDFDocument.load(arrayBuffer, { password: userPassword, ignoreEncryption: true })
4. If pdf-lib can load it, save directly — this preserves text selectability
5. If pdf-lib can't handle the encryption, fall back to the canvas rendering approach
```
Try the direct pdf-lib approach first. If it doesn't work for encrypted files (it may not), fall back to canvas rendering. Either way the app works — it's a quality-of-output question, not a functionality one.

### Error Handling

- **Wrong password:** pdf.js throws a `PasswordException`. Catch it, flag the file as "Wrong password", continue to next file.
- **File not encrypted:** pdf.js loads without needing the password. Include the original file bytes in the zip unchanged.
- **Corrupted PDF:** pdf.js throws a generic error. Flag as "Could not read file", continue.
- **Browser memory:** For very large batches (50+ files or very large PDFs), memory could be an issue. Not a v1.0 concern — Brad's use case is ~36 statements of 2-5 pages each.

### Performance

- Process files sequentially, not in parallel. Parallel processing could overwhelm browser memory and makes the progress UI confusing. Sequential is fine for 36 files — should complete in under 60 seconds.

---

## Visual Design

- **Background:** Near-black (`#0a0a0a`)
- **Text:** White (`#ffffff`) for primary, muted grey (`#888888`) for secondary
- **Accent:** Gold (`#FFD700`) — buttons, progress bar, success indicators
- **Error:** Soft red (`#ff4444`)
- **Font:** `'Inter', 'Segoe UI', system-ui, sans-serif` — load Inter from Google Fonts
- **Border radius:** 8px on cards and buttons, 12px on the drop zone
- **Drop zone:** Dashed 2px border in `#333`, background `#111`. On drag-over: border goes gold, background lightens slightly
- **Cards/containers:** `#111` background, subtle `1px solid #222` border
- **Buttons:** Gold background, black text, bold weight. Hover: lighten 10%. Disabled: grey out to `#444` bg with `#666` text.
- **No gradients, no shadows, no rounded avatars, no decorative SVGs.** Clean and functional. The DLD aesthetic is dark, gold, and professional — not playful.

### DLD Branding (footer)

Small footer: "Built by darKLoud digitAl — where time is everything." with a link to the DLD site (placeholder href `#` for now).

---

## File Structure

```
unvault/
  index.html      ← the entire app (single file)
  README.md        ← generated alongside the app
```

---

## README.md Content

Generate a README that includes:
- Project name and one-line description
- "Privacy first" section explaining client-side processing
- How to use (3-step: drop files → enter password → download)
- Tech stack (pdf.js, pdf-lib, JSZip, FileSaver)
- License: MIT
- Built by DarkLoud DigitAI — link to GitHub profile

---

## What Success Looks Like

Brad drops 12 FNB statements into the app, types his FNB password, hits unVault All, and 15 seconds later downloads a zip containing 12 unVaulted PDFs. He then clears, drops 10 Standard Bank statements, types a different password, and does it again. Total time: under 2 minutes for all 36 statements. Previous workflow: 45+ minutes on SmallPDF.

---

## Session Instructions for Sonnet

1. Read this spec fully before writing any code.
2. Build the complete `index.html` as a single file — all CSS in `<style>`, all JS in `<script>`.
3. Load pdf.js, pdf-lib, JSZip, and FileSaver.js from CDN (unpkg or cdnjs).
4. Test the decryption flow mentally — make sure the pdf.js → pdf-lib pipeline is correct before writing the full UI.
5. Include code comments that explain WHY things work, not just WHAT they do — Brad is learning.
6. Generate the README.md alongside the app.
7. Output both files to `/mnt/user-data/outputs/`.
