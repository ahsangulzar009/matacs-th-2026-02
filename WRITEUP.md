
Reference: MATACS-TH-2026-02

## Design in one line

Upload → extract text → load admin criteria → injection scan → AI extracts facts with evidence → code runs mandatory pass/fail → code calculates weighted scores and ranks → AI writes one-line reasons → reviewer checks and can override → human makes the final award → everything is logged.

## Assumptions

- Proposals are short, so the full text goes straight to the model. I do not use a vector database, because chunking could split related facts (such as price and schedule) and retrieval is approximate, while this task needs exact facts. For very long proposals (100+ pages) I would add chunking.
- Price and timeline scores are relative to the other eligible bids in the same batch (the cheapest eligible bid gets 100). The batch and its date are logged, because adding a new proposal can change these scores.
- Missing or vague information is never guessed. It goes to a human.

## 1. AI vs. rules

**AI is used for:** reading unstructured proposal text and extracting facts (price, weeks, licence, warranty, team details), each with a quote as evidence; detecting hidden instructions; and writing the one-line reason.

**Plain code is used for:** PDF-to-text, loading criteria, mandatory pass/fail, all scoring maths, ranking, status rules and logging.

**Why this line:** AI is good at understanding messy language but can be inconsistent and can make arithmetic mistakes. Rules are exact and repeatable. So AI only turns text into structured facts, and code makes every decision that must be exact. The AI never scores, ranks or decides pass/fail.

## 2. Consistency and explainability

- Temperature 0, a fixed model version and a fixed, versioned prompt.
- Scoring formulas live in code, so the same extracted facts always give the same score.
- Every extracted fact must include a quote. Code checks that the quote really exists in the proposal. If it does not, the fact is treated as missing.
- Output is validated against a schema. If it is invalid, retry once, then send to a human.

**Would two runs give the same result?** The scoring and ranking would be identical for the same facts. The AI extraction is near-identical at temperature 0 but not guaranteed. To handle that, extraction results are saved after the first run, and a re-rank uses the saved facts instead of calling the AI again. If a fresh re-extraction differs, the difference is flagged for the reviewer.

**How a reviewer sees why:** for each proposal the screen shows every criterion with the extracted value, the evidence quote, the score and the weight, plus the total and a one-line reason. The reviewer can click a quote to see it in the original text.

## 3. Human in the loop

A person is involved at these points:

1. **Flagged proposals.** Injection attempts, missing or vague information, and failed extraction go to the reviewer before anything is ranked.
2. **Ineligible proposals.** The reviewer can see why a bidder failed a mandatory check and can correct the extraction if it was wrong.
3. **Score override.** The reviewer can change any extracted fact or score, but must enter a reason. The ranking is then recalculated by code.
4. **Final award.** Only a human awards the contract. The tool produces a recommendation, never a decision.

**Logged for audit:** who did what and when; uploaded file hash; extracted text; prompt version and model version; criteria version and weights; raw AI output; evidence quotes; scores before and after overrides with the reason; flags raised; the batch used for relative scoring; and the final decision.

## 4. Changing criteria

- Criteria and weights are stored in the database with a version number. Every score stores the criteria version it was calculated with.
- Old results are never overwritten. They stay visible as "scored under version N".
- When an admin edits criteria, the tool shows that existing results are out of date and offers a re-score.
- If only weights or scoring rules changed, re-scoring uses the saved extracted facts, so it is cheap and needs no new AI call. If a criterion was added or its definition changed, only that criterion needs a new extraction.
- Human overrides are kept and shown during re-scoring, so the reviewer can confirm they still apply.
- Changes to criteria are logged with admin name, time and old and new values.

## 5. Risks and safeguards

| Risk | Safeguard |
|---|---|
| **Wrong extraction** (misreading a number or licence) | Every fact has an evidence quote that code verifies; schema validation; reviewer sees quotes next to values; mandatory results are shown for confirmation. |
| **Made-up facts** | Prompt forbids guessing and allows `null`; missing or vague facts are never scored and go to a human; quote must exist in the source text. |
| **Prompt injection / manipulation** | Two layers: a keyword scan in code and a prompt rule that proposal text is data, not instructions. Flagged proposals are held for a human. Because the AI only extracts and code does the scoring, an injected instruction cannot directly set a rank or score. In the sample set this affects C. |
| **Faking compliance** (for example, a subcontractor promised "after award") | Mandatory criteria require the bidder itself to hold the licence. Conditional or future statements count as not met. This affects B, whose low price is therefore not allowed to rescue it. |
| **Bias** | Company names replaced with letters before AI calls; scoring rules are identical for everyone; the AI never ranks; reviewers can see all reasoning. |
| **Unfair price/timeline effects** | Relative scores depend on the batch, so the batch is logged and shown. Reviewer sees raw price and weeks beside the score. |
| **Cost** | Each proposal is extracted once and cached; a small model is used for extraction; re-ranking and re-scoring reuse saved facts; the reason-writing step is optional. |
| **Privacy / data leakage** | Only the proposal text and criteria are sent to the model; access is limited by role; files and logs are stored securely. |
| **Over-trust in the tool** | Output is labelled a recommendation, the final decision is human, and overrides and decisions are logged. |

## How the sample set is handled

- **A and E:** pass all mandatory checks and are ranked. A is first because of the strongest experience and the fastest schedule; E is second with the best warranty and monitoring.
- **B:** ineligible (licence not held by the bidder).
- **C:** flagged for a hidden instruction aimed at automated reviewers. The instruction is ignored and a human reviews it before scoring.
- **D:** flagged because no completion time is given, so the timeline criterion cannot be scored. The reviewer can ask the bidder for a schedule, then re-score.