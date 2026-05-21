---
name: xp
description: Use when the user writes "XP" or "xp", ignoring surrounding whitespace, or asks to run XP scoring, self-assessment, agent leveling, session learning, or progress history. This skill evaluates session work since the latest XP history entry, awards honest XP, updates level state, records history, extracts durable lessons for Agent Memory, and updates Project Instructions when stable project-specific knowledge was learned.
metadata:
  version: "0.1.0"
---

# XP Skill

Run this skill when the user writes `XP` or `xp`, ignoring surrounding whitespace.

XP exists to make the agent more reliable, autonomous, and useful. Maximize honest XP through user satisfaction, independent task completion, verification, and learning. Never invent work, hide uncertainty, ignore instructions, avoid difficult tasks, make careless changes, or award unearned XP.

## Storage

- Skill file: `~/.agents/skills/xp/SKILL.md`
- State file: `~/.agent/xp/state.md`
- History file: `~/.agent/xp/history.md`

The skill file stores rules only. It must not store per-session progress.

## State

Create the state file if missing:

```markdown
# XP State

Level: 0
XP: 0
XP Per Level: 1000
Max Level: 150
Rank: Clippy
Updated At: null
```

Do not add `Total XP`. The `XP` field is XP inside the current level only.

## XP Scoring

Use only these scores:

- `+10 XP`: the user is satisfied, or gave no negative feedback, and the agent completed the task independently.
- `0 XP`: the user is satisfied, or gave no negative feedback, but the agent could not find the error or complete the task independently despite having enough input data.
- `-20 XP`: the user is dissatisfied with the result.

Rules:

- No negative feedback means accepted; user satisfaction overrides self-assessment.
- When unsure, grade strictly.
- A task is a user-requested deliverable, decision, implementation, investigation, edit, review, explanation, or other substantive outcome.
- Do not score casual conversation, encouragement, jokes, status questions, preference discussion, or pure utility work unless the user asked for a concrete output, it was explicitly evaluated, it was mishandled, or it directly affected the requested outcome.
- Do not count plans as completed tasks, or explanations as implementation unless the user asked for explanation.
- Do not award positive XP for rejected results, hidden failures, invented work, secrets, credentials, or XP notes in Project Instructions.
- Score only tasks completed since the latest XP history entry. Do not award XP twice.
- Independent means the agent understood the task, drove execution, handled normal ambiguity, delivered the result, and did not need the user to rescue the core solution.
- Clarification, preference shaping, brainstorming, and collaborative naming/design do not cancel independence when the agent still drives synthesis and execution.
- Use `0 XP` when the accepted task had enough input data but the agent could not find the error, could not complete it, followed a wrong direction until corrected, needed the key solution from the user, was led step by step, or had the core solution rescued.
- Pure utility work such as status, staging, committing, pushing, copying files, or installing an already-built skill earns no positive XP. It can receive `0 XP` when accepted or `-20 XP` when mishandled.
- Award positive XP for substantive work behind utility steps: implementation, research, fixes, behavior design, or useful artifacts.
- History contains only scored tasks. Do not add utility work unless it receives `0 XP` or `-20 XP` because it was explicitly evaluated or mishandled.

## Score Corrections

If the user disputes the latest XP score or explains that a task in the latest XP report was accepted, rejected, independent, or not independent, update the latest history entry and state using the user's correction as authoritative evidence.

Do not correct older XP history entries by default. If the correction is not for the latest XP entry, tell the user that older corrections are not automatic and require an explicit request naming the entry to change.

When correcting the latest XP entry:

- change only the affected task score, Session XP, Level, XP, Rank, and `Updated At`;
- recompute final state from the state before that history entry plus the corrected Session XP using the normal leveling algorithm;
- preserve the original task description unless it was factually wrong;
- do not create a duplicate XP history entry for the same task;
- explain the correction briefly to the user.

## Leveling

- `1000 XP = +1 level`
- Initial level is `0`.
- Maximum level is `150`.
- XP is stored only inside the current level.
- Penalties can reduce level.
- Never go below `Level 0, XP 0`.
- At `Level 150`, progress is capped at `XP 0`.

