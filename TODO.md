# Inkloom AI — Outcome Checklist

## Baseline core and stabilization
- [x] Existing core supports DemoWear guidelines, content/PDF checks, deterministic checks, optional strict-JSON AI with rules-only fallback, evidence findings, fix/recheck, review decisions/comments, history, dashboard, PDF report, role switcher, seed/reset, `/health`, Dockerfile, Render blueprint, and handoff docs. Audit evidence: baseline 20 tests passed; after stabilization 24 tests and frontend production build passed; the scripted API journey succeeds.
- [x] Applying a finding updates the draft, but after that text changes, disable other stale finding offsets until the creator re-runs the check; do not apply offsets from an older revision to modified text.
- [x] “Load demo data/brand” must not delete saved work; keep a separate “Reset demo data” control, clearly identify its effect, and confirm before the UI triggers destructive reset.
- [x] Do not silently truncate extracted PDF text at the 25,000-character draft limit. Reject an overlong extraction with a clear message so the unreviewed tail cannot be mistaken for checked content.
- [x] On banned-term update, reject a case-insensitive duplicate with a clear conflict response instead of allowing a database uniqueness exception/500.

## A. AI reliability, input protection, and safe rendering
- [x] Use one AI wrapper class with a 12-second timeout, one retry with backoff, strict Pydantic JSON validation, a SQLite response cache keyed by hash(draft + guideline version), and a daily call cap read from `AI_DAILY_CAP` (default 300). On missing key, invalid response, provider outage, timeout, or exhausted cap, gracefully fall back to rules-only and visibly display exactly “AI analysis unavailable. Showing rule-based checks only.”
- [x] Add per-IP rate limiting on the check endpoint and a 2 MB upload limit; verify PDF magic bytes before extraction. Escape all rendered text in the UI and exported reports.

## B. Improved rule/AI check pipeline
- [x] Detect prices in “Rs”, “INR”, rupee-symbol, and numbers introduced by “only”, “at”, “just”, or “from” formats, including comma grouping and decimals, and compare with the selected product’s approved price.
- [x] Detect banned terms and preferred terminology, missing required disclaimers, excessive exclamation marks, ALL-CAPS words, and evidence-needed superlative claims such as “best”, “#1”, and “top-rated”; matching approved claims must pass and show their evidence.
- [x] Use AI only for contextual tone and unsupported-claim checks. Treat unsupported claims as “needs evidence”, never false. Send only the draft and relevant guideline snippets. Merge/deduplicate rules and AI findings; every finding carries source badge (Rule/AI), severity, evidence reference, suggested fix, and an AI confidence/uncertainty label.
- [x] Build one checker pipeline that accepts text plus optional asset metadata. The final-round image path extracts OCR text and feeds it through the same deterministic/AI checker; visual logo findings join the saved revision. Evidence retrieval and reviewer-feedback learning are integrated after the hybrid check. Evidence: image/evidence/feedback regressions and full API flow.

## C. Brand Health Score
- [x] Calculate a transparent 0–100 score per draft with a stated severity-weight formula and breakdown. Show the score on the report, history, and dashboard, with a tooltip explaining the weights/formula.

## D. Revision comparison
- [x] Let users compare any two revisions of a draft with an inline text diff and show which findings were resolved and which are new.

## E. Batch check
- [x] Let a user paste several drafts or upload a CSV containing a `text` column. Show a summary table with score, top issue, and status; each row opens the full report.

## F. Reviewer power tools
- [x] Let a Reviewer dismiss an individual finding with a reason. Store and show the reason in report/history/PDF; dismissal must not change check-engine behavior. Evidence: revision-scoped `FindingReview` rows, history events, report UI, PDF annotations, and regression coverage.
- [x] Add a comment thread per draft. Store a nullable `reviewer_label` with values `valid`, `false_positive`, or null. The final-round feedback engine now consumes those labels for future checks. Evidence: persisted `DraftComment` / `FindingReview` rows, report/reviewer UI, history/PDF trail, reviewer-learning tests.

## G. Guideline versioning
- [x] Increment a guideline version on every guideline edit. Store the version used by each check. When guidelines changed since a check, show exactly “Guidelines changed since this check. Re-run?” with a one-click re-run. Evidence: all 20 Admin mutation routes bump the version transactionally; additive SQLite migration and stale-submit guard; report prompt and re-run; version regression tests.

## H. Evaluation page
- [x] Add `/evaluation` with a labeled set of about 30 sample drafts and expected finding types, plus a “Run Evaluation” action. Compute and show measured precision, recall, and false-positive counts separately for rules-only, AI-only, and combined predictions; include sample counts and never invent scores. If AI is unavailable, run rules-only evaluation, explicitly report that AI evaluation is unavailable and identify any combined result as rules-only fallback; retain `/health` and never require an API key for the judge flow.

## I. Audit log and service status
- [ ] Add an audit-log page listing role, action, draft identifier/title, and timestamp, with CSV export. Keep records append-only in normal operation, do not include API keys, extracted evidence text, uploaded image bytes, or full comment bodies in the export, and preserve existing workflow/comment/history behavior.
- [ ] Add `/status` showing database health, AI configuration/reachability status (do not claim reachable if not probed), AI calls used vs cap, guideline version, and app version; retain `/health`. Do not send paid AI probes automatically on page load; a provider not explicitly probed is “unknown/not probed”.

