# 0002 — Execution environment decision: infer, then ask, then default to no

## Status

Accepted (2026-08-30)

## Context

`scaffold-ansible-env` must decide whether to scaffold an Ansible execution
environment (EE). The rubric is fixed — scaffold an EE when any one is true:

1. The control node is not the user's to mutate (shared, long-lived, someone
   else's box).
2. The control node's platform ≠ the collection's requirements (system-level
   deps such as `libssh`, compilers, a Python the distro won't provide).
3. More than one person or pipeline runs it, so "works on my machine" has
   actual victims.
4. The run is scheduled or unattended, where a drifted host is found by an
   outage rather than a human.

The open question was mechanism: ask the user the four questions, infer from
the repo, or always default to no EE and only state the rule.

## Decision

Hybrid, in strict order:

1. **Infer** — evaluate each criterion from available context (the user's
   request, the repo, the collections in `requirements.yml`; a collection
   needing system-level deps makes criterion 2 true on its own).
2. **Ask** — for criteria that cannot be inferred, ask the user; the skill is
   model-invoked inside an interactive agent, so one short question is cheap.
3. **Default to no** — any criterion true scaffolds the EE
   (`execution-environment.yml` consuming the already-emitted
   `requirements.txt` and `requirements.yml`); all false or unresolvable means
   no EE, manifests stay EE-ready, and the generated README states the four
   criteria so the user can opt in later with a one-file change.

## Consequences

- The EE is a first-class output the skill actually decides about, not an
  afterthought it dodges — a pure default-no design was rejected for reading
  as anti-EE.
- Silence still produces the shortest dependency graph, preserving "a demo has
  no retry budget."
- Because `ansible-builder` consumes exactly what the uv path already emits,
  "add an EE" stays a small additional file rather than a parallel dependency
  spec.
