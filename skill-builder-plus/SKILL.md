---
name: skill-builder-plus
license: MPL-2.0
compatibility: Requires Python 3 (used by scripts/ for evals, packaging, and description optimisation)
metadata:
  author: The Ministry of Software Design
  version: 1.0.0
description: |
  Creates, tests, and iteratively improves Claude skills using rigorous eval-driven methodology.

  TRIGGER: Use whenever the user wants to create a new skill from scratch, improve or refactor an
  existing skill, run evals against a skill, measure or benchmark skill quality, optimise a skill
  description for better triggering, or package a skill for distribution. Also trigger when someone
  says "turn this workflow into a skill", "my skill is too slow", "skill isn't triggering reliably",
  "refactor my skill", or "help me test my skill". Trigger even with a vague idea - this skill
  handles the full journey from idea to packaged output.

  DO NOT TRIGGER: for general writing, code reviews, or questions about how skills work in theory.
  Only when the user is actively building, testing, or fixing a specific skill.

  OUTCOME: a SKILL.md file with correct progressive disclosure structure, supporting reference files,
  test evals with benchmark results, and optionally a packaged .skill file ready to install.
---

# Skill Builder Plus

An enhanced skill creator with strict quality rules derived from real-world skill engineering experience.

## Non-Negotiable Rules

These rules are grounded in how context windows work. Violating them costs every future invocation
of the skill you're building.

1. **SKILL.md must be under 200 lines. Reference files should be 200-300 lines each.** The 200-line
   limit on SKILL.md is the threshold at which an LLM can efficiently scan an entry point to decide
   what to load next. Reference files can be longer (up to 300 lines) because they are loaded only
   when needed and cover a single focused topic. Split any reference file that exceeds 300 lines.
2. **Description must have three parts: trigger, negative trigger, outcome.** A description missing
   any of these will mis-fire. Marketplace skills activate only ~20% of the time because of poor
   descriptions.
3. **Skills are workflow capabilities, not documentation dumps.** Ask: what can Claude *do* with this
   skill? Not: what does it *know*? Organise by workflow capability, not by tool.
4. **References are first-class citizens.** SKILL.md is the map. `references/` is where the work
   lives. Treat every detail not needed for routing as a candidate for a reference file.
5. **Cold-start test.** After writing a skill, count how many lines load on first activation
   (SKILL.md body only, not reference files). If it exceeds 500 lines, refactor before proceeding.
6. **3-5 assertions per eval run.** More than 5 dilutes review quality and hides which assertions
   actually discriminate good from bad output.
7. **Explain the why.** Avoid bare imperatives. Give the reason so the model can generalise rather
   than follow rules blindly.

## Process at a Glance

Understand intent → Research → Draft skill → Validate structure → Write evals → Run & review →
Iterate → Optimise description → Package

Read the relevant reference file when you reach each phase:

| Phase | Reference |
|-------|-----------|
| Drafting the skill | `references/writing-guide.md` |
| Running and grading evals | `references/eval-workflow.md` |
| Iterating on feedback | `references/improvement-loop.md` |
| Optimising the description | `references/description-optimization.md` |
| JSON schemas | `references/schemas.md` |
| Subagent instructions | `references/agents/` directory |

## Phase 1 - Understand Intent

Extract from context or ask:

- What should this skill enable Claude to *do*? (workflow, not documentation)
- When should it trigger? When should it *not*?
- Is success objectively verifiable, or does it require human judgement?

Skills with verifiable outputs (file transforms, data extraction, code generation) benefit from
quantitative evals. Skills with subjective outputs (writing style, creative work) rely on human
review. Decide now and set expectations accordingly.

## Phase 2 - Research and Context

If the user has personalisation files (brand guidelines, audience personas, tone examples), ask for
them. These go in `references/` and are referenced from SKILL.md - never embedded in it.

Check whether a shared context folder already exists that other skills reference. If so, point to it
rather than duplicating content. Duplication creates drift.

## Phase 3 - Draft the Skill

Read `references/writing-guide.md` before writing.

Before drafting, ask the user:
- What **licence** should the skill use? Suggest https://choosealicense.com/ for open source options.
  Add as `license: MIT` (or chosen licence) in the frontmatter.
- Do they want to add an **author**? If so, what should it be? Add under `metadata: author: <value>`.
- If the skill bundles scripts or requires specific tools (e.g. Python 3, Node, a CLI), note this in
  `compatibility:`. If you already know the requirements from context, fill it in without asking.

Gates before proceeding to evals:
- [ ] SKILL.md is under 200 lines
- [ ] Every reference file is under 300 lines; split any that exceed this
- [ ] Description has trigger + negative trigger + outcome
- [ ] Every detail not needed for routing is in `references/`
- [ ] Cold-start line count is under 500 (SKILL.md body only)
- [ ] `license` field set in frontmatter (or explicitly noted as proprietary)
- [ ] `compatibility` field set if skill has runtime requirements (Python, Node, CLI tools, etc.)
- [ ] `metadata.author` set if user provided one

## Phase 4 - Evals

Read `references/eval-workflow.md` before starting. The workspace layout and file locations
must match exactly what the aggregate script expects - wrong directory names or file paths produce
a silently empty benchmark. The reference file documents the three most common mistakes.

Write 2-3 realistic test prompts. Use 3-5 assertions per eval. Spawn with-skill and without-skill
runs in the same turn. Tell every subagent the exact save path for outputs - if not told explicitly,
subagents will invent their own structure.

After all runs complete, write grading.json and timing.json, then run the aggregate script, then
generate the viewer. **Always generate the eval viewer for human review before revising the skill
yourself.** Use `--static <path>` in Cowork (no browser available).

## Phase 5 - Iterate

Read `references/improvement-loop.md`.

Generalise from feedback - never overfit to test cases. Read transcripts, not just outputs. Look for
work repeated across test cases that should be bundled as a script.

Consider adding a `learnings.md` to the skill for self-improvement over time (see improvement-loop).

## Phase 6 - Optimise and Package

After the user is satisfied, offer to run description optimisation
(see `references/description-optimization.md`), then:

1. Set `version: 1.0.0` under `metadata:` in the skill's frontmatter (if not already set).
2. Run the packager from `SKILL_DIR` (the path where skill-builder-plus is installed -
   shown in the `available_skills` list for skill-builder-plus in your context):

```bash
cd $SKILL_DIR
python -m scripts.package_skill <path/to/skill-folder>
```

The skill frontmatter should look like this before packaging:

```yaml
---
name: my-skill
description: ...
license: MIT
compatibility: Requires Python 3   # omit if no special requirements
metadata:
  author: The Ministry of Software Design
  version: 1.0.0
---
```

---

## Bundled Resources

Scripts in `scripts/` run without loading into context. Set `SKILL_DIR` to this skill's installed
path (from your `available_skills` context) before running them.

Agents in `references/agents/` are instruction files for subagents spawned during evals:
- `references/agents/grader.md` — evaluates assertions against outputs
- `references/agents/comparator.md` — blind A/B comparison between two outputs
- `references/agents/analyzer.md` — analyses why one version beat another

Reference files (loaded on demand): `references/writing-guide.md`, `references/eval-workflow.md`,
`references/improvement-loop.md`, `references/description-optimization.md`, `references/schemas.md`
