# Inkloom AI — Ready-to-paste hackathon submission

## (a) Completed features — one per line

- No-login DemoWear workflow with Creator / Reviewer / Admin demo roles and guideline CRUD.
- Hybrid brand checks for text, text-based PDF, batches, and ad-creative images; deterministic price, terminology, disclaimer, tone, and evidence-needed rules plus optional validated AI with graceful rules-only fallback.
- Evidence-linked findings with exact spans, severity, evidence/source references, uncertainty, suggested edits, apply-fix/recheck, revision history, and human approval/request-changes flow.
- Reviewer comment threads, finding dismissal reasons, and `valid` / `false_positive` labels; qualifying future findings are downgraded one severity step after at least 3 labels and a false-positive rate above 50%, with an explicit explanation.
- Dashboard charts for ordinary finding/decision/health metrics, false-positive rate per rule, and recurring findings per rule and pseudonymous creator; seeded demo feedback is clearly identified as synthetic.
- Admin evidence library for titled UTF-8 text/Markdown and selectable-text PDFs up to 2 MiB; extracted text is searched and matching excerpts are shown as “Supported by: <document title>”, otherwise “No evidence found”.
- Admin-approved PNG/JPG logo upload and 5 MiB ad-creative PNG/JPG/JPEG checks: Tesseract English OCR feeds the existing hybrid checker; OpenCV template matching can raise an advisory Medium “Approved logo not detected” finding with a confidence label.
- Persistent creative preview, input/type/size validation, friendly OCR/OpenCV fallback notices, additive SQLite storage, report/history/PDF export, revision comparison, and transparent 0–100 Brand Health Score.

## (b) Architecture and technology

FastAPI, Pydantic v2, SQLAlchemy, and SQLite provide the same-origin service and additive persistence. React/TypeScript/Vite, Tailwind CSS, and Recharts provide the UI. PyMuPDF extracts selectable PDF/evidence text; ReportLab exports audit reports; Pillow validates image data; pytesseract invokes Tesseract with a timeout; OpenCV performs multi-scale edge-based logo template matching. Optional OpenAI/Anthropic output is strict-JSON validated and advisory. Deterministic checks, locally matched evidence, reviewer annotations, OCR/logo findings, and final human decisions are saved as revision/workflow records. The Dockerfile builds the React bundle and serves it from one FastAPI container.

## (c) Testing details

- **53 pytest tests pass** in the final-round working copy (7 non-failing dependency deprecation warnings).
- The scripted judge main flow passes independently: sample/check → apply fix → recheck → submit → Reviewer approves → PDF export → dashboard updates.
- The visual regressions exercise actual local Tesseract OCR, rule-engine reuse on OCR text, OpenCV exact-logo match and missing-logo finding, persistence/refreshed image retrieval, Admin role enforcement, bad/wrong files, 5 MiB limit, and friendly OCR-unavailable fallback.
- Evidence tests cover seeded cotton document, exact/high-overlap support, no-match message, text/PDF extraction, role/size/type errors, and guideline versioning.
- Reviewer-learning tests cover the 3-label/>50% threshold, one-step severity floor, dashboard rates, per-creator grouping, and synthetic seeds.
- The TypeScript/Vite production build passes. `git diff --check` and backend `compileall` pass.
- Webdev published version `c0b61cb29e0d9dcbd63d024c8439df375a454796`; public read-only smoke checks returned 200 for home, health, guidelines, and dashboard. The Webdev hosted container build succeeded. A separate local Docker build and incognito-browser session were not run; no live OpenAI/Anthropic response was tested.

## (d) Status of each requested improvement: committed vs delivered

**Checkpoint and delivery status:** the improvements, tests, and handoff docs are committed to the active Manus-managed canonical `main`. Application version `c0b61cb29e0d9dcbd63d024c8439df375a454796` is published at https://inkloom-mhp9bk9a.manus.space. Public read-only smoke checks passed. The source has not been transferred to a verified public GitHub repository.

1. **Reviewer-feedback learning — committed to Manus-managed `main` and included in the published version at https://inkloom-mhp9bk9a.manus.space.** Saved labels feed rates and future severity; threshold, downgrade message, dashboard panels, and synthetic-seed disclosure are covered by tests. Creator IDs are pseudonymous, not verified.
2. **Evidence library — committed to Manus-managed `main` and included in the published version at https://inkloom-mhp9bk9a.manus.space.** Admin text/PDF uploads, extracted storage, conservative matching, exact supported/no-evidence messaging, and seed are tested. Retrieval is lexical, not semantic.
3. **Visual ad-creative checks — committed to Manus-managed `main` and included in the published version at https://inkloom-mhp9bk9a.manus.space.** Tesseract and OpenCV are installed and exercised locally; tests cover rule-engine handoff and graceful fallback. The production Docker image has not been built locally, and template matching is not robust to all transformations.

All three improvements are included in the recorded and published Webdev version above. Their documented limitations still apply; the successful publish does not imply semantic evidence retrieval, robust transformed-logo recognition, real authentication, or live AI-provider availability.

## (e) Other improvements in this round

- Added visible dashboard notes distinguishing synthetic seed examples from real reviewer labels and describing creator IDs as browser-scoped/pseudonymous.
- Added in-flight guards for check, evidence-upload, and logo-upload submissions; bad images have extension, decoded-format, byte-size, pixel-count, timeout, and safe-serving checks.
- Kept all SQLite schema changes additive and documented the real limitations of retrieval, OCR, logo matching, optional AI, and persistent container storage.

## (f) Roadmap

1. Complete and record the public source-repository transfer/access separately before claiming the intended GitHub clone URL is available; the Webdev application release is already live.
2. Replace demo-role labels with real identity/authorization before using confidential brand data.
3. Upgrade evidence matching to reviewed semantic retrieval with document provenance and human confirmation.
4. Add visual logo detection robust to transformations and support multiple approved logo versions, while keeping all results advisory.
5. Add evaluation/audit tools and measure false positives on real, human-labeled samples before tuning severities further.