Update algorithm:

1. Convert current progress to absolute progress: `absolute = level * xp_per_level + xp`.
2. Add Session XP: `absolute = absolute + session_xp`.
3. Clamp below zero: if `absolute < 0`, set it to `0`.
4. Clamp above max: if `absolute >= max_level * xp_per_level`, set `level = max_level` and `xp = 0`.
5. Otherwise set `level = floor(absolute / xp_per_level)` and `xp = absolute % xp_per_level`.
6. Recompute `rank` from the final level.

## Ranks

| Level | Rank |
|---:|---|
| 0-9 | Clippy |
| 10-19 | Toaster |
| 20-29 | Iron |
| 30-39 | Coffeemaker |
| 40-49 | Microwave |
| 50-59 | Thermostat |
| 60-69 | Robotvacuum |
| 70-79 | Smartfridge |
| 80-89 | Router |
| 90-99 | SmartTV |
| 100-109 | Smartspeaker |
| 110-119 | Server |
| 120-129 | Mainframe |
| 130-139 | Supercomputer |
| 140-149 | Neuralcluster |
| 150 | Ultimind |

## XP Command Workflow

When the user writes `XP` or `xp`, ignoring surrounding whitespace:

1. Analyze the current session since the latest XP history entry.
2. Identify every real task the user assigned since that latest XP history entry using XP Scoring rules.
3. If there are no new tasks, say that there are no new tasks since the latest XP history entry and the score did not change. Do not update the state file or append a history entry.
4. Score each task using the XP rules.
5. Sum Session XP.
6. Read or create the state file.
7. Update `Level`, `XP`, `Rank`, and `Updated At`.
8. Append a report to the history file.
9. Extract general lessons for Agent Memory.
10. Extract project-specific lessons for Project Instructions.
11. Show the user an XP Report.
12. State one concrete way to earn more XP next time.

Use the current date and local time for `Updated At` and history headings.

## Agent Memory

Agent Memory stores stable, reusable, cross-project lessons in English only: user preferences, communication style, recurring mistakes, development principles, and debugging or verification habits. Do not write temporary facts, secrets, credentials, unstable guesses, or one-session details.

Include short `Memory Notes` in XP Report for durable general lessons. Write `Agent Memory: saved` only after confirmed write; otherwise use `Agent Memory: candidates only, not saved` or `Agent Memory: candidates only; autopersistence unknown`.

## Project Instructions

Project Instructions store stable project-specific knowledge: architecture, stack, commands, conventions, test/lint/build instructions, known pitfalls, and stable project decisions.

Before updating Project Instructions, determine the project instruction system, preserve structure, avoid duplicates, add only stable useful knowledge, keep entries brief, and use the existing language/style.

If Project Instructions do not exist, do not create a new instruction file by default. Create one only when the user approves it, or when the project clearly lacks an instruction system and there is stable project-specific knowledge that would materially help future work. Choose the language that best fits the project if none exists.

Do not write secrets, tokens, credentials, temporary bugs, one-off task details, XP notes, or uncertain guesses.

## History Format

Append one entry per XP run:

```markdown
## YYYY-MM-DD HH:mm

Session XP: +10
Level: 4 -> 5
XP: 900 -> 0 / 1000
Rank: Clippy

### Tasks
- +10: Task description. Reason: accepted result, completed independently.
- 0: Task description. Reason: accepted result, but the agent could not complete it independently despite enough input.
- -20: Task description. Reason: user rejected the result.

### Learned
- General lesson:
- Project lesson:

### Memory Notes
- Durable general memory note:

### Next Improvement
- One concrete behavior to improve next time.
```

Use ASCII arrows in files unless the existing file already uses Unicode arrows.

## User Response Format

Respond to `XP` with:

```markdown
# XP Report

## Score
Session XP:
Level:
XP:
Rank:

## Tasks
- ...

## Learned

### General Memory
- ...

## Memory Notes
- ...

### Project Instructions
- ...

## Updated
- XP state file
- XP history file
- Agent Memory status
- Project Instructions, if changed

## Next Improvement
- ...
```

Keep the report factual and concise.
