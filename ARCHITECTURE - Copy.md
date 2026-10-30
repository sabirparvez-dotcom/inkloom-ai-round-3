# Inkloom AI — Architecture

## Data flow

1. **Creator input.** The same-origin React client sends text, selectable-text PDF, batch JSON/CSV, or an Ad creative PNG/JPG to FastAPI. Validation checks channel, product, role label, size/type, file contents, and text limits. PDF and evidence-document extraction use PyMuPDF. Ad-creative images are verified by Pillow and stored with metadata; a pseudonymous browser-scoped creator ID supports demo analytics without claiming verified identity.
2. **Guideline snapshot.** The checker loads the selected product, relevant banned/preferred terms, approved claims, enabled tone rules, applicable disclaimers, and the current monotonic guideline version. Every check stores that version. AI receives the draft and relevant guideline snippets, not unrelated drafts or the evidence-library corpus.
3. **Deterministic rules.** Rules compare prices to the selected product, detect banned/preferred terms, required disclaimers, configured tone heuristics, and evidence-needed superlatives. These exact-fact checks do not depend on AI. Unsupported language is marked “needs evidence”, never “false”.
4. **Optional text analysis.** For readable text and OCR output, an optional OpenAI/Anthropic call applies strict JSON validation with a timeout, one retry, caching, and a daily cap. Missing keys, provider failures, invalid responses, or the cap produce a visible rules-only fallback. OCR-empty creative skips the LLM instead of sending empty placeholder text.
5. **Evidence-library retrieval.** After hybrid findings are merged, the checker searches Admin-uploaded UTF-8 `.txt`/`.md` and selectable-text PDFs. It compares normalized claim words with extracted sentences using exact phrase or high lexical overlap. On a match it attaches the title/excerpt as a verified claim and suppresses the “needs evidence” finding; without a match, the report shows “No evidence found”. This is deterministic lexical retrieval, not embeddings, legal validation, or proof that a document is authentic. Uploaded evidence text is not sent to the LLM.
6. **Reviewer-feedback learning.** The engine aggregates one saved `valid`/`false_positive` label per reviewed finding record and explicitly synthetic seeded demo examples. For each finding type, at least 3 labels and a false-positive rate strictly above 50% lower future findings by one severity step (High→Medium→Low→Info) and add “Downgraded by reviewer feedback”. The saved historical finding is not rewritten. The dashboard shows rates per rule and recurring issues by rule and pseudonymous creator/session ID. Seed rows are demo examples, not actual people or past real review activity.
7. **Visual brand checks.** Tesseract OCR is time-limited and capped to 25,000 extracted characters; its text enters the same hybrid checker. OpenCV edge-based multi-scale template matching compares the creative to the single Admin-uploaded approved logo image. When the reference is present and the best match is below the threshold, the saved revision receives a Medium `Visual` finding, a cautious confidence label, and the best-match score. The original image persists for report and Reviewer previews. Missing OCR/OpenCV produces friendly status text; neither package failure crashes the text workflow.
8. **Creator correction and human approval.** Applying a suggestion changes editable text; the creator must re-run the checker. The API blocks submitting stale text or a revision whose guideline version has changed. A Reviewer may label/dismiss findings with a reason, comment, approve, or request changes. Nothing is auto-published.
9. **Audit and outputs.** SQLite stores brand guidelines/version state, evidence text, approved logo and creative bytes/metadata, drafts/creator IDs, reviewer-feedback examples, revisions/findings, review annotations, comment threads, and workflow events. Existing SQLite data is preserved by additive migrations. History, any-two revision comparison, dashboard, and PDF export use saved records; image bytes are fetched with `Cache-Control: no-store` and `X-Content-Type-Options: nosniff`.

## Why a hybrid checker

Approved prices, terminology, disclaimers, and formatting rules have expected values; deterministic checks are easier to explain and less likely to invent facts. AI adds contextual claim/tone coverage, but is advisory, span-validated, and remains subject to human approval. Evidence-library matching is local and deterministic. Visual template matching is a separate signal, not semantic image understanding.

## Reviewer feedback and Brand Health Score

The dashboard rate uses saved reviewer labels plus clearly identified synthetic demo examples. The downgrade threshold is explicit: `label_count >= 3 AND false_positive_rate > 0.50`; the current finding is lowered one step and carries the explanatory note. This is a lightweight rule-level adjustment, not model training or per-user personalization. Creator-level aggregates use a local browser identifier and can be reset or changed by the browser owner.

Each saved revision score is `max(0, 100 − 25×High − 12×Medium − 5×Low)`. Informational findings carry no deduction. A score is a triage indicator, never an approval decision.

## False-positive and stale-result safeguards

- Rule findings cite current product/guideline records; AI findings must use a supplied guideline reference and an exact draft span.
- Approved claim records and matched evidence excerpts are surfaced rather than called false. A document match is not a guarantee of authenticity or sufficient substantiation; a human reviewer still decides.
- Reviewer labels alter only future severity for qualifying rule types. Historical findings/scores remain intact, and the exact downgrade note is visible.
- Evidence retrieval is bounded lexical matching. It can miss paraphrases and may match unrelated wording; no semantic or factual verification is claimed.
- Logo absence is advisory. The visual matcher can miss resized, recolored, rotated, cropped, occluded, or otherwise transformed logos and may occasionally match similar edge patterns.
- OCR runs with a timeout and output cap; the included model is English. Unavailable packages, unreadable images, or timeouts show a notice and do not crash text/rule checks.
- Every successful guideline mutation advances a version. Reports show the exact stale-guideline prompt and one-click recheck; the API blocks stale submission.
- Reviewer decisions remain human-controlled. No social publishing path exists.

## Storage and deployment boundary

SQLite defaults to a local file. A container host with ephemeral filesystems needs supported persistent storage and a configured `DATABASE_URL` to retain data across restarts. The Dockerfile installs Tesseract and English data and builds the frontend into the FastAPI service. The role switcher is not authentication or access control; do not use it to protect confidential or regulated data.
