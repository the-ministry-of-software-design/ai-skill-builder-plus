# Evals

Baseline evals for skill-builder-plus. Use these to measure whether future changes improve or
regress the skill's behaviour.

## Files

**evals.json** - The four eval prompts and their assertions. Feed this to the eval runner.

**baseline-benchmark.json** - Results from the first full eval run (iteration-2, 2026-04-28).
Compared skill-builder-plus against skill-creator across all four evals.
- skill-builder-plus: 100% pass rate
- skill-creator (baseline): 65% pass rate
- delta: +0.35

## Eval summaries

**Eval 1 - New skill from scratch (LinkedIn posts)**
Tests Phases 1-3: interviews for context, produces SKILL.md under 200 lines, 3-part description,
references under 300 lines, test prompts presented before running evals.

**Eval 2 - Refactor a bloated skill (meeting notes, 235 lines)**
Tests the refactor workflow: diagnoses progressive disclosure violation, refactors SKILL.md to under
200 lines, creates reference files each under 300 lines, performs cold-start test.

**Eval 3 - Quick skill, user skips evals (commit messages)**
Tests flexibility: respects skip-evals request, still produces SKILL.md under 200 lines with 3-part
description and conventional commits examples.

**Eval 4 - Eval workspace structure (Jira status reports)**
Tests the eval workflow rules fixed in this version: eval dirs named with eval- prefix,
grading.json at run-1/grading.json, timing.json at run-1/timing.json,
subagent save paths pointing to run-1/outputs/.

## Running evals

Workspace: create `skill-builder-plus-workspace/iteration-N/` as a sibling to the skill folder.

Run new_skill vs old_skill (or new_skill vs previous iteration):

```bash
# SKILL_DIR = installed path of skill-builder-plus (from available_skills in your context)

# After grading all runs:
cd $SKILL_DIR
python -m scripts.aggregate_benchmark <workspace>/iteration-N --skill-name skill-builder-plus

# Generate viewer:
python scripts/generate_review.py \
  <workspace>/iteration-N \
  --skill-name "skill-builder-plus" \
  --benchmark <workspace>/iteration-N/benchmark.json \
  --static <workspace>/iteration-N/review.html \
  --previous-workspace <workspace>/iteration-<N-1>
```

## Key discriminating assertions

These are the assertions that most reliably distinguish skill-builder-plus from skill-creator.
If a future change breaks any of these, investigate before shipping:

- Description contains trigger + negative trigger + outcome (evals 1, 3)
- Cold-start test performed after refactoring (eval 2)
- grading.json placed at run-1/grading.json not outputs/ (eval 4)
- Subagent save paths point to run-1/outputs/ (eval 4)
