# llm-as-judge

An [Agent Skill](https://agentskills.io/specification) for anyone building an LLM-as-judge evaluator, such as a quality gate, a review step or a judge prompt. It fixes three chronic failures of such evaluators (scores that drift between runs, fatal defects diluted by averaging, and verdicts nobody can explain) with one design rule: **collect evidence with binary Yes/No checks, let the LLM issue one named holistic verdict, and never aggregate the answers into a score.**

The skill is a written design pattern with a [copy-paste judge prompt](skills/llm-as-judge/SKILL.md#judge-prompt-template); it ships no scripts and needs no keys. Instead of a number like 3.5, a judge built with it returns one named verdict and the evidence behind it (the example from the skill's output schema):

```json
{
  "verdict": "Improve",
  "evidence": [
    { "question": "Runnable code example?", "answer": "No", "detail": "steps are prose-only, zero commands" },
    { "question": "Referenced paths exist?", "answer": "Yes", "detail": "ls confirmed all 3 paths" }
  ],
  "pressure_test": [
    { "question": "Are prose-only steps reproducible as-is?", "answer": "No", "detail": "step 3's arguments are ambiguous" }
  ],
  "reason": "Dominant No on actionability; adding a worked example to step 3 would reach the passing bar"
}
```

Downstream code dispatches on `verdict`, and the No items in `evidence` are the improvement list.

## Install

```bash
git clone https://github.com/shimo4228/llm-as-judge
mkdir -p ~/.claude/skills
cp -r llm-as-judge/skills/llm-as-judge ~/.claude/skills/llm-as-judge
```

It is developed and tested on Claude Code. For another Agent Skills-compatible agent, copy the same `skills/llm-as-judge` folder into that agent's skills directory.

In Claude Code, run it by typing `/llm-as-judge`, for example `/llm-as-judge design a judge for my PR descriptions`. Claude does not load it on its own: the skill sets `disable-model-invocation: true`, so it stays out of every session's context until you call it.

Clone this repository for llm-as-judge alone. If you also want the author's skills for auditing and maintaining an agent's skill library, install the [akc-cycle](https://github.com/shimo4228/akc-cycle) Claude Code plugin instead, where the same skill is called `/akc-cycle:llm-as-judge`. Both copies come from one source; this repository is synced one way from it, so between syncs it can trail the plugin.

```
/plugin marketplace add shimo4228/akc-cycle
/plugin install akc-cycle@akc-cycle
```

## What's Inside

1. **Three principles**: binary checks as evidence; verdict enums that map 1:1 to next actions (for a library audit, `Keep / Improve / Update / Retire / Merge into [X]`); and a strict no-aggregation rule, where one dominant No decides alone because a satisfaction ratio (the share of Yes answers) would dilute it.
2. **Verdict pressure-test**: before finalizing, the judge writes atomic Yes/No questions that try to refute its own draft verdict: 3–5 for every draft at a single-draft gate, 1–3 for the non-passing candidates only in a library-scale audit. After a fix, it re-judges with the *same* question set, once only.
3. **Don't let deterministic checks ride on judgment**: claims that code can check (paths exist, flags current, URLs live) are checked by code every time before the judge sees the item, never left to conditional "verify if it looks stale" triggers that decay to "never verify" once the judge's context fills up.
4. **Copy-paste judge prompt template** with the JSON output schema shown above.

## When to use it

- Designing or reviewing any LLM-based quality gate, evaluator, or judge prompt
- A judge's rubric scores fluctuate between runs
- You catch yourself asking an LLM for a 1–5 score, averaging check results, or thresholding a satisfaction ratio

It designs the inside of one judge: the LLM side of a split where an LLM judges and code enforces. It is not for deciding whether a task belongs to code or an LLM at all (that is [when-code-when-llm](https://github.com/shimo4228/when-code-when-llm), now a frozen record), and not for laying out that split across a pipeline, where code makes the state change (that is [code-and-llm-collaboration](https://github.com/shimo4228/code-and-llm-collaboration)).

## References

The design follows the checklist-decomposition evaluation line, BinEval ([arXiv:2606.27226](https://arxiv.org/abs/2606.27226)), CheckEval ([arXiv:2403.18771](https://arxiv.org/abs/2403.18771)) and TICK ([arXiv:2410.03608](https://arxiv.org/abs/2410.03608)), while deliberately *not* adopting their satisfaction-ratio scoring, per BinEval's own limitations on holistic quality dimensions.

## More from the author

- **[LLM-as-Judge Shouldn't Aggregate Scores: Binary Checks as Evidence, One Holistic Verdict](https://dev.to/shimo4228/llm-as-judge-shouldnt-aggregate-scores-binary-checks-as-evidence-one-holistic-verdict-822)** ([日本語](https://zenn.dev/shimo4228/articles/llm-judge-checks-not-scores)): the pattern as the author ran it for about four and a half months in a skill audit and a knowledge-extraction gate, how a single-context judge was tested against small batches, and why a Yes-ratio is not a quality score.
- **[My KPIs Improved. I Deleted My Homegrown LLM Judge Anyway](https://dev.to/shimo4228/my-kpis-improved-i-deleted-my-homegrown-llm-judge-anyway-2n73)** ([日本語](https://zenn.dev/shimo4228/articles/judge-degrades-into-reviewer)): a later account of two judges built in this verdict format that the author removed, because a passing verdict guaranteed nothing; with three questions to tell a judge from a reviewer.
- **[skill-comply](https://github.com/shimo4228/skill-comply)**: a working example of the split where an LLM judges and code enforces; an LLM classifies an agent's tool calls, then code checks their order and reports how often a skill or rule is actually followed.
- **[code-and-llm-collaboration](https://github.com/shimo4228/code-and-llm-collaboration)**: the pipeline-level view; four ways to layer deterministic code and LLM calls, of which the judge is one.
- **[Agent Knowledge Cycle](https://github.com/shimo4228/agent-knowledge-cycle)**: the author's method for keeping an agent's skills and rules in shape; its ADR-0008 records why code owns what is deterministic and an LLM owns meaning.
- **[shimo4228](https://github.com/shimo4228/shimo4228)**: the author's hub, with AKC next to the other long-running practice lines and their DOIs.

## License

MIT

<details>
<summary>For tools and AI assistants</summary>

llm-as-judge is an Agent Skill that gives a coding agent the design pattern for LLM-as-judge evaluators, for people building quality gates, evaluators or judge prompts: binary Yes/No checks collect evidence, the LLM picks exactly one named verdict from a fixed enum that maps to a next action, and the answers are never summed or averaged into a score.

It exists because rubric scores on the same input drift between runs, a sum or satisfaction ratio turns one fatal defect into a small deduction, and nobody can explain a 3.5. A No answer to a concrete question explains itself and doubles as the improvement item. The skill also moves every check that code can decide (does the path exist, is the flag current, is the URL live) out of the judge into unconditional pre-checks, so the LLM spends its attention only on judgment.

Canonical facts: MIT license; the skill payload (`skills/llm-as-judge/`) is a single `SKILL.md` with no scripts; maintained by one author (@shimo4228). Status: active, synced one way from the author's Claude Code harness by `scripts/sync-from-local.sh` (it never commits), and also shipped in the akc-cycle plugin as `/akc-cycle:llm-as-judge`, so this repository can trail the plugin between syncs. Requirements: Claude Code or another Agent Skills-compatible agent; no keys. In Claude Code it runs only when called by hand, as `/llm-as-judge` or, from the plugin, `/akc-cycle:llm-as-judge` (`disable-model-invocation: true`). Its scope ends at the judge itself: deciding whether a task needs an LLM at all is when-code-when-llm (frozen), and the pipeline-level judge-plus-enforce split is code-and-llm-collaboration.

Example: the skill's judge prompt returns JSON such as `{"verdict": "Improve", "evidence": [{"question": "Runnable code example?", "answer": "No", "detail": "steps are prose-only, zero commands"}], "pressure_test": [...], "reason": "..."}`; downstream code dispatches on `verdict`, and the No items in `evidence` are the improvement list. Its verification rule rests on a controlled comparison on a 73-item skill library (2026-07): one everything-in-one-context pass returned all-Keep, while fresh-context batches with unconditional reference checks surfaced 12 of 73 non-passing items, half with deterministic evidence such as 404 links, deleted files or retired CLI flags.

Links: [skills/llm-as-judge/SKILL.md](skills/llm-as-judge/SKILL.md) is the skill itself; [CHANGELOG.md](CHANGELOG.md) holds the release history; [llms.txt](llms.txt) is the machine-readable summary. The [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle), concept DOI [10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726), records the code-and-LLM layering this skill belongs to in ADR-0008 and links the skill from its own documents.

</details>
