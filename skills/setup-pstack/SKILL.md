---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Detects your available models and writes ~/.config/pstack/pstack-models.md, which overrides the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Write `~/.config/pstack/pstack-models.md`, the file that sets pstack's model per role. Skills read it when they delegate.

## Steps

### 1. Detect available models

Call `orchestrator_capabilities` and keep the providers whose `canRunChildTask` is true. Their models are what you can pass to a `delegate_task` subagent in this session. That is the only source. If it returns no eligible model, say so and offer `inherit-parent` for every role. An entry is `<providerInstanceId>/<model>`, plus `(<effort>)` when the model lists an effort option (`effort` on Claude, `reasoningEffort` on Codex). Never write an entry it did not return. The aliases `inherit-parent` and `auto` are always valid even though they are not detected models.

### 2. Load current state

Read `~/.config/pstack/pstack-models.md` if it exists. Keep the provider, model, aliases and panel order already chosen for each active role. Drop roles absent from step 5. Older T3 files have no `# defaults: 3` marker, and their detected model choices are indistinguishable from user choices. Preserve them. Migrate the former unlimited budget to large and show the user the changes before writing.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Ask plainly in the thread. Offer these three options with these exact labels, and name the current budget when the file records one.

- `large - xhigh reasoning`
- `medium - high reasoning`
- `small - medium reasoning`

**(b) Apply it.** For unconfigured code, prose and judgment roles prefer Opus 5.5 at xhigh when detected. Use Opus at high for `swarm workers`. If unavailable, use the most capable detected model for code and judgment and a fast coding model for workers. For `arena runners` and `interrogate reviewers` prefer one seat each for Fable 5.2 at high, Opus 5.5 at high, and Gemini 4 Neon at xhigh where offered. For the `arena cross-judge pool` prefer Opus 5.5 at high and Gemini 4 Neon at xhigh. Fill unavailable seats from eligible vendors, preferring diversity. Use only provider/model entries that T3 lists. A provider instance identifies how to run the model. Its vendor identifies who made it.

`large` keeps each starting effort, `medium` lowers it one step and `small` lowers it two on `xhigh > high > medium > low`. Derive efforts from these starting levels on every run so reapplying a budget is idempotent. For an explicitly configured panel model with a nondefault effort, show its current value and let the user retain it. Use the highest available effort at or below the target. Mark a role with no suitable effort as needing a choice. A model with no effort option omits it. Preserve `auto` and `inherit-parent`.

**(c) Show the roles and confirm.** Show every role with its model, marking any real entry not in the detected set as needing a choice. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering the detected models plus `inherit-parent` and `auto` (both mean: this role runs on the parent thread's model) as the options. Ask plainly in the thread. For panel roles (arena runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena runs one judge. It tries only values from a model vendor other than the parent's, in list order, so order it by preference. An `inherit-parent` or `auto` value never judges. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every real entry written must be in the `orchestrator_capabilities` result. Its provider has `canRunChildTask` true, its model is listed under that provider, and its effort, when it has one, is a listed option at xhigh or below. A model with no effort option has no `(<effort>)`. `inherit-parent` and `auto` always pass. If a chosen real entry is not available, stop and ask again.

### 5. Write the file

Write `~/.config/pstack/pstack-models.md` (create `~/.config/pstack/` if it is missing) as a plain file with the header comments, `# defaults: 3`, a `# budget` line with the chosen label and its target effort, and one line per role, using the same labels poteto-mode uses. Overwrite the whole file so re-runs stay idempotent. Shape:

```
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent thread's model (omit delegate_task `target`). Alias entries in a panel list still count toward its fan-out.
# defaults: 3
# budget: large (xhigh)
feature, refactoring: <providerInstanceId>/<model> (<effort>)
bug-fix: <entry>
perf-issue: <entry>
hillclimb: <entry>
judgment and prose: <entry>
arena runners: <entry>, <entry>, <entry>
arena cross-judge pool: <entry>, <entry>
swarm workers: <entry>
interrogate reviewers: <entry>, <entry>, <entry>
```

### 6. Confirm

Tell the user the file was written and that it applies the next time a skill delegates, since skills read it then. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, read `../create-verification-skill/SKILL.md` from this skill's directory (the one your harness gave when it loaded this file, else `~/.claude/skills/setup-pstack/` on Claude Code and `~/.agents/skills/setup-pstack/` on Codex) and follow it. On no, move on without pushing.

### 8. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill in `.agents/skills/` or `.claude/skills/` that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." On yes, read `../create-formal-verification-skill/SKILL.md` from this skill's directory and follow it. On no, move on without pushing.
