# Setup guide — running the judge in ChatGPT, Microsoft Copilot, or Claude

The judge is a text protocol, so it runs anywhere a model can be given standing instructions. What differs per platform is where the 35 questions live and whether the reference files are reachable. The one rule that does not change: **the questions and word lists must be in the instructions themselves**, not only in an uploaded file, because retrieval from uploaded files is not guaranteed on every turn and a missing question silently changes the score.

## ChatGPT — Custom GPT

1. Create a GPT. Name: `Skill Quality Judge`. Description: `Scores a skill or prompt on 7 dimensions with line-level fixes. Paste your instructions to begin.`
2. **Instructions**: paste the whole of `assets/custom-gpt-instructions.md` (7.3k characters; the field allows about 8k).
3. **Knowledge**: upload `references/rubric.md`, `references/worked-example.md`, `references/calibration-set.md`. The instructions tell the model to consult `rubric.md` on borderline checks.
4. **Capabilities**: turn off Web Browsing and Code Interpreter. The judge should not be able to look anything up; the evaluation is of the text alone, and browsing adds a source of variance.
5. **Conversation starters** (these shape how people submit, which matters for reproducibility):
   - `Evaluate this skill. Name: … Description: … Instructions: …`
   - `Compare two versions of this skill. V1: … V2: …`
   - `Run calibration anchor A` / `Run calibration anchor B`
6. **Calibrate** before sharing: run Anchor A and Anchor B from `calibration-set.md` three times each and check the totals land within ±1 of 10 and 33 (`scoring-protocol.md` §4).

What authors see: the report renders as markdown tables in the chat window. The JSON line at the end is what they copy into the team tracker.

## Microsoft Copilot

**Copilot Studio agent (recommended for a shared team judge).**
1. Create an agent. Name and description as above.
2. **Instructions**: paste `assets/custom-gpt-instructions.md`. The field has a similar size limit; the compact file fits.
3. **Knowledge**: add `rubric.md`, `worked-example.md`, `calibration-set.md` as file knowledge sources (or a SharePoint folder holding them).
4. Disable web search for the agent, for the same reason as above.
5. Publish to Teams/M365 Copilot. Calibrate with both anchors before announcing it.

**Declarative agent / "Create a Copilot agent" in M365** — same instruction block in the Instructions field; attach the three reference files as knowledge. If the instruction limit on your tenant is shorter than 7.3k characters, keep sections PREPARE, CHECKS, WORD LISTS and SCORE and move FIXES and OUTPUT into the knowledge file — those two are the least damaging to retrieve lazily.

**Plain Copilot chat (no agent).** Paste the compact instructions as the first message, then the skill to evaluate in the second. Scores from this route are less reproducible (no fixed system prompt, model may vary per session); use it for a quick look, not for the tracker.

## Claude

Drop the `skill-quality-judge/` folder in as a skill (or save `SKILL.md` via the Save skill button if your org allows it). Claude reads `SKILL.md` when a request matches the description and opens the references on demand. In Claude Code, `/skill-quality-judge` invokes it by name.

## Submitting a skill for evaluation — what to paste

The judge needs the same three things every time, in this form, so that C2 always reads the same field and line numbers are stable:

```
Name: <skill name>
Description: <the description field, verbatim — or "none">
Knowledge files: <file names, comma-separated — or "none">
Instructions:
<the full instructions, unedited>
```

For a SKILL.md, paste the whole file including frontmatter; the judge reads `description:` from it.

Do not pre-format or tidy the instructions before pasting. The line numbers in the report refer to the text as pasted, and an author who tidies first then can't find L14.

## Keeping scores comparable over time

- Record the JSON line from every report in one place (a sheet, a list, a JSON file). The `rubric`, `date` and `judge` fields are what make last quarter's number interpretable.
- Re-run the two anchors monthly and whenever the platform announces a model change. If an anchor moves by more than 1, pause the tracker until the cause is found (`scoring-protocol.md` §4).
- When you edit the rubric — add a check, change a word list, move a threshold — bump the version in SKILL.md, the compact instructions and `rubric.md` together, and start a new tracker column. Scores across rubric versions are not comparable and should not sit on the same trend line.

## Migrating a Custom GPT to a skill — the loop

1. Paste the Custom GPT (name, description, instructions, knowledge file names) into the judge. Record the baseline.
2. Apply the Top 3, re-run. Typical first-pass gains: +6 to +12, almost all from Output Specification and Robustness.
3. Apply the remaining fixes dimension by dimension; re-run after each. Stop at Strong (25+) for an individual's skill, Exemplary (31+) for anything shared across a team.
4. Move the fixed text into a `SKILL.md` using Anchor B in `calibration-set.md` as the structural template (Purpose · Inputs · Steps · Output · Rules · Verify · Edge cases · Examples).
5. The judge does not test execution. Once the text scores Strong, run 2–3 realistic test prompts through the skill and compare against the old Custom GPT — that is a separate check, and Anthropic's skill-creator describes the loop for it.
