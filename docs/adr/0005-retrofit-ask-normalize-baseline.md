# 0005 — Retrofits: ask once, normalize mechanically, baseline semantic findings

## Status

Accepted (2026-09-11). Amends ADR 0003.

## Context

ADR 0003 promised that the skill "never touches existing Ansible content."
Step 5 then mandated `pre-commit run --all-files` as the completion gate,
with auto-fixing hooks in the roster and a yamllint config requiring `---`
document markers. On any retrofit with lint findings, the promise and the
gate could not both hold: the fixers rewrite pre-existing files as a side
effect of the mandated command, and a literal agent had to break one rule
or the other (issue 4).

A second problem sits behind the first. The ansible-lint pre-commit hook is
published with `pass_filenames: false` and `always_run: true`, so it lints
the whole tree on every commit no matter which files are staged. Scoping the
scaffold's lint run to its own files does not help: one semantic finding in
a pre-existing playbook leaves the user with a repo they cannot commit to.

Options weighed:

- Keep the read-only promise and scope the gate to skill-created files
  (leaves the repo uncommittable).
- Drop the promise and normalize everything the hooks flag, semantic
  findings included (a larger edit than the user thinks they approved).
- Ask, normalize mechanically, and baseline what remains.

## Decision

Pre-existing content is input the skill may change in two bounded ways,
both approved by one question at survey time:

1. **Mechanical normalization**: auto-fixer output (whitespace, end-of-file,
   line endings, ruff's safe fixes) plus `---` document markers.
2. **Manifest pinning**: rewriting a pre-existing `requirements.yml` per
   ADR 0004.

The question folds in its precondition rather than asking it separately: if
the tree has uncommitted changes to pre-existing files, the question says
so and suggests committing or stashing first so the normalization diff is
reviewable on its own. It also says that `pre-commit install` will apply the
same fixers to any pre-existing file the next time it is committed, so "no"
defers the change rather than preventing it.

Semantic ansible-lint findings in pre-existing files (FQCN, `no-changed-when`,
naming) are never fixed. They are project work, and ADR 0003 already places
project work outside this skill. They are baselined with
`ansible-lint --generate-ignore`, which writes `.ansible-lint-ignore` one
line per file and rule. Baselined findings print as non-fatal warnings on
every run and retire by deleting lines. A pre-existing ignore file is never
overwritten; new lines are appended.

"No" scopes the lint gate to skill-created files via `pre-commit run
--files`, and the done-criterion applies to that scope. The baseline is
generated either way, since it does not modify pre-existing files.

Interaction budget: greenfield stays at a maximum of two questions (EE
criteria, build check). Retrofit adds exactly one. The repo README carries a
decision table so a user can put every answer in the prompt and be asked
nothing.

## Consequences

- ADR 0003's "never touches existing Ansible content" is narrowed to "never
  changes the meaning of existing Ansible content." Its structural
  boundary (no `ansible.cfg`, inventory, roles, playbooks, CI) is unchanged.
- A retrofit always ends committable: the gate is green on its scope and
  inherited debt is visible in one file rather than blocking every commit.
- The generated README derives its claims about pre-existing files from
  what this run actually did, so it cannot promise a posture the steps do
  not enforce.
- `.ansible-lint-ignore` is a skill-created file. The skill's own output
  never appears in it.
