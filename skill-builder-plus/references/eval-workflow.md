# Eval Workflow

Reference for Phase 4 - Running and evaluating test cases.

Scripts and agents are bundled inside skill-builder-plus. Set `SKILL_DIR` to the installed path of
skill-builder-plus, which is shown in the `available_skills` list in your context.

---

## Required Workspace Layout

The `aggregate_benchmark.py` script uses `glob("eval-*")`. **Every eval directory must start with
`eval-`** or the script silently finds nothing. The `run-1/` layer under each config is also
required - the script skips config dirs that contain no `run-*` subdirs.

```
<skill-name>-workspace/iteration-1/
├── eval-<name>/                    ← MUST start with "eval-"
│   ├── eval_metadata.json
│   ├── with_skill/
│   │   └── run-1/                  ← required layer
│   │       ├── grading.json        ← here, NOT in outputs/
│   │       ├── timing.json
│   │       └── outputs/            ← produced files go here
│   └── without_skill/
│       └── run-1/
│           ├── grading.json
│           ├── timing.json
│           └── outputs/
├── benchmark.json
└── review.html
```

**The three mistakes that break aggregation:**
1. Eval dir not starting with `eval-` (e.g. `linkedin-post-skill` → invisible to script)
2. `grading.json` at `with_skill/outputs/grading.json` instead of `with_skill/run-1/grading.json`
3. No `run-1/` directory (script skips config dirs with no `run-*` subdirs)

---

## Sequence

1. Write evals + eval_metadata.json → 2. Spawn all runs in parallel → 3. Draft assertions while
runs execute → 4. Write grading.json + timing.json → 5. Aggregate → 6. Generate viewer → 7. Review
→ 8. Save baseline to evals/

---

## Evals and Metadata

Save evals to `evals/evals.json` (see `references/schemas.md` for full schema).
Write `eval_metadata.json` inside each eval directory before spawning:

```json
{ "eval_id": 1, "eval_name": "what-this-tests", "prompt": "...", "assertions": [] }
```

---

## Spawning Runs

Spawn **with_skill and without_skill in the same turn**. Include this exact save path in every
subagent prompt - subagents will invent their own structure if not told explicitly:

```
Save ALL output files to: <workspace>/iteration-N/eval-<name>/with_skill/run-1/outputs/
Do not create subdirectories. Write session_summary.md into outputs/.
```

For an existing-skill improvement: snapshot first (`cp -r <skill-path> <workspace>/skill-snapshot/`),
then point the baseline at the snapshot using `old_skill/run-1/outputs/`.

---

## Assertions (while runs execute)

3-5 per eval. Must be objectively verifiable - prefer grep, wc -l, or file existence over
eyeballing. If an assertion passes equally in both configurations it is not discriminating; note this
in your analyst pass.

Update `eval_metadata.json` with assertions once drafted.

---

## grading.json and timing.json

Write both to `<eval-dir>/<config>/run-1/` (not inside `outputs/`).

**grading.json** - fields must be exactly `text`, `passed`, `evidence`:
```json
{
  "expectations": [
    { "text": "assertion text", "passed": true, "evidence": "what confirmed this" }
  ],
  "summary": { "passed": 3, "failed": 1, "total": 4, "pass_rate": 0.75 }
}
```

**timing.json** - token/duration data from the task completion notification. Save immediately:
```json
{ "total_tokens": 84852, "duration_ms": 23332, "total_duration_seconds": 23.3 }
```

---

## Aggregating

```bash
cd $SKILL_DIR
python -m scripts.aggregate_benchmark <workspace>/iteration-N --skill-name <name>
```

Verify with `find <workspace>/iteration-N -name "grading.json"` before running if unsure.

---

## Generating the Viewer

```bash
python $SKILL_DIR/scripts/generate_review.py \
  <workspace>/iteration-N \
  --skill-name "my-skill" \
  --benchmark <workspace>/iteration-N/benchmark.json \
  --static <workspace>/iteration-N/review.html
# iteration 2+: add --previous-workspace <workspace>/iteration-<N-1>
```

Provide a `computer://` link to review.html. Tell the user to click "Submit All Reviews" when
done to download feedback.json.

---

## Saving to evals/

After the first successful run — once the user has reviewed the viewer and you have a benchmark —
save the persistent development record into the skill's own `evals/` folder:

- **`evals/evals.json`** — already written during setup; update assertions if you refined them
- **`evals/baseline-benchmark.json`** — copy of `<workspace>/iteration-N/benchmark.json`
- **`evals/README.md`** — short summary: overall pass rate, one line per eval describing what it
  tests, and a list of the key discriminating assertions (the ones that reliably separate
  with-skill from without-skill)

The workspace (`<skill-name>-workspace/`) is ephemeral scratch space. `evals/` is the keeper.
Anyone picking up the skill later should be able to run the evals and compare against the baseline
without needing the workspace. The `evals/` folder is excluded from packaging but stays in the
source folder alongside the skill.
