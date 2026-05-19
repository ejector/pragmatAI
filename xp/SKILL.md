---
name: xp
description: Use when the user writes exactly "XP" or asks to run XP scoring, self-assessment, agent leveling, session learning, or progress history. This skill evaluates the current session, awards honest XP, updates level state, records history, extracts durable lessons for Agent Memory, and updates Project Instructions when stable project-specific knowledge was learned.
---

# XP Skill

Run this skill when the user writes `XP`.

The purpose of XP is to become a more reliable, autonomous, and useful agent. The agent should strive to gain the maximum honest XP by satisfying the user and solving tasks independently. The agent should aim to reach Level 150 as quickly as possible, but only through legitimate performance.

Legitimate XP comes from completing real user-requested tasks, needing less help from the user, understanding intent better, verifying results when possible, learning from mistakes, and improving future behavior.

Never optimize for XP by inventing completed work, hiding uncertainty, ignoring instructions, avoiding difficult tasks, making careless changes, or awarding unearned XP.

## Storage

Production paths:

- Skill file: `~/.agents/skills/xp/SKILL.md`
- State file: `~/.agent/xp/state.md`
- History file: `~/.agent/xp/history.md`

During local development or debugging of this skill, if the user explicitly asks to test it inside the current project before installation, use project-local mirrors:

- `xp/SKILL.md`
- `.agent/xp/state.md`
- `.agent/xp/history.md`

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

- `+100 XP`: the user is satisfied, or gave no negative feedback, and the agent completed the task independently.
- `0 XP`: the user is satisfied, or gave no negative feedback, but the agent could not find the error or complete the task independently despite having enough input data.
- `-200 XP`: the user is dissatisfied with the result.

Rules:

- No negative feedback means the result counts as accepted.
- User satisfaction overrides agent self-assessment.
- When unsure, grade strictly.
- Normal discussion, preference shaping, brainstorming, and collaborative naming/design do not cancel independence.
- Do not count plans as completed tasks.
- Do not count explanations as completed implementation unless the user asked for an explanation.
- Do not award `+100 XP` if the user rejected the result.
- Do not hide failures.
- Do not invent memories, project knowledge, or completed work.
- Do not write secrets or credentials anywhere.
- Do not store XP notes in Project Instructions.

## Independence

A task was completed independently if the agent:

- understood the task without the user solving it for the agent;
- was the main executor;
- handled normal ambiguity;
- delivered the needed result;
- did not need the user to rescue the core solution.

User clarification does not cancel independence if it only clarified intent, scope, or preference.

A task also remains independent when the user participates in expected discussion or brainstorming, as long as the agent still drives the work, synthesizes the result, and does not require the user to solve the core problem.

A task required user help, and should receive `0 XP` if accepted, when the agent had enough input data but could not find the error, could not complete the task, followed a wrong direction until corrected, needed the user to supply the key missing solution, was led step by step, or had the core solution rescued by the user.

## Leveling

- `1000 XP = +1 level`
- Initial level is `0`.
- Maximum level is `150`.
- XP is stored only inside the current level.
- Penalties can reduce level.
- Never go below `Level 0, XP 0`.
- At `Level 150`, progress is capped at `XP 0`.

Examples:

- Level 4, XP 900, Session XP +200 -> Level 5, XP 100.
- Level 4, XP 100, Session XP -200 -> Level 3, XP 900.

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

When the user writes `XP`:

1. Analyze the current session.
2. Identify every real task the user assigned.
3. Score each task using the XP rules.
4. Sum Session XP.
5. Read or create the state file.
6. Update `Level`, `XP`, `Rank`, and `Updated At`.
7. Append a report to the history file.
8. Extract general lessons for Agent Memory.
9. Extract project-specific lessons for Project Instructions.
10. Show the user an XP Report.
11. State one concrete way to earn more XP next time.

Use the current date and local time for `Updated At` and history headings.

## Agent Memory

Agent Memory stores general durable lessons that are not tied to one project.

Language: English only.

Write only stable, reusable lessons:

- user preferences;
- communication style;
- recurring mistakes to avoid;
- general development principles;
- debugging and verification habits.

Do not write temporary facts, secrets, tokens, credentials, unstable guesses, or random one-session details.

If the environment has no writable Agent Memory system, list memory candidates in the XP Report instead of pretending they were saved.

To make useful lessons easier for automatic memory systems to capture, include a short `Memory Notes` block in the XP Report whenever there are durable general lessons. Keep it English-only, factual, stable, and written as direct preference or behavior notes. Do not include temporary session details or project-specific facts there.

Memory status rules:

- Write `Agent Memory: saved` only when a real memory backend confirms the write.
- Write `Agent Memory: candidates only, not saved` when no writable memory backend is available.
- If automatic memory may exist but cannot be verified, write `Agent Memory: candidates only; autopersistence unknown`.

## Project Instructions

Project Instructions store stable project-specific knowledge:

- architecture;
- stack;
- commands;
- conventions;
- test, lint, and build instructions;
- known project pitfalls;
- stable project decisions.

Before updating Project Instructions:

- determine which project instruction system the current project uses;
- preserve existing structure;
- avoid duplicate entries;
- add only stable and useful knowledge;
- keep entries brief;
- use the same language and style already used by the project.

If Project Instructions do not exist and there is stable project-specific knowledge worth saving, create the most appropriate instruction file or system for the project. Choose the language that best fits the project if none exists.

Do not write secrets, tokens, credentials, temporary bugs, one-off task details, XP notes, or uncertain guesses.

## History Format

Append one entry per XP run:

```markdown
## YYYY-MM-DD HH:mm

Session XP: +100
Level: 4 -> 5
XP: 900 -> 0 / 1000
Rank: Clippy

### Tasks
- +100: Task description. Reason: accepted result, completed independently.
- 0: Task description. Reason: accepted result, but the agent could not complete it independently despite enough input.
- -200: Task description. Reason: user rejected the result.

### Learned
- General lesson:
- Project lesson:

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
