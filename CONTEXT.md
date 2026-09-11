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

## Collection lock

`requirements.yml` as emitted by the scaffold: every collection the project
runs, direct and transitive, pinned to the exact version Galaxy resolved at
scaffold time. Galaxy has no separate lockfile, so this one file is both the
request and the resolution (see ADR 0004). Upgrades edit a direct entry and
re-resolve.

## Baseline

The `.ansible-lint-ignore` file the scaffold generates on a retrofit,
listing every semantic ansible-lint finding inherited in pre-existing files,
one line per file and rule. Baselined findings warn but do not block; the
list shrinks as lines are deleted (see ADR 0005).

## Mechanical normalization

The bounded set of changes the scaffold may make to pre-existing files with
the user's consent: auto-fixer output (whitespace, end-of-file, line endings,
ruff safe fixes) and `---` document markers. Anything that changes what the
automation does is not mechanical (see ADR 0005).
