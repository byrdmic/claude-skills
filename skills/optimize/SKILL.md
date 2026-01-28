---
name: prompt-polish
description: Turn a raw prompt into a high-quality Claude Code prompt by (1) exploring this repo for relevant context and (2) applying our prompting playbook. Output ONLY the optimized prompt.
argument-hint: "\"raw prompt\"" / <paste raw prompt here - multiline ok>
context: fork
agent: Explore
disable-model-invocation: true
allowed-tools: Read, Grep, Glob
---

# Prompt Polish (output-only)

## Inputs
Raw prompt (unoptimized):
$ARGUMENTS

## References (load these files)
- Prompting best practices: prompt-best-practices.md
- Output template: prompt-template.md

## Your job
You are NOT solving the user’s task. You are producing the *best possible prompt* they should run next in Claude Code.

### Step 1 — Gather repo context (read-only)
Use Glob/Grep/Read to quickly find the most relevant context for this raw prompt:
- Start with CLAUDE.md and README if they exist.
- Then locate the files/modules that the prompt likely touches.
- Read only what’s necessary and extract the “minimum sufficient context”.

### Step 2 — Rewrite the prompt using the best practices
Use as much guidance as possible from the prompt-best-practices.md.

### Step 3 - Run the refined prompt through plan mode
Once the prompt has been refined, switch to plan mode and immediately start the Design phase to create a valid plan based on the refined prompt.

### Step 4 - Ask the user to verify the plan
Once the plan has been created, ask the user if they would like to make any changes to the plan (like normal Claude Code plans).
