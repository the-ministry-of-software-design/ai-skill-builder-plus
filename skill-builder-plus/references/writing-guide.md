# Writing Guide

Reference for Phase 3 - Drafting a skill.

---

## Anatomy of a Skill

```
skill-name/
├── SKILL.md                  (required - under 200 lines)
│   ├── YAML frontmatter      (name, description, license, compatibility, metadata)
│   └── Markdown instructions
├── references/               (first-class, not optional extras)
│   ├── domain-a.md
│   ├── domain-b.md
│   └── agents/               (subagent instruction files, if needed)
│       └── grader.md
├── scripts/                  (executable code for repetitive/deterministic tasks)
└── assets/                   (templates, fonts, icons — files used in output)
```

### SKILL.md frontmatter

```yaml
---
name: my-skill               # kebab-case, max 64 chars
description: |               # trigger + negative trigger + outcome
  TRIGGER: ...
  DO NOT TRIGGER: ...
  OUTCOME: ...
license: MIT                 # use https://choosealicense.com/ to pick one
compatibility: Requires Python 3   # omit if no special requirements
metadata:
  author: Your Name or Org   # optional
  version: 1.0.0             # set to 1.0.0 at first packaging
---
```

Ask the user for licence and author before drafting (SKILL.md Phase 3 pre-questions). Fill in
`compatibility` if you know the requirements from context (e.g. the skill bundles Python scripts);
otherwise ask. Set `version: 1.0.0` just before packaging (Phase 6).

## Progressive Disclosure - Three Tiers

**Tier 1 - Metadata (always in context)**
- YAML frontmatter only (~100 words)
- Claude reads this to decide whether to activate the skill
- This is the only part that costs context on every conversation

**Tier 2 - SKILL.md body (loaded when skill activates)**
- Maximum 200 lines
- Acts as a table of contents and quick-start guide
- Points to reference files; does not contain their content
- If you are approaching 200 lines, abstract to a reference file

**Tier 3 - Reference files and scripts (loaded on demand)**
- 200-300 lines each, focused on a single topic. Split any reference file that exceeds 300 lines.
- Claude reads only the file relevant to the current step
- Scripts execute without loading into context at all

**Why this matters:** Uncontrolled loading is a direct performance tax. Activating five skills that each load 1,000 lines floods 5,000 lines into context before any work begins. Total budget for all loaded skill descriptions is ~15,000 characters. Keep your SKILL.md lean so others fit too.

## The Three-Part Description

Every description must contain all three:

**1. Trigger** - what event or situation activates this skill?
- Include specific keywords that should fire it
- Example: "Use when the user says 'research what's trending' or asks to research any topic"

**2. Negative trigger** - what should NOT activate it?
- Be explicit; prevents over-firing
- Example: "Does not trigger for general web browsing or simple URL fetching"

**3. Outcome** - what does the skill produce?
- Example: "Produces a structured research brief with sources that other skills can consume"

Marketplace skills activate only ~20% of the time because of poor descriptions. A missing negative
trigger causes false positives; a vague outcome makes Claude unsure whether the skill applies.

**Template:**
```
TRIGGER: Use whenever [specific situations and keywords].
DO NOT TRIGGER: for [near-miss situations that share keywords but don't need this skill].
OUTCOME: produces [concrete deliverable description].
```

## Skills Are Capabilities, Not Documentation

The most common mistake: treating skills like documentation dumps. Ask yourself:

- What workflow moment does this skill activate during?
- What can Claude *do* with this skill that it could not do (or would do worse) without it?

Organise by workflow capability, not by tool:
- Not "Cloudflare skill" and "Docker skill" and "GCloud skill"
- But "DevOps deployment skill" that handles all three, routing to the right reference file

This also keeps total installed-skill count manageable.

## Cold-Start Test

After writing a SKILL.md, count the lines of the body (everything after the YAML frontmatter).
If it exceeds 500 lines on first activation, refactor immediately.

Practical check: imagine Claude has just activated the skill. What has loaded? SKILL.md body.
Nothing from references/ yet. Is that under 500 lines? If not, abstract sections to reference files.

## Writing Patterns

**Output format template:**
```markdown
## Output format
Use this exact structure:
# [Title]
## Summary
## Key findings
## Next steps
```

**Examples pattern:**
```markdown
**Example:**
Input: "Added user authentication with JWT tokens"
Output: `feat(auth): implement JWT-based authentication`
```

**Explaining the why (preferred over bare imperatives):**

Instead of: `ALWAYS include the date in the filename.`

Write: `Include the date in the filename so that multiple versions can coexist in the same directory
without collision - users often generate several drafts before settling on a final version.`

The model understands reasoning and can apply it to edge cases. Bare rules get followed literally
and break in situations the rule didn't anticipate.

## Domain Organisation

When a skill supports multiple related domains, route to domain-specific reference files:

```
cloud-deploy/
├── SKILL.md       (overview, selection logic)
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```

In SKILL.md: "Read `references/aws.md` if deploying to AWS, `references/gcp.md` for GCP."
Claude reads only the relevant file.

## Personalisation via References

A skill without user-specific context produces generic output. To personalise:

1. Add context files to `references/`: brand guidelines, tone of voice examples, audience personas
2. Reference them from SKILL.md: "Read `references/brand-guidelines.md` before writing any output"
3. Consider a shared context folder referenced across multiple skills - all skills benefit without
   duplicating content

## Self-Improvement Loop (optional but recommended for long-running skills)

Add a `learnings.md` file to the skill. After each task, the skill (or a wrap-up skill) appends
observations: what patterns emerged, what worked, what failed. Future invocations read this file.

Keep `learnings.md` structured and pruned. Dumping notes forever recreates the bloat problem.
Strip surplus information regularly (weekly or after each major iteration).
