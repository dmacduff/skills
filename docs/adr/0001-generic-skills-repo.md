# 0001 — One generic skills repo, not per-domain repos

## Status

Accepted (2026-08-30)

## Context

The first skill to ship is an Ansible environment scaffolding skill. It could live
in a domain-named repo (`ansible-scaffold`), which is instantly legible to a
network-automation reader, or in a generic skills repo that accumulates every
future skill.

## Decision

One generic repo: `dmacduff/skills`, matching the `mattpocock/skills` convention.
The first release contains exactly one skill; the repo stays private until that
skill passes a clean-checkout install test.

## Consequences

- Distribution works at repo granularity: `npx skills@latest add dmacduff/skills`
  is published once, and every future skill ships through a URL people may
  already have installed.
- One live, growing repo reads as a practice; several thin single-skill repos
  read as abandoned experiments.
- Domain legibility becomes a README problem: each skill gets a prominent,
  deep-linkable README section, so a domain reader can be sent to the section
  rather than the repo.
