# Inkloom AI

Inkloom AI checks marketing copy against an approved brand before publication. It combines deterministic rules with optional AI, retrieves supporting evidence, records reviewer feedback, checks ad-creative text/logo placement, and routes every draft to a human. Nothing is auto-published. The DemoWear judge flow needs no account and no provider key.

## Try it as a judge

1. Open the deployed app and choose **Try a sample draft** (or **Load demo data**; it preserves saved work).
2. Run **Run Brand Check**. Review highlighted findings, guideline/evidence references, suggestions, severity, and the Brand Health Score. Try an Ad creative PNG/JPG from **New Content** to run OCR and logo checks.
3. Switch to **Reviewer** to label/dismiss findings, add a comment, and approve or request changes. Open Dashboard, History, revision comparison, and the PDF report. Nothing is published.

Admin can manage guidelines, upload supporting text/PDF evidence, and upload the approved logo image. The Evidence Library seeds **Cotton Certificate CC-2026-14**, clearly marked as demo data. The dashboard includes feedback rates and recurring issues. Sample/demo feedback is labeled as synthetic; it keeps the first-use panels populated.

## What works

- DemoWear seed brand, ₹599 Classic T-shirt, approved cotton claim/evidence, prohibited/preferred terms, tone rules, disclaimer, logo metadata, and sample drafts.
- Creator/Reviewer/Admin role switcher, checks for text and selectable-text PDFs, JSON/CSV batch checks, apply-fix/recheck, workflow decisions, comments, finding labels/dismissals, history, any-two revision diff, dashboard, and PDF export.
- Rule checks for prices, terminology, disclaimers, tone, and evidence-needed claims; optional strict-JSON AI with validation, timeouts, caching, and visible rules-only fallback.
- Reviewer learning: false-positive rate per finding rule; at least 3 labels and strictly more than 50% false positives lowers the rule's severity one level and displays **Downgraded by reviewer feedback**. Creator grouping uses a pseudonymous browser-scoped ID, not a verified identity.
- Evidence library: Admin upload of UTF-8 `.txt`/`.md` or text-based `.pdf`, maximum 2 MiB. Text is extracted and stored. A conservative sentence-level lexical match can suppress an unsupported-claim finding and show **Supported by: <document title>** plus the excerpt; otherwise it says **No evidence found**.
- Ad-creative checks: verified PNG/JPG/JPEG upload up to 5 MiB; Tesseract English OCR text enters the existing hybrid checker. OpenCV searches for the one uploaded approved-logo template. A non-match adds a Medium visual finding with a confidence label. Uploaded images persist for the report/reviewer preview.
- Additive SQLite migrations preserve existing rows. `/health` remains available. Docker includes Tesseract and compiles the frontend into the FastAPI-served app.

## Architecture and safeguards

See [ARCHITECTURE.md](ARCHITECTURE.md) for the full data flow. Exact facts stay deterministic; AI findings require valid spans/references and remain advisory. Uploaded evidence is matched locally with lexical overlap and is not sent to the LLM. OCR/OpenCV failures return notices and do not stop text/rule checks. Image upload validates both extension and decoded file format, limits files to 5 MiB and images to 20 megapixels, and bounds OCR/matching work. Reviewer labels affect future checks only; they do not rewrite saved reports.

The transparent score is `max(0, 100 − 25×High − 12×Medium − 5×Low)`. Informational findings do not deduct points. Guideline version snapshots prevent submission of stale checks.

## Clone and run locally

The intended GitHub destination is `https://github.com/sabirparvez-dotcom/inkloom-ai.git`. **The Webdev-to-GitHub transfer/public visibility has not been verified in this working copy.** The clone command below will work once that repository is available to the cloning user; do not treat the URL as proof of a public repository.

```bash
git clone https://github.com/sabirparvez-dotcom/inkloom-ai.git
cd inkloom-ai
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
cd frontend
npm ci
npm run build
cd ..
uvicorn backend.inkloom.main:app --host 0.0.0.0 --port 3000
```

Open `http://localhost:3000/`. Vite output is built into `backend/inkloom/static/` and served by FastAPI; browser calls are same-origin. Python 3.12 and Node.js/npm are required. For local image OCR, install Tesseract 5 with its English language data (Ubuntu/Debian: `sudo apt-get install tesseract-ocr tesseract-ocr-eng`; macOS: `brew install tesseract`). If Tesseract is absent, OCR shows a friendly unavailable notice and the rest of the workflow continues. OpenCV, Pillow, and their Python dependencies are installed from `requirements.txt`.

Run the complete suite from the repository root:

```bash
.venv/bin/python -m pytest -q
```

SQLite defaults to `./inkloom.db`; set `DATABASE_URL` to change it. Optional AI uses `OPENAI_API_KEY` or `ANTHROPIC_API_KEY` from the server environment. Do not commit `.env` or keys. Visitors never need to configure secrets.

## Docker and deployment

Build and run the single-service image:

```bash
docker build -t inkloom-ai .
docker run --rm -p 3000:3000 -v "$PWD/data:/var/data" -e DATABASE_URL=sqlite:////var/data/inkloom.db inkloom-ai
```

The Dockerfile installs Tesseract English data, builds the Vite frontend, installs pinned Python dependencies, serves the static bundle through FastAPI, and honors `PORT` (3000 by default). The `/health` endpoint is the container readiness check. The Render blueprint is provided, but no Render deployment is claimed here. A persistent disk and `DATABASE_URL=sqlite:////var/data/inkloom.db` are needed for durable SQLite on a host whose container filesystem is ephemeral.

The final-round application is published from Webdev checkpoint `c0b61cb29e0d9dcbd63d024c8439df375a454796` at **https://inkloom-mhp9bk9a.manus.space**. Publication succeeded. Read-only public checks returned 200 for `/`, `/health`, `/api/guidelines`, and `/api/dashboard`; the live guide/evidence seed and feedback metrics were present. The deployment reports AI unconfigured and displays the rules-only notice, as designed; judges do not need a key.

## Verification and current release status

The final-round code is committed to Manus-managed `main` and the application version above is live. The code has **53 passing pytest tests**; the scripted judge flow and production TypeScript/Vite build pass. It has not been transferred to a verified public GitHub repository; the intended clone destination remains unverified. The live AI status is rules-only because no provider key is configured. See [KNOWN_ISSUES.md](KNOWN_ISSUES.md) and [SUBMISSION.md](SUBMISSION.md) for limitations and submission details.
