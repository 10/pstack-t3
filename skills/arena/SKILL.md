---
name: arena
description: "Spawn N parallel candidates at the same task, pick a base, graft the strongest parts of the losers into it. Use for /arena, 'arena this', 'throw it in the arena', or when one attempt at a non-trivial artifact would lock in the wrong shape."
disable-model-invocation: true
---

# Arena

A skill named in bold or in backticks, or run as `/<name>`, is the file `../<name>/SKILL.md` beside this skill's directory, and a principle named without its prefix is `../principle-<name>/SKILL.md`. Read the file and follow it. Most pstack skills are user-only, so your skill tool cannot load them. Every `delegate_task` call follows the defaults in the Subagents section of `../poteto-mode/SKILL.md`.

Open a task list (your harness's task tool, or a checklist file at `~/.local/state/pstack/checklists/<repo>-<branch>.md` when it has none) with one entry per phase (Frame, Fan out, Cross-judge, Pick, Graft, Verify) before launching anything.

## Phase A: Frame

Every candidate gets the same prompt, so the prompt is the contract.

1. State the artifact each candidate is producing.
2. Derive the rubric, 3-6 concrete gradeable criteria for what success looks like on *this* task. It is the picker's tool in Phase D. Candidates only see the task.
3. Read the `arena runners` line in `~/.config/pstack/pstack-models.md`. Run one seat per entry. With no line, use the parent model and the strongest model of each other eligible vendor that `orchestrator_capabilities` lists, up to three seats. Say so and suggest `/setup-pstack`. For `auto` or `inherit-parent` omit `target` on a `delegate_task` seat and `modelSelection` on a `t3_thread_launch` candidate. Keep configured panel efforts. If T3 rejects a seat, report it and follow the harness's rules for alternatives. Spawn more runners for multiple design directions. Repeat a model when the work is generation-bound.
4. Give each candidate its own output location (a git worktree where possible, otherwise `/tmp/arena-<slug>/candidate-<n>/`), per the **separate-before-serializing-shared-state** principle skill. A candidate that writes code gets its own worktree. Commit the base first, because uncommitted edits do not reach a new worktree.

## Phase B: Fan out

Spawn all N candidates in one turn, each with the task, the absolute path to the shared grounding, its own output path, and instructions to produce the artifact and a short rationale naming the alternatives it considered and what it rejected. A candidate that writes code is a `t3_thread_launch` thread titled `arena <slug>: candidate <n>`, with `workspaceStrategy: {"type":"worktree","baseRef":"<base commit>","branch":"arena-<slug>-<n>","startFromOrigin":false}`, `modelSelection: {"instanceId": ..., "model": ..., "options": ...}` from its seat, and the brief in `message`. Wait on it with `t3_thread_wait` and take its worktree path from `t3_thread_read`. Any other candidate is a `delegate_task` child with `mode: "async"`, the same title, `target` from its seat, and its `/tmp` output path. Its completion notification wakes you.

If a candidate produces no output, proceed with N-1 and note the dropout.

## Phase C: Cross-judge

After all Phase B candidates complete, choose one read-only judge from `arena cross-judge pool` in `~/.config/pstack/pstack-models.md`. With no line, use the strongest eligible model per vendor listed by `orchestrator_capabilities`. Take the first real entry from a vendor other than the parent's. Two provider instances can serve the same vendor, and Fable and Opus count as one. Skip `auto` and `inherit-parent`. If the parent's vendor is unknown, use the first real entry and report that the vendor check could not run. If no eligible judge runs, say so and pick the base in Phase D. While you read the candidates, the judge scores each by path label on each rubric criterion and recommends a base. Give the `delegate_task` child the rubric, absolute candidate paths, and a read-only brief.

## Phase D: Pick a base

Read every candidate end to end before picking. Score each against the rubric criterion by criterion, not on holistic feel, and compare with the cross-judge when Phase C ran one. Agreement on the base confirms the pick. Disagreement means one of you is biased or the rubric was ambiguous. Read both rationales before deciding.

Pick the base a future maintainer can extend most easily without breaking invariants. When two feel tied, prefer the cleaner boundary or smaller API, per the Laziness Protocol.

## Phase E: Graft

Walk each losing candidate once more for what is worth porting into the base, usually one or two things per candidate, not most of it. Fold each graft in by hand, per the **redesign-from-first-principles** principle skill, so the result stays coherent under one mental model. Don't paste mechanically.

If the candidates converge on one shape, that is strong agreement. Note it and ship the consensus shape. No graft is needed. If they wildly diverge, Phase A was under-specified. Reframe and re-run rather than averaging the divergence.

## Phase F: Verify

Verify the synthesized artifact like any other output, per the **prove-it-works** principle skill. If verification finds a problem the arena missed, either Phase A was wrong (re-frame and re-run) or a candidate caught it and you missed the graft (go back to Phase E). Don't paper over it.

## Outputs

One synthesized artifact, with a short synthesis note alongside naming the base and why, the cross-judge's verdict or that Phase C skipped the judge, each graft with its source candidate, what was rejected and why, any convergence or dropouts, and the verification result.
