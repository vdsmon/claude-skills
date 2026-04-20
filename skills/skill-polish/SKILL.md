---
name: skill-polish
description: >-
  Post-mortem for any skill. Scans the conversation for friction
  (corrections, skipped steps, rejected tool calls), traces each to
  the responsible skill file, and applies concrete edits. Use when the
  user says "skill-polish", "polish the skill", "improve the skill",
  "that should have been automatic", "you skipped X", or "close the
  gaps". Works for ANY skill.
---

# Skill Polish

You just watched a skill execute in this conversation. Something wasn't smooth. Your job: find what went wrong, trace it to the skill's instructions, and fix the instructions so it doesn't happen again.

This is not about the code that was written — it's about the *skill itself*. The skill's reference files, its SKILL.md, its workflow descriptions. You're improving the tool, not the output.

## Why this matters

Skills get invoked thousands of times. A small friction point — a vague instruction that gets misinterpreted, a missing mandatory step, a "should" that needed to be a "must" — compounds across every future invocation. Fixing the skill file is high-leverage: one edit prevents the same mistake in every future conversation.

## How to find friction

Scan the conversation history for these signals, ordered from most obvious to most subtle:

1. **User corrections** — "why did you skip X", "no, do Y first", "that should have been automatic". These are direct instructions the skill failed to encode.

2. **Rejected tool calls** — The user blocked a tool use and provided guidance. The skill's instructions led to an action the user didn't want.

3. **Manual interventions** — The user had to step in and do something the skill should have handled. Look for moments where the user gave commands or information that the skill should have produced on its own.

4. **Wrong sequence** — Steps executed in the wrong order, or a step was skipped that should have run. The skill's flow control was ambiguous.

5. **Wasted work** — The agent did something that turned out to be unnecessary, then had to backtrack. The skill's instructions sent it down a dead end.

6. **Validated surprises** — The agent did something unexpected that the user *liked*. ("that's brilliant, add that to the skill"). These are techniques worth codifying.

For each signal, note:
- What happened (the friction)
- What should have happened (the desired behavior)
- Which skill file is responsible (trace it)

## How to trace friction to skill files

1. **Identify the skill** — Which skill was active when the friction occurred? Check `SKILL.md` frontmatter for the skill name, and look at which reference files were read during the conversation.

2. **Find the responsible file** — Read the skill's directory structure. Match the friction to the stage/step/section that governed the agent's behavior at that moment. Common locations:
   - `SKILL.md` — main workflow, stage summaries, pipeline flow
   - `references/<stage>.md` — detailed stage instructions
   - Frontmatter `description` — triggering issues

3. **Read the current text** — Always re-read the file with the `Read` tool before proposing edits. Skill files may have been modified since they were loaded earlier in this session (by the user, another session, or a previous `/skill-polish` run in this same conversation). Never rely on what you remember from earlier — the file on disk is the source of truth. Read the exact passage that led to the incorrect behavior. Understand *why* the agent misinterpreted it. Common root causes:
   - **Too vague** — "check the project docs" instead of "run /test-form"
   - **Too soft** — "auto-advance" when it needed "immediately continue, no pause"
   - **Missing entirely** — The desired behavior wasn't mentioned at all
   - **Wrong default** — The fallback behavior was wrong for this case
   - **Buried** — The instruction existed but was lost in a wall of text

## How to fix

For each friction point, produce a concrete edit — not a suggestion, an actual change to the file. Follow these principles:

- **Scripts over instructions.** Everything that COULD be a script SHOULD be a script. If a skill describes a deterministic sequence of commands (check X, then run Y, then verify Z), that sequence belongs in a shell script or mise task — not in prose the agent interprets at runtime. Scripts are reproducible, testable, and eliminate an entire class of agent misinterpretation. When you find inline command sequences in a skill, propose extracting them into scripts and having the skill reference the script instead. This is the single highest-leverage improvement you can make.
- **Be specific over general.** "Run `/test-form` for form task types" beats "consider running task-type-specific tests."
- **Explain the why.** Don't just add a rule — explain why it matters. The agent reading the skill is smart; if it understands the reasoning, it can handle edge cases the rule doesn't cover.
- **Match the weight to the risk.** A skipped step that silently produces wrong output needs bold formatting and explicit "do NOT skip" language. A minor sequence preference can be a gentle note.
- **Don't over-correct.** If the skill worked 90% of the time and failed on one edge case, add handling for the edge case. Don't rewrite the whole section.
- **Codify validated techniques.** If the agent improvised something good, write it into the skill with enough detail for future agents to reproduce it.

## Workflow

### Step 1: Identify the skill(s) used

List which skills were invoked in this conversation. If the user specified one, focus on that. Otherwise, identify the primary skill that had friction.

### Step 2: Gather friction signals

Scan the conversation systematically. Present findings as a numbered list:

```
1. SKIPPED STEP — Form testing was skipped, went straight to commit
   Should have: Run /test-form or fake-data Spark test
   Responsible: references/testing.md, Step 4.2

2. PREMATURE STOP — Stopped after PR instead of auto-advancing to feedback
   Should have: Immediately continued to Stage 9
   Responsible: references/pr.md "Next" section + SKILL.md Stage 8 summary

3. VALIDATED TECHNIQUE — Fake-data Spark test was improvised and worked well
   Should be: Documented as a named technique in references/testing.md
```

### Step 3: Propose edits

For each friction signal, show:
- The file path
- The current text (quoted)
- The proposed replacement
- Why this fixes the issue

Present all edits together for review. Don't apply yet.

### Step 4: Apply with approval

Use `AskUserQuestion` to present a picker with these options:
- **"Apply all"** — apply every proposed edit
- **"Apply selected"** — let the user specify which numbered edits to apply (follow up to ask which ones)
- **"Skip"** — don't apply, just note for later

After applying, also save relevant learnings as feedback memories if they contain insights that generalize beyond this specific skill.

### Step 5: Summary

Show what was changed, which files were modified, and a one-line note on how this improves future runs.
