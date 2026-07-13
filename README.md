# llm-as-judge

An [Agent Skill](https://agentskills.io/specification) that fixes the three chronic failures of LLM-as-judge evaluators — scores that drift between runs, fatal defects diluted by averaging, and verdicts nobody can explain — with one design rule: **collect evidence with binary Yes/No checks, let the LLM issue one named holistic verdict, and never aggregate the answers into a score.**

## Install

### Claude Code

```bash
# Copy into your global skills directory
cp -r skills/llm-as-judge ~/.claude/skills/llm-as-judge
```

### SkillsMP

```bash
/skills add shimo4228/llm-as-judge
```

## What's Inside

1. **Three principles** — binary checks as evidence, verdict enums that map 1:1 to next actions (`Keep / Improve / Retire / Merge into [X]`), and a strict no-aggregation rule (one dominant No decides alone; a satisfaction ratio dilutes it)
2. **Verdict pressure-test** — before finalizing a failing verdict, the judge generates 1–3 atomic refutation questions against its own draft; after a fix, re-judge with the *same* question set, once only
3. **Verification ownership** — deterministically checkable claims (paths exist, flags current, URLs live) get code-owned unconditional pre-checks, never conditional "verify if it looks stale" triggers that decay in a loaded context. Grounded in a controlled comparison on a 73-item library where a single-context pass returned all-Keep while small fresh-context batches surfaced 12 non-passing items
4. **Copy-paste judge prompt template** with a JSON output schema downstream code can dispatch on

## When It Triggers

- Designing or reviewing any LLM-based quality gate, evaluator, or judge prompt
- A judge's rubric scores fluctuate between runs
- You catch yourself asking an LLM for a 1–5 score, averaging check results, or thresholding a satisfaction ratio

Not for deciding whether a task belongs to code or an LLM at all (that is [when-code-when-llm](https://github.com/shimo4228/when-code-when-llm)), and not for the architecture-level judge+enforce state-mutation split (that is [code-and-llm-collaboration](https://github.com/shimo4228/code-and-llm-collaboration)).

## References

The design follows the checklist-decomposition evaluation line — BinEval ([arXiv:2606.27226](https://arxiv.org/abs/2606.27226)), CheckEval ([arXiv:2403.18771](https://arxiv.org/abs/2403.18771)), TICK ([arXiv:2410.03608](https://arxiv.org/abs/2410.03608)) — while deliberately *not* adopting their satisfaction-ratio scoring, per BinEval's own limitations on holistic quality dimensions.

## License

MIT
