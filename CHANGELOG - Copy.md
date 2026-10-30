# Changelog

## 2026-10-24 — Reviewer tools and guideline versioning (Milestones F–G)

- Added revision-scoped reviewer labels (`valid`, `false_positive`, or null) and reasoned finding dismissal. These audit annotations remain separate from engine output and scoring.
- Added persistent per-draft Creator/Reviewer comment threads, recorded comments and reviewer actions in the workflow history, and included those notes in PDF exports.
- Added monotonic guideline version bumps for every successful Admin guideline create, update, and delete, plus demo reset. Each check stores the guideline version used.
- Added an additive SQLite migration giving legacy revisions guideline version 1; new review/comment tables are created without dropping existing data.
- Added the exact stale-guideline report notice and one-click recheck; the API prevents submission until a stale revision is rechecked.
- Validation: 45 pytest cases pass; the frontend TypeScript/Vite production build succeeds.
- No external repository transfer or deployment was performed for these changes.

## 2026-10-30 — Reviewer learning, evidence library, and visual ad checks

- Reviewer `valid` / `false_positive` labels now feed per-rule rates; 3+ labels and >50% false positives lower future severity one step with the exact downgrade note. Added rule/creator recurrence panels and clearly marked synthetic dashboard seed examples.
- Added Admin text/Markdown and selectable-text PDF evidence uploads (2 MiB), extraction/storage, conservative lexical claim matching with excerpts, and “Supported by” / “No evidence found” report states; seeded the DemoWear cotton certificate sample.
- Added verified PNG/JPG/JPEG ad-creative uploads (5 MiB), bounded Tesseract English OCR through the existing hybrid checker, Admin approved-logo image upload, OpenCV edge template matching, confidence-labeled missing-logo findings, persistent reviewer/report previews, and friendly unavailable/error handling.
- Tests: 53 pytest cases pass; the scripted main flow passes independently; production TypeScript/Vite build, backend compileall, and `git diff --check` pass. Local OCR/OpenCV tests used Tesseract 5, pytesseract, OpenCV, NumPy, and Pillow.
- Published Webdev version `c0b61cb29e0d9dcbd63d024c8439df375a454796` at https://inkloom-mhp9bk9a.manus.space. Read-only public smoke checks returned 200 for home, health, guidelines, and dashboard. The hosted container build succeeded; a separate local Docker build, live AI-provider call, and incognito-browser test were not performed. GitHub transfer/public visibility remains unverified; see `KNOWN_ISSUES.md` and `SUBMISSION.md`.
