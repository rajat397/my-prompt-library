# Evaluation Plan: `question-sharpener` skill

You are the **executor agent**. Run this plan end to end, save everything in the structure below, and return the results folder to the user. Do not edit the skill. Do not skip steps. If something is impossible in your environment, record it in `results/DEVIATIONS.md` and continue.

## 0. What is being tested

The skill makes an assistant (a) decide whether a request needs sharpening, (b) ask at most 3 targeted cross-questions in one round when it does, (c) restate the sharpened question, (d) answer with assumptions and "what would change my answer", (e) stay out of the way on trivial, already-specific, or "just answer" requests.

Hypotheses:
- **H1** On substantive requests, answers with the skill are better than without, judged blind.
- **H2** The skill is not just "asking questions": it beats a naive control that always asks 3 clarifying questions.
- **H3** The skill does not add friction where it is not needed: zero questions on `direct` cases.
- **H4** It obeys its own limits: at most 3 questions, one round, no re-asking known context.
- **H5** Overhead (tokens, turns) is modest relative to quality gain.

## 1. Inputs you receive

- `question-sharpener.skill` (zip; unzip to get the skill folder with `SKILL.md` and `references/`)
- `eval-cases.json` (18 cases + 12 trigger queries)
- This file

## 2. Conditions

| Id | Condition | How |
|---|---|---|
| A | Baseline | Fresh session, no skill available. Prompt only. |
| B | Skill | Fresh session with the skill installed/available. Prompt only. Do not mention the skill by name. |
| C | Naive control (run only on cases c01, c04, c07, c11, c18) | Fresh session, no skill. Append to the prompt: "Before answering, ask me 3 clarifying questions." |

Run each case in each applicable condition **2 times** (reps) if budget allows, otherwise 1. Every run is a **fresh session** with no shared memory or prior conversation. Record model name/version used for every run; use the same model across conditions.

## 3. Simulated user protocol

The model under test may ask cross-questions. You play the user, using only the case's `hidden_context`:

1. Send `prompt` exactly as written. Never add the hidden context yourself.
2. If the model asks questions, answer each one briefly (one sentence), truthfully from `hidden_context`. If a question is not covered by it, reply "not sure, use your judgment". If it offers options or defaults and the hidden context does not decide it, pick the default.
3. If the model gives a first answer without asking anything, accept it. Do not prompt it to ask. Do not follow up.
4. Allow one round of questions only. If the model asks a second round, answer it but log `multi_round = true`.
5. For `direct` cases, if the model asks any question, answer minimally ("go ahead with your best judgment") and log it as a violation.
6. Stop after the model delivers its final substantive answer.

Save the full transcript of every run as `results/transcripts/<case_id>__<condition>__rep<n>.md` with turns labelled USER / MODEL.

## 4. Automatic measures (you compute, per run)

Record in `results/runs.csv` one row per run with columns:

`case_id, condition, rep, model, expected_lane, n_questions_asked, n_question_rounds, asked_on_direct_case (bool), questions_over_limit (bool, >3), restated_question (bool), states_assumptions (bool), states_what_would_change_answer (bool), names_failure_mode_or_counterargument (bool), gives_improved_prompt_block (bool, c15 only), followed_requested_format (bool), total_tokens, total_turns, final_answer_word_count`

Definitions:
- `n_questions_asked`: count of distinct questions put to the user before the final answer.
- `restated_question`: the answer opens with a 2-4 line restatement of situation / output / success criterion.
- `states_assumptions`: explicit list of assumptions the model made that the user did not confirm.
- `names_failure_mode_or_counterargument`: contains the strongest case against its own recommendation or a named way a smart person would get this wrong.

## 5. Quality judging (blind)

Use a **separate judge agent** (a different session from any executor run; if possible a different model family or at least a fresh context). The judge never sees the condition label.

### 5a. Rubric scoring (per run, per `sharpen` case)
Give the judge: the case `prompt`, the case `hidden_context` and `good_answer_signals`, and the **final answer only** (remove question/answer turns and any restatement of the improved prompt does stay, since it is part of the answer). Score 1-5 on each:

1. **Fit to the real situation**: uses the specifics in `hidden_context` correctly (1 = generic, 5 = clearly tailored)
2. **Non-obviousness**: for ideation/strategy only; otherwise score N/A (1 = standard advice, 5 = surprising and defensible)
3. **Checkability**: assumptions, evidence, and reasoning are visible and could be falsified
4. **Actionability**: reader could act this week; concrete next step or test
5. **Honesty about uncertainty**: states what would change the answer; does not simply agree with leading framing

Judge outputs one JSON line per run: `{"run_id":..., "fit":n, "non_obvious":n|null, "checkable":n, "actionable":n, "honesty":n, "one_line_reason":"..."}`.

