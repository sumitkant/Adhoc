# Worked example — a full evaluation, exactly as the judge should output it

This is the report the judge produces for the Custom GPT instruction set below. Use it to see the expected level of detail, how evidence is written, how fixes use ⟨placeholders⟩ instead of invented facts, and how the Top 3 is derived mechanically. It also doubles as a regression test: a correctly configured judge should reproduce every verdict in this file.

## The submitted text

The author pasted their Custom GPT's Instructions field. No separate Description field was supplied, so C2 reads the first paragraph. No knowledge files were listed.

```
Procurement Policy Assistant

You are a helpful assistant for the procurement team. Answer questions about our procurement policy and help people fill in purchase requests.

Always be professional and accurate. Use the policy document to answer questions. If you are not sure, try to give the best answer you can.

When someone wants to raise a purchase request, collect the details and produce a summary they can paste into Coupa. Make sure the summary includes the vendor, the amount, the cost centre and the business justification. Keep it short.

Purchases over the threshold need VP approval, so remind the user about that. Follow the SOP for sole-source vendors.

Do not give legal advice. Avoid making mistakes about policy details. Don't be too verbose.

Format the summary nicely.
```

Line numbering (non-empty lines only):

```
L1 Procurement Policy Assistant
L2 You are a helpful assistant for the procurement team. Answer questions about our procurement policy and help people fill in purchase requests.
L3 Always be professional and accurate. Use the policy document to answer questions. If you are not sure, try to give the best answer you can.
L4 When someone wants to raise a purchase request, collect the details and produce a summary they can paste into Coupa. Make sure the summary includes the vendor, the amount, the cost centre and the business justification. Keep it short.
L5 Purchases over the threshold need VP approval, so remind the user about that. Follow the SOP for sole-source vendors.
L6 Do not give legal advice. Avoid making mistakes about policy details. Don't be too verbose.
L7 Format the summary nicely.
```

## The report

---

# Skill Quality Report — Procurement Policy Assistant
Rubric v1.0 · 2026-10-28 · Judge: ⟨model⟩ · 7 lines / 127 words · Attachments considered: none

## Summary
| # | Dimension | Score |
|---|---|---|
| 1 | Clarity & Specificity | 3/5 |
| 2 | Context Completeness | 3/5 |
| 3 | Task Structuring | 1/5 |
| 4 | Output Specification | 1/5 |
| 5 | Instruction Precision | 1/5 |
| 6 | Use of Examples | 0/5 |
| 7 | Robustness & Error Prevention | 1/5 |
| | **Total** | **10/35 (29%) — Weak** |

## 1. Clarity & Specificity — 3/5
| ID | Verdict | Evidence |
|---|---|---|
| C1 | YES | L2: "Answer questions about our procurement policy and help people fill in purchase requests." |
| C2 | YES | L2 (first paragraph; no description field supplied): "questions about our procurement policy", "fill in purchase requests" |
| C3 | YES | L6: "Do not give legal advice." |
| C4 | NO | SOP (L5) never expanded. VP is on the allowlist. |
| C5 | NO | count=2: L3 "best" ("try to give the best answer"), L7 "nicely" |

**Fixes**
- C4 · Where: L5 · Problem: "SOP" is never expanded, so the model must guess which procedure is meant. · Replacement: "Follow the Standard Operating Procedure (SOP) for sole-source vendors, ⟨file name⟩ § ⟨section⟩."
- C5 · Where: L3 and L7 · Problem: "best answer" and "nicely" leave the decision to the model. · Replacement for L3: "If the policy does not cover the question, say 'Not covered in the policy' and name the closest clause." Replacement for L7: "Format the summary as the 4-row table in the Output section."

## 2. Context Completeness — 3/5
| ID | Verdict | Evidence |
|---|---|---|
| X1 | YES | L2: "for the procurement team" |
| X2 | YES | L4: "the vendor, the amount, the cost centre and the business justification" |
| X3 | NO | L5: "over the threshold" — the threshold value is not given and no source is named; L3 "the policy document" is not a specific file name. |
| X4 | YES | L6: "Do not give legal advice." |
| X5 | NO | count=2: L3 "the policy document", L5 "the SOP" |

**Fixes**
- X3 · Where: L5 · Problem: the model is told to apply a threshold it has not been given. · Replacement: "Purchases over ⟨amount⟩ require VP approval (⟨policy file⟩ § ⟨section⟩); when the amount exceeds it, add the line 'Requires VP approval'."
- X5 · Where: L3, L5 · Problem: two documents are referenced but neither is named or attached. · Replacement for L3: "Answer only from ⟨Procurement_Policy_⟨year⟩.pdf⟩." Replacement for L5: see X3 and C4 above.

