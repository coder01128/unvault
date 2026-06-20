# unVault

Batch PDF password remover. Drop a pile of password-protected PDFs, enter the password once, get them all back unlocked in a zip. Built for tax season.

**[Try it →](https://coder01128.github.io/unvault/)**

---

## Privacy first

Everything runs inside your browser. No files are uploaded. No server is involved. The decryption happens locally via JavaScript libraries — your bank statements never leave your machine.

This is non-negotiable for financial documents.

---

## How to use

1. **Drop your PDFs** onto the drop zone (or click to browse). Add as many as you need.
2. **Enter the password** shared by the batch (e.g. your ID number for FNB statements).
3. **Click unVault All** — a zip file downloads automatically when done.

Repeat with a new batch for a different bank / different password.

---

## Tech stack

| Library | Role |
|---|---|
| [pdf.js](https://mozilla.github.io/pdf.js/) (Mozilla) | Decrypts and renders encrypted PDFs in the browser |
| [pdf-lib](https://pdf-lib.js.org/) | Reconstructs clean, unencrypted output PDFs |
| [JSZip](https://stuk.github.io/jszip/) | Bundles all output files into a single `.zip` |
| [FileSaver.js](https://github.com/eligrey/FileSaver.js/) | Triggers the zip download across browsers |

No build step. No framework. Single `index.html` file.

---

## How the decryption works

**Preferred path (text-selectable output):** pdf-lib loads the encrypted PDF using the provided password and saves a new copy without encryption. The output is a proper PDF with selectable text.

**Fallback path (image-based output):** If pdf-lib can't handle the encryption type (common with AES-256 from modern banks), pdf.js decrypts the PDF in memory, renders each page to an HTML canvas, and pdf-lib wraps those canvas images into a new unencrypted PDF. Output is image-based — not text-selectable, but fully readable and suitable for filing.

Both paths produce valid, unlocked PDFs.

---

## License

MIT

---

Built by [DarkLoud DigitAI](https://github.com/coder01128) — where time is everything.
