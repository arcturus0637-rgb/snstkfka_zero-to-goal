---
name: zero-to-goal
description: A project harness for taking a large software project from a stated end goal to a verified, merged result. A central agent plans, critic agents check four key decisions, and 2-4 parallel implementation branches each work in their own git worktree. Use it whenever the user states an overall goal for a multi-module or multi-day project ("I want to build X", "the final goal is...", "let's make an app that..."), asks to run work in parallel branches or sessions, or asks to "turn on the harness". Do not use it for one-line fixes, installs, research questions or chat.
---

# Zero to Goal: a project harness

This harness is for work too big for one sitting.
- It turns a stated goal into confirmed module goals, then into parallel branches.
- It makes sure each step ends in **evidence, not claims**.
- It keeps the user as the judge at every decision that is costly to undo.

## The switch

Turn the harness **on** only when the user states an **overall final goal** for a project.

Until then, and for small things, work normally. Small things are a one-line fix, an install, a lookup, a question, or chat.

Turn it **off** when the user says so.

Do not guess the scale and switch it on yourself. The user stating a goal is the switch.

## Roles

- **Central (you).** You design, choose methods, merge branches and report to the user.
  - Do the design serially, not in parallel. One coherent design beats several half-designs stitched together.
- **Critic.** A separate agent you start (Agent tool) only at the four checkpoints below. See `references/critic.md` for its brief.
- **Implementation branches.** Each independent unit gets its own **git worktree and branch**, and its own session if your environment can start sessions.
  - Otherwise, run each branch as a subagent with worktree isolation, and say that is what you did.

## Model and effort for sub-sessions

Pick the model and the `effort` level for every sub-session or subagent from how much reasoning the task needs. Do not run every branch at the central's settings; that spends the most expensive reasoning on work that does not need it.

Some hosts let an agent set `effort` only when an instruction file explicitly asks for it. This section is that instruction.

| Task | Model | Effort |
|---|---|---|
| Mechanical or read-only: search, listing files, taking screenshots, simple replacements, formatting | cheapest model | low |
| Implementing a module whose contract is already fixed: inputs, outputs and file boundaries are decided | middle model | medium |
| Design judgment, debugging with an unknown cause, a contract that needs interpreting | same model as central | high |
| The critic, merging branches and resolving conflicts, the final "done" check | same model as central | high or above |

- **Never run evidence judgment cheaply.** Reading a wrong measurement as a pass is the mistake the evidence rules below exist to stop, and it happens at exactly this step.
- **Write the choice into the branch brief:** the model, the effort, and one line of reason. The user can then see it and change it.
- **The central is the ceiling.** Some hosts cap a sub-session at the central's own model and effort and silently ignore a higher request (Claude Desktop's session tool does). If a task needs more reasoning than the central has, do not try to raise the child. Tell the user, and ask them to switch the session's model.
- **When a branch reports a blocker,** decide whether to send it back one tier up. Do not retry at the same setting with nothing changed; that retry adds no information.
- **Name tiers, not model ids.** Ids change; "cheapest, middle, same as central" does not.

## Flow

1. **Overall goal.** The user states it.
   - Ask about anything ambiguous, in detail, before you assume. Ask a few focused questions per round, not twenty.
   - Run the critic on the goal.
   - Show the user both sides at full length, then let them decide.
   - The goal is confirmed only when the user says so.
2. **Approach and module goals.** You propose the stack, the process layout and per-module goals.
   - Each module goal has measurable done criteria.
   - Run the critic on the approach.
   - The user edits it and confirms.
   - Implementation does not start before this.
3. **Contract first (module 0).** Before any parallel work, build the shared contract on `main` yourself. See "Contract-first splitting".
4. **Split and implement.**
   - Split so that **no two branches touch the same files**.
   - Keep to **2-4 branches at once**. Each one rebuilds context from scratch, so more branches cost more than they save.
   - Give each branch a brief that lists:
     - its allowed folders
     - its done criteria
     - its model and effort, with one line of reason (see "Model and effort for sub-sessions")
     - "never push"
     - "ask central for contract changes"
5. **Blockers.** A branch reports a blocker to you, and you propose a fix.
   - **Proposing a method change triggers the critic.**
   - The user decides.
6. **Merge.** You merge the branches, resolve conflicts, then rebuild and retest **everything** on the merged result. See "After a merge".
7. **Before declaring done.** Run the critic a final time on the whole result.
   - Parts that are each correct can still fail together. This is the only checkpoint that catches that.

## Contract-first splitting

