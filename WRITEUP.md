# WRITEUP.md — MATACS-TH-2026-02
# AI Proposal Triage Design

---

## 1. AI vs. Rules — Where is the line drawn?

**Rule-based steps:**
- Mandatory criteria checks (Ontario licence, WSIB, insurance) — these are binary pass/fail. The proposal either states a licence number held by the bidder or it does not. There is no interpretation needed, and using AI here would risk a model hallucinating a passing result where none exists. Rules are safer and faster for deterministic decisions.
- Weighted score calculation — once raw criterion scores exist, the weighted total is arithmetic. A formula is more reliable and auditable than asking a model to calculate it.
- Criteria versioning and stale-score detection — purely database logic.

**AI steps:**
- Text extraction from PDF proposals — proposals use varied language and structure. AI handles this better than regex.
- Manipulation detection — detecting instruction-like text in unstructured input requires language understanding.
- Scoring weighted criteria — fields like "team and approach" require judgment about what constitutes a named lead, a complete safety plan, or a monitoring commitment. AI interprets the nuance; the rubric in the prompt keeps it anchored.

The line is drawn at determinism. Where the right answer is a fact check or a formula, the code handles it. Where the right answer requires reading and interpreting unstructured text, the model handles it — with a structured output schema to keep results consistent.

---

## 2. Consistency and Explainability

**Repeatability:** All LLM calls use `temperature=0`. Prompts use structured output schemas (JSON with strict field names). The system runs the manipulation check and extraction pass once per proposal and caches the result — it does not re-score on every page load. If criteria change, all proposals are re-scored in a new batch (see Question 4).

**Explainability:** Every score is stored alongside the evidence quote extracted from the proposal. A reviewer opening a ranked result sees, for each criterion: the score, the reason in one sentence, and the exact text that was used to reach that conclusion. For example: Price score 84/100 — "Our total price is $238,000 CAD, including permits and utility connection." This lets a reviewer agree or dispute any individual score without reading the full proposal first.

If the same five proposals were submitted again, the scores would be identical because inputs, prompts, and temperature are all fixed.

---

## 3. Human in the Loop

**Review points:**
- Flagged proposals (manipulation detected) are held and not shown in the ranked list until a human reviewer explicitly releases or disqualifies them.
- Before any award decision, the reviewer sees the full ranked list with scores and evidence. The system produces a recommendation, not a decision.
- The reviewer can override any rank with a written reason before the list is finalized.

**Audit log entries (per proposal):**
- Timestamp of upload and each processing step
- Extraction output and manipulation check result
- Raw criterion scores and weighted total
- Reviewer ID and timestamp of any override
- Final status (ranked, flagged, ineligible, awarded)

No award can be recorded without a reviewer action. The audit log is append-only and includes the criteria version active at the time of scoring.

---

## 4. Changing Criteria

When an admin edits criteria or weights after proposals have been scored, the system:

1. Saves the new criteria as a new version (e.g., v1 → v2) and records the timestamp.
2. Marks all proposals scored under v1 as **stale**.
3. Shows a banner in the reviewer interface: "Criteria were updated. X proposals were scored under the previous version. Re-score all to compare on the same basis."
4. Provides a one-click re-score button that runs all proposals through the scoring prompt again under v2.
5. Prevents mixing v1 and v2 scores in the same ranked comparison — the reviewer must either re-score all or revert to v1.

Scores from different criteria versions are never displayed side-by-side without a clear version label.

---

## 5. Risks and Safeguards

| Risk | Description | Safeguard |
|---|---|---|
| **Wrong extraction** | AI misreads a licence number or price | Every extracted value includes a verbatim evidence quote; reviewer can spot-check before accepting results |
| **Hallucination** | AI invents a fact not in the proposal | Extraction prompt instructs the model to return null if a field is absent, never infer; null values trigger a manual check flag |
| **Prompt injection** | A proposal embeds instructions to manipulate scores (Proposal C in this set) | Dedicated manipulation-detection prompt runs first; flagged proposals are removed from auto-scoring and queued for human review |
| **Inconsistency** | Same proposal scores differently on two runs | temperature=0, fixed prompts, cached extraction results |
| **Bias** | Model favors proposals written in certain styles | Prompts reference only structured extracted fields (numbers, booleans) for scoring, not raw proposal prose |
| **Stale scores** | Criteria change but old scores remain visible | Criteria versioning and stale detection (see Question 4) |
| **Cost overrun** | LLM called repeatedly for large proposal sets | Extract once, cache JSON; scoring prompt uses only the cached JSON, not the full PDF text each time |
