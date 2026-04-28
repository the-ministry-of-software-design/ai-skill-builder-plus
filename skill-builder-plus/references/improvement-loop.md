# Improvement Loop

Reference for Phase 5 - Iterating on feedback.

---

## How to Think About Improvements

### Generalise, don't overfit

You are iterating on 2-3 examples, but the skill will be used on thousands of different prompts.
An improvement that fixes a test case by hardcoding assumptions about it is harmful. Ask:

- Why did the model fail here? What underlying misunderstanding does this reveal?
- What rule or explanation would help the model handle *this kind of situation* in general?

If the feedback is "the chart is missing axis labels", the generalised fix is not
"always add axis labels to charts" - it is explaining *why* axis labels matter
(readability, interpretability) so the model applies the principle to charts, tables, and any
other visual where context is implicit.

### Keep the prompt lean

Read the transcripts, not just the outputs. If the model is spending time on steps that produce
nothing useful, the skill may be instructing it to do unnecessary work. Remove those instructions.
Less is often more: a lean, clear skill outperforms a detailed, cluttered one.

### Explain the why

If you find yourself writing ALWAYS or NEVER in all caps, or using rigid numbered lists for things
that require judgement, reframe. Give the model the reasoning. Smart models given good reasoning
outperform models given prescriptive rules - and they generalise better to new situations.

### Look for repeated work across test cases

Read transcripts from all test runs. If two or three subagents independently wrote the same helper
script (e.g., `extract_table.py`, `convert_to_docx.py`), that is a strong signal the skill should
bundle that script. Write it once, put it in `scripts/`, and instruct the skill to use it.
Every future invocation benefits without reinventing the wheel.

### Consider a learnings.md

For skills that will be used repeatedly over time, add a `learnings.md` file to the skill:

```
skill-name/
├── SKILL.md
├── learnings.md    ← add this
└── references/
```

In SKILL.md, add: "Read `learnings.md` before starting. These are observations from past runs
that refine the default behaviour."

After each run or batch of runs, the skill (or a wrap-up skill) appends structured observations:
```markdown
## 2026-04-28
- When the input has no column headers, default to row 1 as headers rather than asking
- Users consistently prefer the summary before the detail, not after
```

**Prune regularly.** Strip notes that are no longer relevant. A growing `learnings.md` recreates
the bloat problem you're trying to avoid.

---

## The Iteration Loop

1. Read `feedback.json` and `benchmark.json`
2. Read the transcripts for any eval where the user had complaints
3. Identify the root cause (not just the symptom)
4. Write a draft revision to the skill
5. Re-read the draft with fresh eyes: is it leaner? Does it explain the why? Is it still under
   200 lines?
6. Apply the revision
7. Snapshot the current skill: `cp -r <skill-path> <workspace>/skill-snapshot-v<N>/`
8. Rerun all test cases into `iteration-<N+1>/` with the new skill vs. the previous snapshot
9. Generate the viewer with `--previous-workspace` pointing at the previous iteration
10. Wait for the user to review and share feedback
11. Repeat from step 1

### When to stop

- The user says they are satisfied
- All feedback is empty (everything looks good)
- Benchmark improvement has plateaued across two iterations
- You are making changes but pass rates are not moving - this usually means the remaining failures
  require more example diversity, not more instruction detail

When iteration is complete, update the skill's `evals/` folder with the final state:

- Replace `evals/baseline-benchmark.json` with the best iteration's `benchmark.json`
- Update `evals/evals.json` if assertions were refined during iteration
- Update `evals/README.md` to reflect final pass rates and any new key assertions

This ensures the next person to work on the skill starts with an accurate baseline.

---

## Blind Comparison (optional, for rigorous version comparisons)

When you want an independent verdict on whether a new version is actually better, use the blind
comparison system. Read `references/agents/comparator.md` and `references/agents/analyzer.md` for the full protocol.

The basic idea: give two outputs to an independent subagent without telling it which version
produced which, let it judge quality, then analyse why the winner won.

This is optional. The human review loop is usually sufficient. Use blind comparison when:
- The benchmark scores are close and the user wants confidence before shipping
- You want to isolate which specific change in the skill drove an improvement