## 3. Task Structuring — 1/5
| ID | Verdict | Evidence |
|---|---|---|
| S1 | NO | NOT FOUND |
| S2 | NO | only L1 (title); no section labels |
| S3 | YES | all 2 conditionals have actions: L3 "If you are not sure" → "try to give the best answer"; L4 "When someone wants to raise a purchase request" → "collect the details and produce a summary" |
| S4 | NO | NOT FOUND |
| S5 | NO | no numbered steps (S1 failed) |

**Fixes**
- S1 · Where: replace L4 · Problem: the purchase-request workflow is one prose sentence; the model may reorder or skip parts. · Replacement: "Steps for a purchase request: 1. Ask for any of the four fields the user has not given. 2. Check the amount against the ⟨threshold⟩. 3. Produce the summary table in the Output section. 4. Run the checks in the Verify section."
- S2 · Where: whole text · Problem: no sections; output rules and scope rules are mixed into the workflow. · Replacement: reorganise under "## Purpose · ## Inputs · ## Steps · ## Output · ## Rules · ## Examples".
- S4 · Where: insert after L6 · Problem: "Keep it short" and "Make sure the summary includes…" have no stated winner. · Replacement: "If brevity and completeness conflict, completeness of the four fields wins."
- S5 · Where: see S1 · Problem: no steps exist to be atomic. · Replacement: the S1 steps above, one action each.

## 4. Output Specification — 1/5
| ID | Verdict | Evidence |
|---|---|---|
| O1 | NO | NOT FOUND — "a summary" (L4) is a document type, not a format |
| O2 | NO | NOT FOUND — fields are listed inline in L4, no skeleton |
| O3 | NO | NOT FOUND — L4 "Keep it short" is not numeric |
| O4 | YES | L3: "Always be professional" |
| O5 | NO | NOT FOUND — L6 "Don't be too verbose" is not a form-level exclusion |

**Fixes**
- O1 · Where: replace L7 · Problem: no format is named. · Replacement: "Return the summary as a markdown table."
- O2 · Where: insert after the O1 line · Problem: the four fields are named but their shape is not shown. · Replacement: "| Field | Value |\n|---|---|\n| Vendor | |\n| Amount (⟨currency⟩) | |\n| Cost centre | |\n| Business justification | |"
- O3 · Where: replace "Keep it short." in L4 · Problem: no numeric bound. · Replacement: "Business justification: at most 60 words."
- O5 · Where: insert after L7 · Problem: nothing stops preamble or commentary around the table. · Replacement: "Output only the table and, if applicable, the single line 'Requires VP approval'. No introduction or closing remarks."

## 5. Instruction Precision — 1/5
| ID | Verdict | Evidence |
|---|---|---|
| P1 | NO | count=1: L3 "try to give the best answer you can" |
| P2 | NO | NOT FOUND — no number or closed set governs behaviour |
| P3 | NO | NOT FOUND |
| P4 | NO | L6 "Avoid making mistakes about policy details" — not checkable; L6 "Don't be too verbose" — not checkable |
| P5 | YES | none found |

**Fixes**
- P1 · Where: L3 · Problem: "try to" marks the rule as optional, and the rule itself licenses guessing. · Replacement: see C5/R1 — "If the policy does not cover the question, say 'Not covered in the policy'."
- P2 · Where: insert in Rules · Problem: no quantified constraint anywhere. · Replacement: "Business justification: at most 60 words. Amount: a number with currency code, e.g. 12500 ⟨CUR⟩."
- P3 · Where: insert in Rules · Problem: no defaults for unspecified input. · Replacement: "If no currency is given, assume ⟨currency⟩. If no cost centre is given, ask; never infer it."
- P4 · Where: L6 · Problem: two prohibitions cannot be verified from the output. · Replacement: "Quote the policy clause number next to every policy statement. Keep the whole response under 200 words."

## 6. Use of Examples — 0/5
| ID | Verdict | Evidence |
|---|---|---|
| E1 | NO | NOT FOUND |
| E2 | NO | NOT FOUND (E1 failed) |
| E3 | NO | NOT FOUND (E1 failed; O1 and O2 also failed) |
| E4 | NO | NOT FOUND (E1 failed) |
| E5 | NO | NOT FOUND (E1 failed) |

