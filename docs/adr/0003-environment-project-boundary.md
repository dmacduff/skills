# 0003 — scaffold-ansible-env emits environment only; project structure is a separate skill

## Status

Accepted (2026-08-30)

## Context

A scaffold that stops at the environment (uv-pinned Python, lock artifacts,
`requirements.yml`, pre-commit, the EE decision) drops the user into a
directory with a perfect toolchain and no Ansible: no `ansible.cfg`, no
inventory, no playbooks. The obvious temptation is to prepopulate those.

`ansible.cfg` is the borderline case — it configures the tool, which sounds
like environment. But its most valuable lines (`inventory`, `roles_path`,
`collections_path`) are pointers into project structure, and a prepopulated
config pointing at directories the scaffold refuses to create ships broken
references on day one. Stripped of structural pointers it is only preferences
about how automation is run — project territory either way.

## Decision

`scaffold-ansible-env` emits environment plus only the glue its own tools
need: `.gitignore`, `.pre-commit-config.yaml`, `.yamllint`, an empty-but-valid
`requirements.yml`, the generated README, and (when the rubric fires) the EE
files. It never emits `ansible.cfg`, `inventory/`, `roles/`, or playbook
stubs, and never touches existing Ansible content when run in a non-empty
project.

Project structure belongs to a future `scaffold-ansible-project` skill whose
first act is to call this one — the layering is the design. That skill is out
of scope for v1: it is the natural second commit to the generic repo.

The generated README also gives no starter `ansible.cfg` snippet: copy-paste
advice superseded by the future skill is cruft likely to go stale uncaught.

## Consequences

- "Minimal" stays true: the skill has no opinion about what the user is
  automating, only what they run it with.
- Users must create `ansible.cfg` and project layout themselves until the
  project-scaffold skill exists.
- The composability the skill was designed for (model-invoked, callable by
  other skills) is exercised by the planned layering rather than asserted.
