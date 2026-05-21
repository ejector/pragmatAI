---
name: buff
description: Use when the user writes exactly "buff", ignoring surrounding whitespace. This skill buffs the active target by running a scored improvement loop: rate the current work from 0 to 10, propose changes to reach 10, apply them, and repeat until the score is 10 or no meaningful score gain remains.
metadata:
  version: "0.1.0"
---

# Buff

Use this skill when the user writes exactly `buff`, ignoring surrounding whitespace.

Improve only the active target: current diff, changed files, branch/PR diff, document, skill, prompt, or artifact the user asked to improve. If the target is unclear, infer it from active changes; ask only when multiple unrelated targets make that unsafe. Do not expand scope.

## Core Loop

Repeat these steps:

1. Evaluate the current changes from `0` to `10` and record the score.
2. If the score is `10`, report that there is nothing meaningful left to improve and stop.
3. If there is a previous loop score and the score changed only slightly, report that further improvement is not worth continuing and stop.
4. Propose concrete changes that would raise the score toward `10`.
5. Apply the proposed changes.
6. Verify with the strongest practical method.
7. Return to step 1.

Default "changed only slightly" threshold: less than `0.5` score improvement since the previous loop score. Use a stricter threshold if the user specifies one.

## Scoring

Score against the user's goal and the active target.

Use this scale:

- `10`: excellent; no meaningful improvement remains.
- `8-9`: strong; only minor issues remain.
- `6-7`: acceptable but meaningfully improvable.
- `4-5`: works partially; notable gaps or quality issues.
- `1-3`: poor; major issues.
- `0`: unusable or wrong target.

When scoring, prioritize correctness, user requirements, safety, clarity, maintainability, verification, and fit with existing conventions.

## Change Rules

Apply only changes that plausibly improve the score.

Do not:

- make unrelated refactors;
- churn wording/style without score impact;
- change user-owned work outside the target;
- hide uncertainty;
- claim verification that was not run;
- keep looping after score improvement becomes negligible.

## Verification

After each applied change pass, verify with the strongest practical method available: tests, lint, typecheck, build, formatting, previews, screenshots, or direct reread.

If verification cannot be run, state that and continue only when the improvement is still justified.

## Output

At the end, report:

```markdown
## Buff Report

Scores: initial -> ... -> final
Changes made:
Verification:
Stopped because:
Remaining risks:
```

Keep it brief.
