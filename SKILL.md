---
name: memory-maintenance
description: Audit, organize, validate, and clean the persistent memory of an AI coding agent (Claude Code, Codex, or similar). Use when asked to inspect, organize or clean AI memory, remove noisy, outdated, duplicate or conflicting memories, or re-check memory after a Git branch switch, merge or worktree change (整理記憶、清理 memory、檢查過時或重複的記憶). Not for editing instruction files such as CLAUDE.md or AGENTS.md, rules, Skills or hooks.
---

# Memory Maintenance

## Purpose

Maintain accurate, relevant, non-duplicated AI memory while preventing stale, conflicting, irrelevant, or branch-incompatible information from influencing the current task.

This Skill manages memory as versioned knowledge with an explicit scope and lifecycle. It does not assume that every AI-related file is disposable memory.

## When to Use

Use this Skill when the user asks to:

- Scan or audit AI memory.
- Clean memory noise, duplicate, obsolete, or invalid information.
- Organize global, project, or branch-specific memory.
- Synchronize memory after a Git branch switch.
- Review branch memory after a merge or rebase.
- Promote verified branch knowledge into project memory.
- Archive or recover memory.

Do not run a full cleanup merely because a coding task has started.

## Required Reference

Before `review` or `cleanup`, read:

`references/execution-principles.md`

This reference holds the detailed checklists: what to examine first, what must not be removed, how to judge validity, what a cleanup plan must contain, which operations need confirmation, and recovery rules. If it disagrees with this file, this file wins. If this reference is missing or unreadable, do not perform destructive or potentially disruptive changes. Explain the limitation and continue only with safe, read-only inspection if appropriate.

## Platform Layout

Where memory is stored and how it is formatted depends on the agent platform. Identify the platform you are running on, then read only the matching file before `scan`:

| Platform | Read |
|---|---|
| Claude Code | `references/platform-claude-code.md` |
| Codex | `references/platform-codex.md` |

On any other platform, or when the platform cannot be identified, ask the user where memory is stored and in what format, and stay read-only until they confirm.

Rules for every platform:

- Instruction and configuration files (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, rule files, Skills, hooks, settings) are not memory. They are out of scope. If a memory really belongs in one of them, propose that to the user; do not write it there.
- Keep the store's native file format; do not convert files to another schema.
- If the store has an index or summary file, keep it consistent with the memory files as part of the same approved plan.
- Unless the platform file says otherwise, there is no branch-level store and no way to choose which memories load. Identify branch-specific memory from its content. `branch-sync` then reports which memories do not fit the current branch and proposes edits; it does not change what is loaded.
- An archive location is agreed with the user in the plan.

## Supported Memory Scopes

### Global

Knowledge intended to apply across repositories, such as stable development preferences, general conventions, and cross-project workflows.

### Project

Knowledge belonging to one repository and intended to remain valid across its relevant branches, such as confirmed architecture, business rules, schema conventions, and project-wide decisions.

### Branch

Knowledge that is valid only for a specific branch or work-in-progress, such as unfinished implementation details, branch-specific decisions, temporary workarounds, and unresolved issues.

When scope is uncertain, use the narrowest plausible scope and mark the item for review. Never promote branch knowledge to project or global scope without verification and approval.

## Operating Modes

Select the mode that matches the user's request. If the user does not specify a mode, begin with `scan` and `review`; do not assume permission to modify files.

### `scan`

Read-only discovery.

1. Identify the target directory and requested scope.
2. Detect repository root, current branch, and worktree when Git is present.
3. Discover candidate memory files within the authorized target.
4. Read and inventory their content and metadata.
5. Classify items where possible.
6. Report unreadable, ambiguous, or out-of-scope items.

Do not edit, move, rename, archive, or delete files.

### `review`

Read-only analysis and planning.

1. Perform or reuse a current scan.
2. Identify duplicates, conflicts, stale claims, noise, and branch-incompatible content.
3. Verify claims against available source code, configuration, project documentation, and Git history where relevant.
4. Identify promotion candidates: branch memory whose change appears to be merged, and memory that reads as a standing rule or a project specification. For each candidate, give the verification evidence and the suggested destination (project memory, an instruction file, or project documentation).
5. Produce a proposed change plan with reasons and evidence.
6. Separate high-confidence findings from items requiring human review.

Do not modify files. Listing a promotion candidate is not a promotion: promoting into project memory follows `promote` and its approval, and a move into an instruction file or project documentation is only proposed to the user, never written by this Skill.

### `cleanup`

Apply an approved cleanup plan.

1. Re-scan the target before applying changes.
2. Check for changes since the review, including Git status and file timestamps/content.
3. Preserve a recoverable copy or use an appropriate version-control mechanism.
4. Apply only approved operations.
5. Validate the resulting memory and inspect the diff.
6. Report every change and any operation not completed.

Default to dry-run. Explicit approval is required before deleting, moving, archiving, replacing, or materially rewriting memory. Approval for one plan does not authorize unrelated changes discovered during execution.

### `branch-sync`

Reconcile active memory context after a branch or worktree change.

1. Detect repository root, current branch, and worktree again; do not rely on cached identity.
2. Load global memory that is explicitly configured for the current agent.
3. Load project memory, while treating branch-sensitive claims as requiring validation.
4. Load only branch memory matching the current repository and branch.
5. Keep other branches' unfinished context out of the active working context.
6. Identify worktree-local or uncommitted context separately.
7. Report missing, conflicting, or ambiguous memory.