### 5b. Blind pairwise comparison (per case, per rep)
For each `sharpen` case present the judge **A vs B final answers in randomised order** (record the mapping in `results/pairwise_key.json`, hidden from the judge). Ask: "Which answer would better serve this person given their real situation? Answer A, B, or Tie, with 2 sentences of reasoning." Run pairs for A-vs-B for all `sharpen` cases, and C-vs-B for the five control cases.
Also run each pair a **second time with order swapped** to catch position bias. If the two verdicts disagree, record as Tie.

### 5c. Question quality (condition B and C, per question asked)
Judge scores each cross-question 0/1 on: `material` (would change the answer), `not_redundant` (not already given or inferable from the prompt), `quick_to_answer` (answerable in seconds, ideally with options/default). Also flag `challenge_question` (true if it challenges a leading frame or hidden assumption). Report per-run mean.

## 6. Behaviour checks (you score, per run, from transcripts)

Pass/fail, and write one line of evidence for each fail:

| Check | Applies to | Pass condition |
|---|---|---|
| B1 No friction | `direct` cases, B | zero questions asked |
| B2 Respects override | c14, B | zero questions; states assumptions |
| B3 Question cap | `sharpen` cases, B | ≤3 questions and a single round |
| B4 No re-asking known info | c16 and any case where the prompt already gives context, B | no question about something the prompt stated |
| B5 Challenge on leading frame | c03, c17, c18, B | at least one question or explicit statement that tests the opposite or the hidden assumption |
| B6 Answers what it restated | `sharpen` cases, B | final answer matches the restated question (no drift) |
| B7 Prompt-mode deliverable | c15, B | paste-ready prompt in a code block with situation, task, output format, success criteria, constraints, plus a list of open gaps |
| B8 Closing line | `sharpen` cases, B | one concrete line on what was missing from the original question (not a lecture) |
| B9 Profile offer | all B runs | offered at most once; not offered on trivial cases |
| B10 Missing-input handling | c08, B | notices that no code/approach was pasted and asks or states the assumption rather than pretending to review |

## 7. Trigger test (optional, only if the `claude` CLI is available)

Use the 12 `trigger_queries` in `eval-cases.json`. For each, run 3 times in a fresh session with the skill installed and record whether the skill was invoked (the skill-creator's `scripts/run_eval.py` does this). Report trigger rate per query, plus precision and recall. If the CLI is unavailable, skip and note it in `DEVIATIONS.md`.

## 8. Human spot check

Pick 5 pairwise comparisons: the 2 where the judge preferred A (baseline), the 2 closest ties, and 1 where B won strongly. Put them into `results/human_review.md` in randomised order with A/B labels only, and leave a blank line for the user's verdict after each. The user will fill these in.

## 9. Output structure to return

```
results/
  runs.csv
  behavior_checks.csv        # case_id, check_id, pass, evidence
  judge_rubric.jsonl
  judge_pairwise.csv         # case_id, rep, pair (A_vs_B | C_vs_B), verdict_1, verdict_2, final, reason
  pairwise_key.json
  question_quality.csv
  trigger_results.csv        # if run
  transcripts/
  human_review.md
  DEVIATIONS.md
  SUMMARY.md
```

`SUMMARY.md` must contain, in this order:
1. Setup (model(s), reps, conditions run, anything skipped)
2. Headline numbers
   - Pairwise win rate of B over A (ties counted as half), with n
   - Pairwise win rate of B over C, with n
   - Mean rubric score per dimension, A vs B (and C where available)
   - Mean questions asked by B on `sharpen` cases; % of `direct` cases with zero questions; % runs over the question cap
   - Behaviour check pass rates B1-B10
   - Median total tokens and turns, A vs B (overhead ratio)
3. Results by category (ideation, decision, technical, research, writing, trivial, override, prompt-for-agent, specific, contested, leading)
4. Where B lost or was worse than A: list each, with transcript path and a 2-sentence description
5. Question quality findings
6. Surprises and anything that looked like judge bias or noise (include position-bias disagreement rate)
7. Raw observations about friction: places where a real user would likely have been annoyed
8. Do **not** recommend edits to the skill. Observations only; the user will interpret.

## 10. Pass thresholds (for reference in your summary; report pass/fail per threshold)

| Metric | Target |
|---|---|
| B beats A in blind pairwise, `sharpen` cases | ≥ 65% (ties as half) |
| B beats C in blind pairwise | ≥ 55% |
| Zero questions on `direct` cases (B1, B2) | ≥ 90% of runs |
| Question cap and single round (B3) | 100% |
| Challenge on leading frame (B5) | ≥ 80% |
| Median token overhead of B vs A | ≤ 1.8x |
| No `direct` case where B's answer is worse than A's | 100% |

## 11. Rules of integrity

- Do not tune prompts, re-run only the cases you like, or discard failures. If a run errors, rerun once and log both.
- Keep judge and executor contexts separate.
- Never tell the model under test that it is being evaluated.
- Quote transcripts verbatim in the failure section.
