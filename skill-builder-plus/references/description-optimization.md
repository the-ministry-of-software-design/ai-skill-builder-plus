# Description Optimization

Reference for Phase 6 - Optimising the skill description for reliable triggering.

Scripts and assets are bundled inside skill-builder-plus. Set `SKILL_DIR` to the installed path of
skill-builder-plus, which is shown in the `available_skills` list in your context.

---

## Why This Matters

The description field is the *only* mechanism that determines whether Claude invokes a skill.
Marketplace skills activate only ~20% of the time because of poor descriptions.

Run this phase after the skill's content is stable - optimising the description before the skill
works correctly is wasted effort.

## How Skill Triggering Works

Skills appear in Claude's context with their name + description. Claude decides whether to consult
a skill based on that description alone. Important: Claude only consults skills for tasks it cannot
handle easily on its own. Simple, one-step queries rarely trigger skills even if the description
matches. Complex, multi-step, specialised queries reliably trigger skills when the description
matches.

This means eval queries should be substantive - not "read this PDF" but "extract the revenue table
from this quarterly report and flag any figures that changed by more than 15%".

---

## Step 1 - Generate Trigger Eval Queries

Create 20 queries: 8-10 should-trigger and 8-10 should-not-trigger.

```json
[
  {"query": "the user prompt", "should_trigger": true},
  {"query": "another prompt", "should_trigger": false}
]
```

**Should-trigger queries (8-10):**
- Different phrasings of the same intent - some formal, some casual
- Cases where the user doesn't name the skill or file type but clearly needs it
- Uncommon use cases
- Cases where this skill competes with another skill but should win

**Should-not-trigger queries (8-10 - these are the most valuable):**
- Near-misses: share keywords or concepts with the skill but need something different
- Adjacent domains where a naive keyword match would fire but shouldn't
- Cases where the query touches on something the skill does, but another tool is more appropriate

**What to avoid:** obvious negatives. "Write a fibonacci function" as a negative for a PDF skill
tests nothing. The negative cases should be genuinely tricky.

**Query quality bar:** queries must be realistic and specific. Include file paths, personal context,
column names, company names, URLs. A little backstory. Some casual speech or abbreviations.
Mix short and long. Focus on edge cases, not clear-cut cases.

**Bad:** `"Format this data"`, `"Extract text from PDF"`, `"Create a chart"`

**Good:** `"ok so my boss just sent me this xlsx (called something like 'Q4 sales final FINAL v2.xlsx')
and she wants me to add a column showing profit margin as a percentage - revenue is col C, costs col D i think"`

## Step 2 - Review with User

Present the eval set using the HTML template bundled with this skill:

1. Read `$SKILL_DIR/assets/eval_review.html`
2. Replace `__EVAL_DATA_PLACEHOLDER__` with the JSON array (no quotes - it's a JS variable)
3. Replace `__SKILL_NAME_PLACEHOLDER__` and `__SKILL_DESCRIPTION_PLACEHOLDER__`
4. Write to a temp file and open it (or in Cowork, save to workspace and provide a link)
5. User edits queries, toggles should-trigger, adds/removes entries, clicks "Export Eval Set"
6. The file downloads to `~/Downloads/eval_set.json` (check for `eval_set (1).json` if multiple)

Do not skip this review. Bad eval queries lead to bad descriptions.

## Step 3 - Run the Optimization Loop

Tell the user: "This will take a few minutes - I'll run the loop in the background and update you."

Save the eval set to the workspace, then:

```bash
cd $SKILL_DIR
python -m scripts.run_loop \
  --eval-set <path-to-trigger-eval.json> \
  --skill-path <path-to-skill> \
  --model claude-sonnet-4-6 \
  --max-iterations 5 \
  --verbose \
  > <workspace>/description-optimization-log.txt 2>&1 &

LOOP_PID=$!
```

Use `claude-sonnet-4-6` (or whatever model is powering the current session) so the triggering test
matches what the user actually experiences.

Periodically tail the log to give the user updates:
```bash
tail -20 <workspace>/description-optimization-log.txt
```

The script: splits the eval set 60/40 train/test, evaluates the current description (3 runs per
query for reliability), calls Claude to propose improvements based on failures, re-evaluates each
new description on both splits, iterates up to 5 times. Selects the best description by test score
(not train score) to avoid overfitting.

## Step 4 - Apply the Result

The script outputs JSON with `best_description`. Update the skill's SKILL.md frontmatter.

Show the user before/after side by side and report the train/test scores for both.

Kill the log tail if still running:
```bash
kill $LOOP_PID 2>/dev/null
```
