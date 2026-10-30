# Known issues and scope boundaries

## Implemented, with important limitations

- **Reviewer-feedback learning:** Rates include clearly seeded synthetic examples so first-run charts and the threshold demonstration are populated. These rows are not actual reviewer history. The downgrade applies per finding type after 3+ saved labels and >50% false positives; it is a simple one-step rule adjustment, not trained AI or a causal accuracy guarantee.
- **Creator analytics:** Drafts are grouped using a pseudonymous browser-scoped ID. It is not a verified person, account, or secure identity. Clearing browser storage or changing devices can change attribution. The demo role switcher/API role field is not authorization.
- **Evidence library:** Matching is normalized sentence-level lexical overlap, not vector search, OCR, semantic entailment, or authenticity verification. It may miss paraphrases and may support unrelated wording with overlapping terms. Scanned PDFs are rejected; only UTF-8 `.txt`/`.md` and selectable-text PDF are accepted (2 MiB upload, 100 pages, 500,000 extracted characters). Human review is still required before relying on a match.
- **Visual checks:** OCR uses Tesseract English and is capped/time-limited. It is not a full ad-layout or image-understanding system. OpenCV edge template matching can miss rotated, cropped, recolored, resized beyond its search scales, overlaid, or otherwise transformed logos and may return a similar-looking false match. Only one approved logo image is active. A “logo not detected” finding is advisory with a confidence label, not proof that a logo is absent.
- **Runtime dependencies:** Local tests in this workspace used Tesseract 5, `pytesseract`, OpenCV, NumPy, and Pillow. The Dockerfile declares the same OCR/OpenCV runtime dependencies, but this exact Docker image was not built/launched locally in this session. If OCR/OpenCV is unavailable in another runtime, the UI shows a friendly status and the remaining checks continue.
- **AI provider:** A successful live OpenAI/Anthropic response was not exercised in this final-round run. The no-key/provider-failure fallback is tested, and rules-only operation remains usable.
- **Persistence:** Local SQLite is suitable for the hackathon demo. Container filesystems may be ephemeral; a supported persistent volume and matching `DATABASE_URL` are required for durable deployed history. The Webdev-managed database capability is not enabled for this service.

## Source and publication status

- The target GitHub repository is `sabirparvez-dotcom/inkloom-ai`, but this working copy still reports the Manus-managed repository as canonical. No code transfer or public-access verification has been completed. The clone URL in README is the intended destination, not proof that it is publicly available.
- The final-round application is published at https://inkloom-mhp9bk9a.manus.space, version `c0b61cb29e0d9dcbd63d024c8439df375a454796`. Read-only public smoke checks returned 200 for `/`, `/health`, `/api/guidelines`, and `/api/dashboard` using a browser User-Agent. OCR/logo analysis was tested locally; no creative upload was sent to the production database.
- No Render deployment, public GitHub push, real-provider success, or browser/incognito test is claimed here.

## Deliberately out of scope

- No real sign-in, verified identity, or secure role authorization.
- No social-media integration or auto-publishing; nothing is auto-published.
- No multilingual rule/model/OCR support.
- No scanned-PDF OCR, general visual brand recognition, ad-layout interpretation, or multi-version logo library.
- No semantic evidence search, authenticity/provenance verification, or automatic claim truth determination.