This mode changes the active memory selection/context only when the environment supports it. Do not delete or rewrite other branches' memory merely because the current branch changed.

### `promote`

Propose or perform promotion of branch knowledge into project memory.

Before promotion:

- Verify the relevant change is present in the target branch; a merge message alone is insufficient.
- Verify the information is still true in the target branch.
- Confirm it is applicable beyond the source branch.
- Preserve unique historical or migration context where useful.
- Obtain explicit approval before modifying project memory.

Never promote all branch memory wholesale.

### `archive`

Move approved obsolete or completed memory into an archive location while preserving provenance. Prefer archive over permanent deletion when historical context may be useful. Do not archive active or unresolved work.

## Git Branch and Worktree Rules

### Before a branch switch

When the Skill is invoked before a switch:

- Record repository root, branch, and worktree identity.
- Inspect `git status` without changing the working tree.
- Identify unfinished branch-specific context.
- Check whether memory files have uncommitted changes.
- Propose preserving relevant progress in the matching branch memory.
- Do not write branch-specific assumptions into project memory.

The Skill must not perform the branch switch unless the user separately requests and authorizes that operation.

### After a branch switch

When invoked after a switch, or when a changed branch is detected:

- Re-detect repository, branch, and worktree.
- Re-evaluate which memories apply.
- Keep global memory available.
- Use project memory only to the extent it remains applicable.
- Activate only matching branch memory.
- Treat other branches' unfinished notes as inactive historical context.
- Revalidate temporary notes against the new branch.

### Worktrees and concurrent agents

- Treat each worktree as a distinct working environment.
- A branch may be checked out in more than one worktree; branch identity alone may not identify uncommitted state.
- Keep uncommitted or worktree-local context separate where necessary.
- Never overwrite a file that another agent or process may be editing.
- If concurrent modification is suspected, stop writes and request synchronization.
- Do not remove memory because a worktree or branch was deleted.

## Memory Classification

Classify memory items using the categories below:

| Category | Meaning | Default handling |
|---|---|---|
| Permanent | Stable, long-lived convention or fact | Keep |
| Current | Verified current project information | Keep |
| Branch-specific | Valid only for a branch or work-in-progress | Isolate |
| Temporary | Short-lived task context | Reassess |
| Deprecated | Superseded by verified newer information | Archive or update |
| Duplicate | Repeats information already represented | Merge carefully |
| Invalid | Demonstrably false or inconsistent with verified evidence | Propose correction |
| Noise | No durable informational value | Propose removal |

Age alone is not evidence that a memory is obsolete. Lack of recent activity is not proof that work is complete.

## Safety Requirements

- Default to read-only `scan` and `review`.
- Never delete uncertain information.
- Never modify files outside the user's authorized scope.
- Do not treat every file under `.claude/`, `.codex/`, `.agents/`, `.ai/`, or another AI directory as memory.
- Protect user-authored instructions, agent configuration, Skills, Hooks, and project documentation.
- Do not overwrite user-maintained configuration.
- Do not run `git reset`, `git checkout`, `git switch`, branch deletion, or unrelated cleanup as part of memory maintenance.
- Do not discard uncommitted changes.
- Do not infer that a memory is safe to remove solely because a referenced class, file, or branch is not found in the current checkout.
- When evidence is insufficient or sources conflict, preserve the information and mark it `needs-review`.
- Do not expose secrets or copy credentials into memory reports.
- For destructive changes, obtain explicit approval and ensure a recovery path.
- If the environment has its own rule or hook for memory writes (an approval prompt or a write gate), follow it. It takes precedence over the approval steps in this Skill. Group the writes so that each group fits within the approval that gate grants.
- If a tool call is denied, or no permitted tool can perform an operation (for example deleting or moving a file), stop that operation. Do not reach it through another tool or path. Give the user the exact command to run and report the operation as not completed.

## Metadata Guidance

When the existing memory format supports metadata, use fields such as:

- `id`
- `type`
- `scope` (`global`, `project`, or `branch`)
- `repository`
- `branch`
- `worktree_id` when needed
- `status` (`active`, `in-progress`, `deprecated`, `archived`, `needs-review`)
- `created_at`
- `updated_at`
- `source`
- `description`
- `superseded_by` when applicable

Do not force a metadata migration merely to make files conform to this schema. Propose migration separately and preserve existing information.

## Required Report

For every completed scan, review, synchronization, or cleanup, report:

1. Mode and target scope.
2. Target directory.
3. Repository root, branch, and worktree when applicable.
4. Memory files scanned and any excluded paths.
5. Items retained.
6. Duplicates proposed or merged.
7. Items updated.
8. Items archived.
9. Items deleted.
10. Items requiring manual review.
11. Promotion candidates and their suggested destination.
12. Evidence or rationale for material decisions.
13. Git status and diff verification, if applicable.
14. Any incomplete operation or remaining risk.

Do not claim a change succeeded unless it was actually performed and verified.

## Completion Criteria

A maintenance operation is complete only when:

- The requested scope was identified.
- Discovered memory was classified or explicitly marked for review.
- Scope and branch compatibility were evaluated.
- Only approved changes were applied.
- No unrelated or user-maintained files were modified.
- Results and remaining uncertainties were reported.