Parallel branches only work if they meet at a fixed interface. Module 0 is that interface. It contains:
- **A contract document**: the source of truth for messages, ids, units, enums and error codes.
- **Type mirrors** for each language in the project, for example C# records and TypeScript types.
- **Interfaces** for each module, so other branches can code against fakes.
- **The storage schema**, if there is one, with table ownership per module.
- **Fixtures built from real data** where you can, so the UI and the tests see realistic shapes.

Rules for the contract:
- Only the central agent changes the contract, and only on `main`.
- A branch that needs a change asks central. It never edits the contract and never works around it.
- When you add an interface method that another branch will implement later, give it a **default implementation**. Returning an empty result is enough. `main` then keeps building while the implementer catches up.
- Record every contract decision in the document, including the user's decisions and the reasons.

The `.sln`, workspace files and other shared build files belong to central. Branches list what they need added, and you add it at merge time.

## The critic

It runs at **four checkpoints only**:
- goal confirmation
- approach confirmation
- a method change
- before declaring done

It does not review every commit or tool call.

- It analyses and criticises **with evidence**. No contrarianism, and no devil's advocate for its own sake.
- **Its scope:**
  - unmeasurable goals
  - goals that contradict each other
  - work too big for one session
  - gaps in premises
  - conclusions the evidence does not support
  - missing skills
  - outdated facts, such as an end-of-life runtime
- The user's own goal is in scope, not only your methods.
- If it has nothing to say, it stays silent. It never reports "no issues".

**Reporting:** give the user the critic's claims and evidence, and your claims and evidence, both at near-original length.
- Do not trim the critic into your own summary. That removes what the user needs to judge.
- The user decides.

## Evidence rules

These come from real mistakes. Each one cost a wrong conclusion.

- **Quote the verdict line** of a test run, for example `Passed: 41, Failed: 0`. Never judge a pass from the last few lines of output. Those may be a stack trace.
- **A branch's report is a claim, not proof.** Before marking a branch done, re-run its tests yourself and read its probe output yourself.
- **Never judge a noisy quantity from one measurement.** Latency, throughput and market depth all need a distribution.
- **Check that fixtures and test harnesses do not pin values the real app fills in.** Examples: a context flag hardcoded to `false`, an empty environment, a fake clock.
- **Check that test data has the same shape as real use.** A tokenised corpus is not real text, and a synthetic tree is not a real disk.
- **Report status in three words:** implemented, ran, or unverified. Never call something verified when it was not.
- **When you cannot run a measurement**, for example because it needs admin rights or a device you cannot reach, write a **probe**:
  - The probe is a small program that runs the real thing and writes a plain-text report file.
  - Hand the user the exact command.
  - Read the report file yourself afterwards; do not take their summary.
  - Print raw fields in the probe, such as event payload names and counts, so a wrong assumption shows up in the report.
- **Compare before and after.** "Reduced from 27% to 15%" is a result. "Fixed" is a claim.

## After a merge

- Rebuild every project and run every test suite on the merged `main`. Quote each verdict line.
- **Reinstall dependencies from the lockfile** before building, for example with `npm ci`. A stale `node_modules` from before the merge gives false type errors, or worse, false passes.
- Record the merge commits and anything still unverified in the project file.
- Branches stay unmerged until you merge them. When you report a paused state, always name the branches that are not yet merged.

## Permissions

- **Commit:** freely. More restore points are safer.
- **Push:** only with the user's approval, every time. Once a push is out, you cannot take it back.
- **Deleting branches, `git reset --hard`, removing worktrees:** ask first, because they destroy work.
- **Anything an elevated or privileged process must do:** the user runs it. You provide the command and read the result.

## Skills checkpoint

When work starts, list the skills that apply to this project. Examples: a design system, test-driven development, a security review, data visualisation.
- Skills are easy to forget when there are hundreds. Writing the list down is the checkpoint.
- The critic may point out a skill that is missing.
- If they conflict, the priority is **user instructions > process skills > implementation skills**.

## Project files

When the harness is on, keep two files in the project. Do not commit them; add them to `.gitignore`.

- `.claude/PROJECT.md` holds:
  - the overall goal
  - the user's decisions, with dates
  - the acceptance criteria
  - the confirmed approach and module goals
  - progress
  - a **branch table**: branch, worktree, head commit, state, and whether it is merged
- `.claude/CHECKLIST.md` lists the user's requests as items, ticked off as they are done. Create it only when a request has several items or spans several turns.

Templates are in `assets/`.

A new session reads both files first and carries on from them. It does not ask for the goal again.

## Reports

Every implementation report covers three things:
- what was built
- the method
- why that method

A completion report also covers:
- what was verified, and with which command, quoting the verdict lines
- what failed or could not be checked
- whether it is committed
- that nothing was pushed, unless the user approved a push
