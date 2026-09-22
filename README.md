# Persian Pharmacology Notes - Transcription & Handbook Build

Two related deliverables in one workspace. First, a page-by-page visual transcription of 182
pages of handwritten Persian pharmacology notes (`1.pdf` 99 pages, `2.pdf` 83 pages) - CamScanner
image scans with no text layer, so each page was cropped into overlapping top/bottom halves and
transcribed by reading the images. Second, a Node script tree that turns the resulting structured
drug data into a formatted Persian/English `Pharmacology-Randomized.docx` handbook. The method and
transcription rules are recorded in `PROGRESS.md`.

**Suggested repo name:** `persian-pharm-transcripts`
**Stack:** Markdown transcripts, Node.js with the `docx` library (v9.7.1), plus one unrelated Go/Gin subproject
**Status:** finished
**Last modified:** 2026-09-02

## What it does

- **Transcription pipeline.** 364 crops in `img/` (`A_###t/b.png` from 1.pdf, `B_###t/b.png` from
  2.pdf; top crop = 0–56 % of page height, bottom = 44–100 %, overlap so no line is cut) were
  transcribed in 89 batches into `parts/<TAG>_p###-p###.md`, then concatenated into
  `1-transcript.md` (99 page markers, 195 headings) and `2-transcript.md` (83 markers, 171).
  Rules: Persian stays Persian, drug names stay English, each drug is `### <fa> (<en>)`, page
  breaks are `--- page N ---`, illegible text is `[?]`, and every drug keeps the monograph field
  order برند / دوز max / اشکال دارویی / موارد مصرف / عوارض / تداخل / بارداری / شیردهی.
- **Handbook build (`docx-build/`).** `data1.js`–`data4.js` hold the structured content
  (47 categories → groups → drugs, ~534 drug entries with brand, forms, max dose, indications,
  interactions, side effects, pregnancy/lactation). `gen-sections.js` regroups categories into
  top-level sections and normalises some titles (e.g. «قطره‌ها» → «قطره‌های گوش و چشم»), rewriting
  the data files as JSON and emitting `sections.js`. `build.js` renders the document with the
  `docx` API: a دارو | اشکال | برند + دوز | مصرف/عوارض/تداخل table per group, letter layout with
  9360-twip content width, fixed accent palette, TOC, footers with page numbers.
  `patch.js`, `patch2.js`, `dedash.js` are one-shot text fixers applied over the data files
  (`dedash.js` replaces embedded em dashes with Persian separators).
- **Reference sources.** `PFC_original_backup.pdf` is a 129-page typed Persian pharmacology summary
  («خالصه و نکات کاربردی داروشناسی») kept as a cross-check source; `PMNT.docx` is a separate
  condition-first clinical note (pancreatitis and friends) with the same monograph fields.
- **`ftp-ui/` - unrelated subproject.** A small Go web app (Gin + `jlaffaye/ftp`) that browses a
  remote site over FTP: `GET /` (embedded HTML), `/api/list`, `/api/download`, `POST /api/upload`,
  `POST /api/mkdir`. Configured from `FTP_HOST`/`FTP_PORT`/`FTP_USER`/`FTP_PASS`/`WEB_PORT`/
  `SITE_NAME` environment variables (defaults to `localhost`/`anonymous` and the site name
  "Araz Leather - Files"), with an nginx vhost and a systemd install script.

## Layout

```
1-transcript.md / 2-transcript.md   finished transcripts (final artefacts, 123 KB / 148 KB)
PROGRESS.md                         method + transcription rules (status checkboxes now stale)
img/                                364 page crops            parts/  89 per-batch transcriptions
docx-build/                         data1-4.js, gen-sections.js, sections.js, build.js, patch*.js
Pharmacology-Randomized.{docx,pdf}  generated output; -v2 … -v9 are earlier regenerations
ftp-ui/                             main.go, go.mod, deploy.sh, upload_and_deploy.bat, nginx-ftpui.conf
```

## Running it

```bash
cd docx-build
npm install
node gen-sections.js     # rewrites data*.js as JSON and regenerates sections.js
node build.js            # writes Pharmacology-Randomized.docx
```

`build.js` writes to a hardcoded absolute path (`C:/Users/Administrator/Downloads/pdf-extract/...`);
edit it before running elsewhere. The `docx` → `pdf` conversion that produced
`Pharmacology-Randomized.pdf` was done in LibreOffice/Word, not by a script here.

For the FTP UI:

```bash
cd ftp-ui && go run .            # local, reads env vars with dev defaults
bash deploy.sh                   # on the server: installs Go, builds to /opt/ftpui, sets up the ftpui service
```

## Notes

- `img/` and `parts/` are intermediate working files; only the transcripts and `docx-build/`'s
  scripts and data are worth versioning. `docx-build/node_modules/` is present and must be ignored,
  along with the generated `Pharmacology-Randomized*` files.
- The two transcripts are hand/visually transcribed from someone's personal study notes - treat
  doses and interactions as unreviewed, not as reference data.
- The generated handbook carries a student name and student ID in its header block, and the
  source PDFs are third-party scans. Strip personal data and clear rights before publishing
  any of this content publicly; the scripts themselves are safe to share.
- "Randomized" in the output filename is not explained by `build.js` - the generator applies no
  shuffling, so the ordering is fixed by the `data*.js` files. `-v2` … `-v9` are successive
  regenerations from Aug 31 – Sep 2, not separate algorithms.
- `PROGRESS.md` predates completion and still lists both PDFs as "transcribed: NONE YET"; the
  finished transcripts supersede it.
- Related but separate: `Documents/Projects/monograph-ocr` does the same job on a printed 617-page
  Persian drug monograph book rather than handwritten notes.
