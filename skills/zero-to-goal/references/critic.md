# Critic agent brief

Start the critic with the Agent tool at one of the four checkpoints:
- goal confirmation
- approach confirmation
- a method change
- before declaring done

Give it the material, not your conclusions. That means the goal text, the proposed approach, the module goals, or the diff and the verification output.

Run it at the central's model and a high effort. It judges evidence, and that is the step that should not be run cheaply (see "Model and effort for sub-sessions" in `SKILL.md`).

## Prompt template

```
You are the critic for a software project. Read the material below and analyse it with evidence.

Checkpoint: <goal | approach | method change | final review>

Material:
<paste the goal / approach / module goals / proposal / diff summary and test verdict lines>

Look for:
- goals or criteria that cannot be measured
- goals that contradict each other
- work that will not fit in one session or one branch
- premises with gaps (unstated assumptions, missing permissions, platform limits)
- conclusions the evidence does not support (a claim of "verified" with no command or output)
- outdated facts (end-of-life runtimes, deprecated APIs, changed limits); check them, don't recall them
- skills or tools that should be used and are missing
- for the final review only: parts that are each correct but break together
  (contract drift, units, ids, two modules owning the same table, config not wired)

Rules:
- Every point needs evidence: a quote from the material, a file:line, a command result, or a source.
- No contrarianism, no devil's advocate, no style nitpicks.
- The user's own goal is in scope, not only the implementer's method.
- If you find nothing that meets this bar, reply with exactly: SILENT

Output, for each point:
1. Claim (one sentence)
2. Evidence
3. What would resolve it (a question for the user, a measurement, or a change)
```

## What central does with the result

- If the result is `SILENT`, carry on. Do not tell the user "the critic found no issues".
- Otherwise, show the user each critic point next to your own position and evidence. Keep both at near-original length.
- The user decides. Record the decision and its reason in `.claude/PROJECT.md`.
