# Skill Builder Plus — Claude Skill

A skill for [Claude Code](https://claude.ai/claude-code) and [Cowork](https://claude.ai) that creates, tests, and iteratively improves Claude skills using a rigorous eval-driven methodology.

Licensed under the [Mozilla Public License 2.0](LICENSE).

---

## What it does

Give Claude a skill idea — or an existing skill that isn't working well — and Skill Builder Plus takes it through a structured workflow to a production-ready, packaged output:

**Understand intent** — extracts what the skill should *do* (workflow capability, not documentation), when it should trigger, and whether success is objectively verifiable or requires human review.

**Draft the skill** — writes a `SKILL.md` under 200 lines with a proper three-part description (trigger, negative trigger, outcome), and puts supporting detail into `references/` files loaded only when needed.

**Run evals** — spawns paired with-skill and without-skill runs against realistic test prompts, grades assertions, produces a benchmark report, and opens an HTML viewer for human review.

**Iterate** — reads transcripts (not just outputs) to generalise improvements without overfitting to test cases, and optionally records learnings for self-improvement over time.

**Optimise and package** — runs description optimisation to improve trigger accuracy, sets versioning in frontmatter, and packages the skill into a `.skill` file ready to install or distribute.

---

## Installation

### Prerequisites

Python 3 is required for eval scripts, packaging, and description optimisation.

### Claude Code

1. Download `skill-builder-plus.skill` from the [latest release](../../releases/latest).
2. Install it from your terminal:

```bash
claude plugin install skill-builder-plus.skill
```

### Cowork

1. Download `skill-builder-plus.skill` from the [latest release](../../releases/latest).
2. In the Cowork sidebar, open **Settings → Skills → Install from file**.
3. Select the `.skill` file and confirm.

---

## Usage

Ask Claude to build or improve a skill. You can be as brief or as specific as you like:

```
Create a skill that summarises Slack threads into action items.
```
```
My skill isn't triggering reliably — can you fix it?
```
```
Turn this workflow into a skill and run evals on it.
```
```
Refactor my existing skill and benchmark it against the current version.
```

Claude will:

1. Ask clarifying questions about intent, trigger conditions, and licence
2. Draft or refactor the `SKILL.md` and reference files
3. Validate structure against the non-negotiable rules (line counts, description format, cold-start size)
4. Write eval prompts and run paired with/without-skill tests
5. Grade outputs, generate a benchmark report, and open the HTML viewer
6. Iterate based on review feedback
7. Optimise the description and package the skill

### Phases and what triggers them

| What you say | Where Claude picks up |
|---|---|
| "Create a skill from scratch" | Phase 1 — Understand Intent |
| "My skill isn't triggering" | Phase 6 — Description Optimisation |
| "Run evals on my skill" | Phase 4 — Evals |
| "Improve based on these results" | Phase 5 — Iterate |
| "Package this skill" | Phase 6 — Package |

---

## How it works

The skill has two layers:

**`SKILL.md`** — the routing layer. Defines the seven non-negotiable rules, a process overview, and a phase table that tells Claude which reference file to load at each stage. Under 200 lines so the full entry point loads in one pass.

**`references/`** — the work layer. Six reference files covering writing, eval workflow, improvement loop, description optimisation, and JSON schemas. Each is 200-300 lines and loaded only when that phase is active.

**`references/agents/`** — instruction files for subagents spawned during evals:
- `grader.md` — evaluates assertions against outputs and produces structured scores
- `comparator.md` — blind A/B comparison between two outputs
- `analyzer.md` — analyses why one version beat another

**`scripts/`** — Python scripts that run outside the context window:
- `run_eval.py` — runs a single eval case
- `run_loop.py` — orchestrates a full eval loop
- `aggregate_benchmark.py` — collates grading and timing results into a benchmark file
- `generate_report.py` / `generate_review.py` — produces the HTML viewer
- `improve_description.py` — optimises skill descriptions for trigger accuracy
- `quick_validate.py` — validates skill structure without running full evals
- `package_skill.py` — packages a skill folder into a `.skill` file

**`evals/`** — eval prompts and benchmark results. `baseline-benchmark.json` stores the reference scores for regression comparison.

---

## Contributing

Contributions are welcome. If you have a skill type that the builder handles poorly, or ideas for improving the eval methodology, please open an issue or pull request.

**Good places to start:**

- Improving assertion quality guidance in `references/eval-workflow.md`
- Extending the improvement loop with more generalisation heuristics
- Adding support for new output formats in the packager
- Writing evals for edge-case skill types (highly subjective outputs, multi-file skills, skills with complex runtime dependencies)

Please open an issue before starting significant work so we can discuss the approach first.

---

## License

This project is licensed under the [Mozilla Public License 2.0](LICENSE). You are free to use, modify, and distribute this skill, including in proprietary projects, provided that modifications to the licensed files are shared under the same licence.

The scripts in `scripts/`, the agent files in `references/agents/`, and the eval viewer are derived from Anthropic's [skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator) project and are subject to the [Apache License 2.0](LICENSE-APACHE). See [NOTICE](NOTICE) for the full list of affected files.
