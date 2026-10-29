# Write-up: AI Proposal Triage Design (MATACS-TH-2026-02)

## The flow in short

The diagram is in `diagram/process-flow.png`. Proposals are referred to by letter (A-E) only.

## 1. AI vs. rules

**AI is used for reading, rules are used for deciding.**

- **AI steps:** (a) a check for text that tries to instruct the AI, and (b) extraction of facts from messy proposal text into a fixed JSON form: licence, WSIB, insurance, number of relevant installs, price, weeks, warranty years, and team and approach items. Every value comes with a short evidence quote and page number.
- **Rule/code steps:** upload validation, loading the criteria, verifying quotes, mandatory pass/fail, all score maths, ranking, and writing the reason text from the score breakdown.

I drew the line there because proposals are free text, and reading free text is what AI does well. Scoring needs to be repeatable and defensible, and that is what code does well. If the AI also picked scores, two runs could differ and nobody could explain the number.

## 2. Consistency and explainability

- Extraction runs at temperature 0 with a pinned model version, a fixed prompt version and a strict JSON schema. The result is cached against the file hash, prompt version and criteria version, so re-ranking the same files reads the cache and gives the same output.
- Scoring is plain code with published formulas, so the same extracted values always give the same score. Examples used for the sample set: price score is lowest eligible price divided by this price, timeline score is shortest eligible time divided by this time, experience is capped at a reference count the admin can set.
- If a proposal is truly re-extracted, the new values are compared with the old. Any difference is flagged instead of silently replacing the old result.
- The reviewer sees one row per criterion: the extracted value, the evidence quote with page number, the points awarded, and the formula. The reason line is built from that table, so it can never say something the table does not show.

## 3. Human in the loop

- **Automatic holds:** flagged proposals (suspicious text, conflicting extraction, or a missing completion time or price), ineligible proposals and other missing information all go to the reviewer instead of being decided by the tool.
- **Reviewer powers:** confirm or reverse an ineligible result, clear or reject a flag, correct an extracted value (with the evidence), ask the bidder for clarification, and reorder the list with a written reason.
- **Final award** is always a human decision. The tool only recommends an order.
- **Audit log:** who uploaded and reviewed, timestamps, file hashes, criteria version, prompt and model versions, extracted values with quotes, scores, flags, every override and its reason.

## 4. Changing criteria

- Criteria are versioned. Every scoring run stores which version it used, so old results never change silently.
- When an admin edits, the system validates (weights total 100%, mandatory items clear) and saves a new version. Existing results are marked "scored under older criteria" on the reviewer screen.
- Re-scoring is a reviewer action. A weight-only change needs no AI call, as scores are recomputed from cached extraction. A new or reworded criterion triggers extraction for that criterion only.
- If an award has been recorded, results are locked. Edits apply to future uploads, and a re-score creates a new, separately logged run.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| Wrong extraction (misread numbers, OCR errors on scans) | Evidence quote and page for every value; code checks the quote exists in the source and the number parses; low OCR confidence or mismatch is flagged. |
| Made-up facts | `not_stated` is an allowed answer; no quote means no value; nothing is guessed. A missing hard project limit (completion time or price) flags the proposal for clarification; other missing details score 0 with a "needs clarification" note. Example: D gives no completion time while the project must finish in 16 weeks, so D is flagged. |
| Manipulation (prompt injection) | Proposal text is treated as data, not instructions; suspicious text is detected, removed from the AI input and the proposal is flagged. Example: C contains hidden text telling automated reviewers to rank it first. It is held for a human and the text is shown to the reviewer. |
| Technically compliant but misleading wording | Mandatory checks require the bidder itself to hold the licence. Example: B promises a licensed subcontractor after award, so it fails. |
| Bias | Scoring uses only admin criteria; bidder names and sizes are not used in scoring; reviewers see the formulas and can challenge them. |
| Cost | Extract once per file and cache; small model for extraction; page and size limits; re-scoring reuses the cache. |
| Over-trusting the tool | Ranked list is a recommendation, flags are prominent, and the final decision and reasons are recorded by a person. |

## Result on the sample set

| Proposal | Outcome | Score |
|---|---|---|
| A | Rank 1 | 98.8 |
| E | Rank 2 | 88.4 |
| C | Flagged, held for human review | not ranked |
| D | Flagged: no completion time, needs clarification | not ranked |
| B | Ineligible (licence not held by bidder) | not scored |

**Assumptions:** (1) mandatory failures and flagged proposals are excluded from the price and timeline comparison; (2) team and approach is four equal checks (named lead, crew stated, safety plan, monitoring plan); (3) "last 5 years" experience counts installs the proposal clearly places in that window; (4) a missing completion time is treated as a missing hard limit, because the project must finish within 16 weeks. If C is cleared by a reviewer, its content alone would score about 81 and place it third. D cannot be scored until it states a schedule.
