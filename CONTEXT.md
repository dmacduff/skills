# Glossary

## Skill

A packaged set of instructions an agent loads to perform a kind of task.
Distributed at repo granularity via `npx skills@latest add dmacduff/skills`.

## Execution Environment (EE)

A container image, built with `ansible-builder`, that packages Ansible plus a
project's Python and Galaxy dependencies so runs do not depend on the control
node's state. Scaffolded only when the four-criterion rubric says so
(see ADR 0002).

## EE-ready manifests

The default dependency artifacts emitted in the exact form `ansible-builder`
consumes — a pip-compatible `requirements.txt` and a Galaxy `requirements.yml`
— so adding an EE later is one additional file, never a parallel dependency
spec.

## Derived artifact

A file generated from an authoritative source and never hand-edited. The
pip-compatible `requirements.txt` and PEP 751 `pylock.toml` are derived from
`uv.lock`, which is primary; derived files say so in their headers.