**Fixes**
- E1 · Where: new "## Examples" section · Problem: no example at all; the model has nothing to imitate. · Replacement: "**Example 1** — Input: 'Need to buy 20 laptops from ⟨vendor⟩, about ⟨amount⟩, cost centre ⟨code⟩, replacing end-of-life devices.' Output: ⟨the table from O2, filled⟩"
- E2 · Where: same section · Problem: — · Replacement: label each example "**Example N**" and fence the output.
- E3 · Where: same section · Problem: — · Replacement: build the example output from the O2 skeleton, field for field.
- E4 · Where: same section · Problem: — · Replacement: keep the Input line above each Output.
- E5 · Where: same section · Problem: no edge-case example. · Replacement: "**Example 2 (missing field)** — Input: 'Buy a licence from ⟨vendor⟩, ⟨amount⟩.' Output: 'To raise this request I need: cost centre, business justification.'"

## 7. Robustness & Error Prevention — 1/5
| ID | Verdict | Evidence |
|---|---|---|
| R1 | NO | L3: "If you are not sure, try to give the best answer you can." — licenses guessing; counterproductive |
| R2 | NO | NOT FOUND — L4 "collect the details" states no behaviour for missing fields |
| R3 | NO | L6 "Do not give legal advice." — prohibition with no response behaviour |
| R4 | NO | NOT FOUND |
| R5 | YES | L5: over threshold → "remind the user"; L5: sole-source → "Follow the SOP" |

**Fixes**
- R1 · Where: L3 · Problem: the only uncertainty rule tells the model to guess. · Replacement: "When the policy does not cover a question, say 'Not covered in the policy' and name the closest clause; never answer from general knowledge."
- R2 · Where: insert in Steps · Problem: no behaviour for incomplete requests. · Replacement: "If any of the four fields is missing, ask for all missing fields in one message before producing the table."
- R3 · Where: L6 · Problem: the model is told what not to do but not what to say. · Replacement: "For legal questions, reply: 'I can't advise on legal matters — contact ⟨legal team / mailbox⟩.'"
- R4 · Where: new final step · Problem: no self-check. · Replacement: "Before replying, confirm: four rows, no empty cells, VP line present if amount > ⟨threshold⟩, no text outside the table."

## Top 3 changes
1. E1 — Add one labeled input→output example built from the output table (Examples is the lowest dimension at 0/5).
2. S1 — Rewrite the purchase-request workflow as four numbered, single-action steps (Structure is 1/5; first tied dimension in order).
3. O1 — Name the output format: "Return the summary as a markdown table" (Output is 1/5; next in dimension order).

## Record
```json
{"skill":"Procurement Policy Assistant","rubric":"v1.0","date":"2026-10-28","judge":"⟨model⟩","scores":{"clarity":3,"context":3,"structure":1,"output":1,"precision":1,"examples":0,"robustness":1},"total":10,"band":"Weak"}
```

---

## Notes on the judgment calls in this example

These are the places where a judge might wobble, and why the verdict above is the right one under the rubric:

- **C2 passed although there is no description field.** The rule says fall back to the first paragraph, and L2 names two request types. If the author later supplies a vague Description field ("Your procurement buddy"), C2 flips to NO — the description is what a router sees.
- **X4 and C3 both cite L6.** One line serving two checks is fine. The checks ask different questions (is there a boundary? is there a hard constraint?) and L6 answers both.
- **S3 passed even though L3's action is a bad one.** S3 only asks whether each conditional has *an* action. The quality of that action is R1's job, where it fails.
- **R5 passed on L5.** "Over the threshold" and "sole-source" are two deviations from the default path, each with a prescribed behaviour. That the behaviours point at unsupplied material is X3/X5's concern, already penalised there. Penalising it again under R5 would be double-counting a *defect*; the rubric only allows a line to *satisfy* multiple checks.
- **O5 failed on "Don't be too verbose."** It is a length wish, not a form-level exclusion (nothing about preamble, markdown, or output-only). O3 also fails on it because there is no number.
- **"Coupa" and "VP" were not flagged under C4.** VP is on the allowlist; "Coupa" is a product name, not a 2–6-capital token. C4 is deliberately mechanical.
- **Top 3 ordering.** Examples (0) is lowest, so E1 first. Structure, Output, Precision and Robustness tie at 1; dimension order breaks the tie, so S1 then O1. A judge who "feels" R1 is more important is right about importance and wrong about the protocol — the author can read the full fix list for R1.
