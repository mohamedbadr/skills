---
name: change-review
description: Review code changes, including uncommitted work, along three independent axes — Plan (does the change do everything the plan, spec, or ticket asked, and nothing more?), Impact (did the change break any caller, contract, data, config, or behavior elsewhere in the project?), and Standards (does it follow the repo's conventions?). Verifies every finding against the code before reporting, then saves a review report. Use after implementing a plan and before manual review or committing, or when the user says "review my changes", "check against the plan", "did I miss anything", "did this break anything", or "review since <commit/branch>". Works with plans from the feature-plan skill, any other spec or ticket, or no plan at all.
---

# Change Review

Reviews a change before a human does, so the human spends their time on judgement, not on catching missed steps and broken callers. Read-only: never edit, stage, or commit code during the review.

## Step 1: Pin down the change

Default scope is **all uncommitted work**: staged, unstaged, and untracked files compared to `HEAD`. Other scopes only if the user names them.

| User says | Diff command |
|-----------|--------------|
| nothing / "my changes" | `git diff HEAD` plus `git ls-files --others --exclude-standard` (read untracked files whole) |
| "since X" / a branch | `git diff $(git merge-base X HEAD)` (two-dot against the merge-base, so uncommitted work is included) |
| a PR or commit range | that range only |

Stop early and tell the user if the ref does not resolve or the diff is empty. List the changed files and line counts before continuing, so the scope is visible.

## Step 2: Find the spec

Look in this order and use the first hit. Say which source was used.

1. A path or ticket the user passed.
2. A file in `.plans/` (from the feature-plan skill) whose name or **Affected files** table matches the changed files. If several match, pick the latest **Last updated** and say so.
3. Issue or work item IDs in recent commit messages or the branch name, fetched with the available tracker tool.
4. A matching spec under `docs/`, `specs/`, or similar.
5. Ask the user once. If they have none, skip the Plan axis and say so — never invent requirements.

## Step 3: Review the three axes

Run each axis in its own sub-agent when sub-agents are available, so no axis sees another's reasoning. Give each sub-agent the diff command, the changed-file list, the spec path (Plan only), and this instruction: **"Do not invoke skills or spawn further agents. Return findings only."** Without sub-agents, run the axes one after another.

- **Plan** — walk the spec step by step and mark each step Done, Partial, Missing, or Deviated, citing the diff hunk. Then flag scope creep (changes no step asked for), files in **Affected files** that were not touched, and assumptions or open questions the code silently decided.
- **Impact** — find what the change can break outside the diff. For every changed public symbol, contract, schema, config key, or behavior, search for its users and check they still work. Run the build and the existing tests related to the changed files. See [REFERENCE.md](REFERENCE.md#impact-checklist).
- **Standards** — check the diff against the repo's documented conventions (`CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, `CODING_STANDARDS.md`, `.editorconfig`, linter configs) and the patterns of neighbouring code. Fall back to the baseline in [REFERENCE.md](REFERENCE.md#standards-baseline). Skip anything a linter already enforces.

## Step 4: Verify every finding

Before reporting, re-open the cited file and confirm each finding against the current code. For Impact findings, confirm the caller really exists and really breaks. Drop what does not hold up; mark what is likely but unproven as **Plausible**. A short list of true findings beats a long list of guesses.

## Step 5: Report

Use the template in [REFERENCE.md](REFERENCE.md#report-template). Rules:

- Every finding has a severity (Blocker, Major, Minor, Nit), a `file:line`, the evidence (quoted spec line, caller location, or rule), and a concrete fix.
- Give each axis its own status and its worst finding. No single overall verdict: one clean axis must not hide a failing one.
- List what was **not** checked (tests that could not run, environments not available, areas outside the diff that were skipped).
- Map unchecked items from the plan's **Verification checklist** to the human: which ones still need manual testing.

Save the report as Markdown:

- With a `.plans/XXXX-<name>.md` spec: `.plans/XXXX-<name> review.md` next to it. Overwrite on re-run and refresh its date.
- Otherwise: `.reviews/XXXX-<short name>.md` at the project root, numbered like the feature-plan skill (highest existing plus one, from `0001`).
- Follow the user's instructions instead if they ask for another location or "don't save".

Tell the user the path, show the per-axis summary inline, and open the file in the editor. Then ask which findings to fix. Do not fix anything until the user says so.

## Notes

- Best run in a fresh session: the session that wrote the code shares its blind spots.
- Repeated runs will keep surfacing new minor items. Treat findings as leads; stop when no Blocker or Major remains, not when the list is empty.