## J. Judge usability and persistent demo data
- [ ] Home has a three-step “How to use” banner; buttons “Load demo data” and “Try a sample draft”; and about 12 prefilled historical sample drafts tagged “sample data” with findings so the dashboard is not empty. Include a “Reset demo data” action with an explicit warning/confirmation that names what is deleted; never make “Load demo data” destructive.
- [ ] Provide friendly empty states, visible loading/skeleton feedback, toast errors, a page-level error boundary, responsive layout, accessible contrast, text severity labels (not color alone), and Ctrl+Enter to run a check. Preserve existing keyboard and form behavior and do not double-submit or auto-publish.

## K. Automation and final-round hardening
- [ ] Add a GitHub Actions workflow that installs the project dependencies, runs pytest on pushes and pull requests, and does not expose secrets. Ensure `.gitignore` covers `.env`, `*.db`, `node_modules`, and `dist`; inspect Dockerfile, `render.yaml`, and `.env.example` and correct only demonstrable defects. Do not push to a user GitHub remote or claim external CI execution unless performed.
- [ ] Fix only reproducible issues found during H–K; keep SQLite changes additive, tests isolated, the app login-free, secrets undisclosed, and the published service/data untouched. Run the full suite and scripted main flow after each outcome group. Record unresolved infrastructure/identity/provider limits honestly rather than claiming every scope boundary is a defect.

## Final hackathon round — current user-authorized scope
- [x] **1. Reviewer-feedback learning:** The `valid` / `false_positive` labels reviewers already save change future check behavior. For each finding rule, the false-positive rate is computed from saved labels. At least 3 labels and strictly more than 50% false positives lowers that rule's severity one level and shows exactly “Downgraded by reviewer feedback”. The Dashboard shows per-rule false-positive rates and recurring mistakes per rule and pseudonymous creator/session identifier. Seeded demo feedback keeps the panels populated; it is explicitly marked as synthetic. Evidence: `reviewer_feedback.py`, dashboard/API/UI, threshold/creator tests, and main-flow checks.
- [x] **2. Evidence library:** Admin uploads titled UTF-8 text/Markdown or text-based PDFs (2 MB max); selectable text is extracted and stored. When an unsupported claim is found, a conservative lexical matcher searches the library. A match shows “Supported by: <document title>” with its excerpt and suppresses the unsupported-claim finding; otherwise the report says “No evidence found”. Seeded document: “Cotton Certificate CC-2026-14”. Evidence: upload/match/no-match/PDF/validation/version tests and Admin/report UI.
- [x] **3. Visual brand checks for ad creatives:** Accept PNG/JPG (5 MB max) and verify file type/content. Tesseract OCR extracts text and sends readable copy through the existing hybrid rules/AI checker. Admin uploads an approved logo image; OpenCV template matching searches the creative. If not detected, create a Medium “Approved logo not detected” Visual finding with a confidence label. Missing OCR/OpenCV and decode failures return friendly statuses rather than crashing. Evidence: installed Tesseract/OpenCV in local runtime; real OCR, rule reuse, exact match, missing-logo, invalid/oversized files, OCR-unavailable, persistence, and refresh tests; Dockerfile installs Tesseract and requirements pin OCR/OpenCV packages.
- [x] **Cross-cutting safety and delivery:** Empty text, wrong/oversized files, missing OCR/OpenCV, and repeated check/evidence/logo submissions are guarded; image bytes and metadata persist for refreshed report/reviewer views. The full pytest suite and main flow were run after each feature step. README, ARCHITECTURE, KNOWN_ISSUES, CHANGELOG, and SUBMISSION were updated. Published successfully from the active existing Webdev project at https://inkloom-mhp9bk9a.manus.space (version `c0b61cb29e0d9dcbd63d024c8439df375a454796`); read-only public smoke checks returned 200. No project transfer or recreation was performed.

## Quality, deployment package, and truthful submission
- [x] Keep database changes additive: preserve existing data and tables; add columns/tables without dropping tables. Add tests for new features, keep at least 20 passing tests, and retain a scripted end-to-end main-flow test. Evidence: idempotent SQLite migrations, `create_all` for new tables, migration regressions; 53 pytest cases pass, including reviewer learning, evidence, OCR/OpenCV, and the scripted main flow.
- [ ] Add a GitHub Actions workflow that runs tests on push. Ensure Dockerfile, `render.yaml`, `.env.example`, and `.gitignore` correctly exist; ignore `.env`, `*.db`, `node_modules`, and `dist`; commit no secrets.
- [x] Update README with judge steps, architecture diagram, stack, clone/run/deploy/test instructions, and limits. Update `ARCHITECTURE.md`, `KNOWN_ISSUES.md`, and `SUBMISSION.md`; maintain `CHANGELOG.md`. Evidence: final-round feature descriptions, install/run commands, limitations, test results, and status are synchronized across those files.
- [x] `SUBMISSION.md` contains ready-to-paste completed features, architecture/technology summary, exact test results, committed-vs-delivered status for each current improvement, other improvements, current gaps, and roadmap.
- [x] After upgrades, run the full flow: load sample → check → apply fix → re-run → submit → Reviewer approves → export PDF → dashboard updates; run the no-key rules-only fallback tests. Inspect Docker/Render/config declarations; do not claim a local Docker image build, GitHub transfer, Render deployment, live-provider success, or incognito browser test unless actually performed. Evidence: scripted main-flow test, fallback test, config and Dockerfile inspections.

## Scope boundary
- [ ] This request authorizes continuing the already-planned H–K improvements, but does not authorize publishing a new version, changing deployment configuration, transferring the repository, adding login, or entering secrets. Preserve the existing project and deployment.
- [x] No GitHub transfer or Render deployment has been performed. OCR/OpenCV behavior was exercised locally; the deployed app is operational and the hosted container build succeeded. The live health endpoint reports AI unconfigured, so do not claim provider-backed AI behavior.
