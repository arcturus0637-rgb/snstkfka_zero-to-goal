# zero-to-goal

A Claude Code skill that runs a large software project from a stated end goal to a verified, merged result. You stay the judge at every decision that is costly to undo.

## What it does

- **It switches on when you state a final goal.** Small tasks (fixes, installs, questions) stay outside the harness.
- **A central agent designs; parallel branches build.** Work is split into 2-4 branches that never touch the same files. Each branch gets its own git worktree, and its own session where your environment supports that.
- **The contract comes first.** Module 0 is a shared contract: the message format, types, interfaces, schema, and fixtures built from real data. Branches build against the contract and use fakes for each other, so they can run in parallel without blocking.
- **A critic checks four checkpoints:** goal confirmation, approach confirmation, any method change, and before declaring done. It argues only with evidence, and stays silent if it has nothing. Both sides are shown to you in full, and you decide.
- **Every step ends in evidence:**
  - Test verdict lines are quoted.
  - Branch reports are re-verified centrally.
  - Noisy quantities are measured more than once.
  - Fixtures are checked for pinned values.
  - Privileged measurements go through probes that you run and the agent reads.
- **The permissions are safe by default.** Commits happen freely. Pushes, branch deletion and hard resets need your approval.
- **The state survives sessions.** `.claude/PROJECT.md` (goal, decisions, module goals, branch table) and `.claude/CHECKLIST.md` let a new session continue without asking for the goal again.

## Install

Copy the skill into your Claude Code skills folder:

```bash
git clone https://github.com/arcturus0637-rgb/snstkfka_zero-to-goal.git
cp -r snstkfka_zero-to-goal/skills/zero-to-goal ~/.claude/skills/
```

On Windows (PowerShell):

```powershell
git clone https://github.com/arcturus0637-rgb/snstkfka_zero-to-goal.git
Copy-Item -Recurse snstkfka_zero-to-goal\skills\zero-to-goal $HOME\.claude\skills\
```

Then start a project by stating its goal, for example: *"The final goal is a desktop app that shows disk usage as a live graph."* You can also say *"turn on the harness"*.

## Layout

```
skills/zero-to-goal/
|-- SKILL.md              the harness: switch, roles, flow, evidence rules, permissions
|-- references/critic.md  the critic agent's brief and output format
`-- assets/               PROJECT.md and CHECKLIST.md templates
```

## Where it came from

It was distilled from running a real multi-module desktop project this way:
- a contract on `main`
- three parallel branches
- an index, an activity collector and a UI
- elevated probes run by the user
- a central merge followed by a full re-test

The evidence rules are the mistakes that project, and earlier ones, actually made.

## License

MIT
