# Scoring protocol — how scores stay reproducible

The rubric makes each verdict binary and evidence-backed. This file covers everything *around* the verdicts: how the judge is configured, how to detect drift, how to handle awkward inputs, and how to compare versions. Follow it and the same skill scored by two people on two days lands on the same number.

## 1. Where variance comes from, and what each rule removes

| Source of variance | Rule that removes it |
|---|---|
| Holistic impression ("feels like a 7") | No dimension is rated directly; its score is the count of YES answers |
| Reading intent into the text | Judge the text as written; unsure → NO |
| Partial credit | None. A check is YES or NO |
| Different judges counting different words | Word lists in `rubric.md` §8; match whole words, case-insensitive |
| Vacuous cases decided both ways | Defect checks pass when nothing offending exists; presence checks fail when nothing is found — always |
| Evidence invented after the verdict | Evidence first (quote + line), then verdict; a YES with no quote flips to NO on re-read |
| Fix list reflecting taste | Fixes are generated only from NO verdicts; one fix per NO |
| Top-3 reflecting taste | Lowest dimension first, tie → dimension order, then lowest check ID |
| Model sampling | Temperature 0 (or lowest available); same model version recorded in the header |
| Rubric changes between runs | Rubric version in every report; scores across versions are not comparable |

## 2. Judge configuration

- **Model**: pick one and record it in the header. Scores from different models are comparable in *direction* (both should flag the same missing output spec) but not in exact count, because models differ in how aggressively they match the word lists. Re-baseline with the calibration set when the model changes.
- **Sampling**: temperature 0 / deterministic mode where the platform exposes it. Custom GPTs and Copilot agents do not expose temperature; there, run the calibration set three times when first deploying to measure the spread (§4).
- **Context**: web browsing and code execution off. The judge should not be able to look things up; the evaluation is of the text, not of the world.
- **Knowledge files**: `rubric.md` is uploaded alongside the instructions so borderline checks resolve the same way every time. The compact instruction block in `assets/custom-gpt-instructions.md` already contains all 35 questions and the word lists, because retrieval from knowledge files is not guaranteed on every turn and the questions must be in context every time.

## 3. Input handling

**Single prompt / instruction set.** Number non-empty lines and run.

**Skill folder (SKILL.md + references/ + scripts/).** The primary file is SKILL.md. References count as *supplied material*: a reference named in SKILL.md ("see references/policy.md") satisfies X3 and does not count as unsupplied for X5. The judge does not score the references themselves unless asked; the question is whether SKILL.md points at them properly.

**Custom GPT export.** Ask the author to paste three things: the Name, the Description field, and the Instructions. C2 reads the Description field. If the author lists uploaded knowledge file names, those names are supplied material.

**Copilot agent.** Same as Custom GPT: name, description, instructions, and the names of any knowledge sources.

**Long prompts (over ~400 lines).** Still run the full protocol — reproducibility depends on not sampling. If the platform's context limit forces truncation, evaluate the first 400 lines, state that in the header, and mark every check that depends on later content as NO with evidence `beyond evaluated range`.

**Prompt in another language.** Evaluate in that language; the word lists are English, so note in the header `word-list checks (C5, P1) applied by translation` and treat those two checks as lower-confidence.

**No text pasted.** Ask for it. Never score a skill from its name, a screenshot description, or memory of a previous version.

## 4. Drift monitoring with the calibration set

`calibration-set.md` holds two anchor prompts with expected per-check verdicts: a weak anchor (expected 10/35) and a strong anchor (expected 33/35). Use them like a scale's reference weights.

**At deployment.** Run both anchors three times each. Record every total. The deployment is calibrated when every run of each anchor is within ±1 of its expected total and no dimension differs by more than 1 from the expected score.

**Monthly, or whenever the platform's underlying model changes.** Run both anchors once. If either total moves by more than 1, or any dimension moves by more than 1, do not trust that month's real scores until the cause is found. The usual causes, in order: (1) the compact instructions were edited and a question drifted; (2) the knowledge file was replaced; (3) the platform changed model — re-baseline and note the new model in the tracker.

**Checks that drift most.** C5 and P1 (word-list counts) and P5 (contradictions) are the least mechanical. If drift appears, these are where to look first. The word lists are fixed precisely so that drift there indicates a configuration problem, not a judgment call.

## 5. Bands and what they mean

| Total | Band | Meaning for the author |
|---|---|---|
| 0–14 | Weak | The prompt describes a wish, not a procedure. Expect large run-to-run variation. Not ready to publish as a skill. |
| 15–24 | Developing | The task is clear but the output and failure handling are not pinned down. Usable for an individual; not for a team. |
| 25–30 | Strong | Publishable. Remaining gaps are usually examples (dimension 6) and edge cases (R5). |
| 31–35 | Exemplary | Reference-quality. Use it as a template for other skills. |

Bands are on the total only. Do not band individual dimensions; a 1/5 in Examples on a strong skill is a to-do, not a verdict.

## 6. Optional weighting profiles

Equal weights are the default because they keep every skill on one scale. If a team wants a single number that reflects the kind of skill, apply one of these profiles *after* computing the unweighted scores, and report both:

| Profile | Use for | Weights (dims 1–7) |
|---|---|---|
| `default` | everything | 1,1,1,1,1,1,1 |
| `structured` | skills that emit JSON, tables, forms, or transform input→output | 1,1,1,1.5,1,1.5,1 |
| `conversational` | Q&A / advisory skills with free-text answers | 1,1.5,1,0.5,1,0.5,1.5 |

Weighted total = Σ(score × weight) / Σ(5 × weight) × 35, rounded to one decimal. Always print the unweighted total too; trackers should store the unweighted one.

## 7. Comparing two versions

1. Score each version with the full protocol — two complete reports, same judge, same session.
2. Build the delta table:

```
| Dimension | v1 | v2 | Δ | Flipped checks |
|---|---|---|---|---|
| 1 Clarity & Specificity | 3 | 5 | +2 | C4 N→Y, C5 N→Y |
| … | | | | |
| Total | 10 | 24 | +14 | |
```

3. List regressions (YES→NO) first, with both quotes, because an edit that fixed one check and broke another is the thing the author most needs to see.
4. Only check-level flips count as evidence of change. A dimension whose score moved with no flipped check is a judging inconsistency — re-run that dimension on both versions.

## 8. Recording results

Every report ends with the JSON record:

```json
{"skill":"procurement-policy-assistant","rubric":"v1.0","date":"2026-10-28","judge":"gpt-4o","scores":{"clarity":3,"context":3,"structure":1,"output":1,"precision":1,"examples":0,"robustness":1},"total":10,"band":"Weak"}
```

Paste one line per evaluation into a tracker (a sheet, a list, a JSON file). With `rubric`, `date` and `judge` recorded, a score from last quarter can be read correctly next quarter. When the rubric version changes, start a new column; do not mix versions on one trend line.

## 9. What this protocol deliberately does not do

- It does not run the skill. Execution quality is a different evaluation with different tooling (test prompts, graders, baselines — see Anthropic's skill-creator for that loop).
- It does not predict whether a router will pick the skill. C2 checks that the description names triggers, which is necessary but not sufficient; triggering accuracy needs a should-trigger / should-not-trigger query set.
- It does not reward length. A 20-line skill can score 33; a 300-line one can score 12.
- It does not judge domain correctness. A policy threshold can be wrong and still satisfy X3; catching that is the author's job, and the judge should not pretend otherwise.
