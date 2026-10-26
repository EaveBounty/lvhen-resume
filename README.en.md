[中文](README.md) ｜ **English**

# 履痕 (Lvhen) — local-first resume builder and career record

<p align="center"><img src="docs/assets/icon.png" width="112" alt="Lvhen app icon"></p>

> **A resume builder whose data is a folder of plain text you own.** Lvhen turns scattered career records (education, work, projects, papers, awards, attachments) into one continuously compiled resume, tailors it to a target job, and exports PDF (Typst on desktop, DartPdf fallback), single-column DOCX for ATS, or Markdown. Android / Windows / Linux / Web.

- **Display name**: 履痕 ｜ **Repository**: `lvhen-resume` ｜ **Package**: `com.eavebounty.lifeline`
- **Site**: <https://eavebounty.github.io/lvhen-resume/en/> ｜ **中文**: <https://eavebounty.github.io/lvhen-resume/>
- **License**: PolyForm Noncommercial License 1.0.0 ｜ © 长沙市果垂素宇工程设计有限公司

## Screenshots

| Home — compiled resume | Tailored export |
|---|---|
| <img src="docs/assets/screenshot-home.png" width="1280" height="900" alt="Lvhen home screen showing the compiled resume" loading="lazy"> | <img src="docs/assets/screenshot-export.png" width="1280" height="1500" alt="Lvhen export wizard with template preview" loading="lazy"> |

| Evaluation — gaps and scores | Categories — custom + drag to sort |
|---|---|
| <img src="docs/assets/screenshot-eval.png" width="1280" height="1400" alt="Lvhen resume evaluation screen" loading="lazy"> | <img src="docs/assets/screenshot-categories.png" width="1280" height="900" alt="Lvhen category management screen" loading="lazy"> |

## Why

Career material lives in chat logs, old resumes, cloud notes and disk folders. Every application means re-assembling it, re-typing it and re-formatting it — and the result is locked inside whichever tool produced it.

Lvhen's answer:

- **Data is files** — JSON/YAML plus attachments in a folder *you* choose; the app never owns it.
- **One record, one file** — `<uuid>.json` per experience, designed for multi-device merges.
- **Rebuildable index** — `index.sqlite` is a cache; delete it and nothing is lost.
- **No sync engine** — bring Syncthing, WebDAV or Git. Sync is a solved problem.
- **Compile, don't retype** — any data change recompiles the resume on the home screen.

```
data/
├── profile.json                     # identity, contact, headline, summary
├── records/<category>/<uuid>.json   # one experience per file
├── resumes/<id>/                    # per-application spec and export
└── index.sqlite                     # rebuildable (gitignore)
```

## Features

| Feature | Notes |
|---|---|
| 10 resume templates | ATS single-column, academic CV, fresh graduate, teacher, creative, business, tech-compact, elegant-serif, minimal-mono, modern-two-col |
| Per-job tailoring | Questionnaire + role research → plan (keep / compress / delete + reasons + ATS keywords) → render |
| 60 role profiles | Industry and seniority aware; lists missing hard requirements and orders fixes by marginal benefit |
| Exports | PDF (Typst desktop / DartPdf everywhere), DOCX (ATS single column), Markdown |
| Page fitting | Measures page count, trims by priority weights, re-renders (≤8 rounds) up to a hard page limit |
| AI intake | Paste text or photos → structured records with human confirmation; local Ollama supported |
| Platforms | Android (arm64/armv7/x86_64), Windows, Linux, Web demo (in-memory) |

## Build

```bash
flutter pub get
dart run build_runner build        # drift codegen
flutter build linux                # or: apk --release / windows / web
flutter test
```

Requirements: Flutter 3.44+, Android `compileSdk=36`. Release builds are produced by GitHub Actions (`release.yml` on `v*` tags).

## First launch

Choose an empty folder as the sync root. The app does **not** create one for you and does not write outside it. Recommended sync: Syncthing (devices), WebDAV (NAS), Git (version history; gitignore `index.sqlite`).

## FAQ

**Is Lvhen open source?** Source-available, not OSI open source: PolyForm Noncommercial 1.0.0. Non-commercial use and modification are allowed; commercial use needs a separate license.

**Does it upload my data?** No. There is no account system and no server-side profile. AI features call the endpoint you configure (a local Ollama instance works).

**Can I keep using Word/LaTeX?** You can export Markdown or DOCX and post-process; Lvhen's value is the structured source data, not a proprietary format.

**Is the web build usable?** It is an in-memory demo (`?demo=1`) for evaluation only — refresh clears state.

## Documentation

- [`ARCHITECTURE.md`](ARCHITECTURE.md) — design principles and module layout
- [`docs/DATA_FORMAT.md`](docs/DATA_FORMAT.md) — data schema
- [`docs/SYNC.md`](docs/SYNC.md) — sync guidance
- [`CHANGELOG.md`](CHANGELOG.md) — release notes

## License

PolyForm Noncommercial License 1.0.0 — see [`LICENSE`](LICENSE) and [`COMMERCIAL.md`](COMMERCIAL.md).
