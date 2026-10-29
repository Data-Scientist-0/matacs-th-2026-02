# MATACS-TH-2026-02 — AI Proposal Triage Design

**Full Name:** Muhammad Hassan  
**Reference:** MATACS-TH-2026-02  
**Role:** Associate Software Developer  
**Time spent:** Approximately 60 minutes  

---

## Repository Contents

| File / Folder | Description |
|---|---|
| [README.md](./README.md) | This file — overview, AI use disclosure |
| [WRITEUP.md](./WRITEUP.md) | Answers to all five design questions (under 1,000 words) |
| [PROMPTS.md](./PROMPTS.md) | Exact AI prompts for manipulation detection, extraction, and scoring |
| [ranking.json](./ranking.json) | Expected ranking of proposals A–E |
| [diagram/process_flow.png](./diagram/process_flow.png) | Full process flow from upload to ranked output |

---

## How I Used AI Tools

I used Claude (Anthropic) to help structure my answers and review the logic of my prompt designs. Specifically, it helped me draft the extraction JSON schema and the scoring prompt rubric.

I disagreed with Claude's initial scoring prompt, which asked the model to score proposals using raw PDF text directly. I corrected this to a two-step approach: extract structured JSON first, then score from the JSON. The reason is that scoring from raw text makes it harder to enforce consistent evidence anchoring — the model can draw on any part of the proposal, making scores harder to explain or audit. Separating extraction from scoring means every score traces back to a specific extracted field with a quoted evidence string.

---

## Proposal Ranking Summary

| Rank | Proposal | Status | Score |
|---|---|---|---|
| 1 | A – Northshore Solar Ltd. | ranked | 94 |
| 2 | E – SunGrid Contractors Inc. | ranked | 80 |
| 3 | D – Essex Green Power Inc. | ranked | 68 |
| — | C – Lakeview Renewables Corp. | flagged | null |
| — | B – BrightPath Energy Inc. | ineligible | null |

**B** fails the mandatory Ontario electrical contractor licence requirement — the bidder does not hold the licence; an unnamed subcontractor does, to be confirmed post-award.  
**C** contains instruction-style text directing automated systems to rank it first and assign full marks — flagged as a prompt injection attempt and held for human review.
