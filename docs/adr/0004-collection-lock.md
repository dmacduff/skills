# 0004 — requirements.yml is the collection lock: exact pins, full graph, project-local resolution

## Status

Accepted (2026-09-11)

## Context

The scaffold locks every Python dependency exactly (`uv.lock`, with derived
exports) and refuses a floating base-image tag, yet emitted `requirements.yml`
with collections listed by name only. Two machines, or one machine on two
dates, could resolve different collection versions while every Python
package matched (issue 3).

Galaxy has no lockfile. `requirements.yml` is the only file
`ansible-galaxy` and `ansible-builder` read, and it constrains only the
collections it lists: pinning `cisco.ios` still lets `ansible.netcommon` and
`ansible.utils` float. The skill also never ran `ansible-galaxy`, so there was
no resolved version to write.

Options weighed:

- Leave unpinned and document the asymmetry.
- Pin direct entries only.
- Pin direct entries exactly, transitives as ranges with the resolved
  version as the floor.
- Pin the full resolved graph exactly.

## Decision

`requirements.yml` is the collection lock. Step 4 installs the direct
collections project-locally (`-p .ansible/collections`, the path ansible-lint
already uses for its own installs and which `.gitignore` already covers),
reads the resolved graph with `ansible-galaxy collection list`, and rewrites
the manifest with every collection, direct and transitive, at its exact
resolved version. Transitives sit under a section comment saying what they
are and that they are regenerated rather than hand-edited; no per-entry
attribution, because building the reverse dependency map is work whose
output nobody acts on.

Exact pins, not ranges, at both levels. A range records a floor and a
ceiling for a resolver to work within, and the result of that resolution
has nowhere to live. A floor on a transitive is therefore an unpinned entry
with extra typing. The traditional cost of exact pins, hand-maintained
upgrades, is low here: the skill runs inside an agent that can re-resolve,
test, and rewrite on request.

## Consequences

- Reproducibility now covers the collection graph, matching the posture the
  scaffold already took for Python and the EE base image.
- `requirements.yml` is derived from a resolution, and the generated README
  says so and gives the upgrade path: edit the direct entry, delete the
  transitive section so stale pins cannot constrain the new resolution,
  reinstall into a fresh path, rewrite pins.
- Only the `collections:` list is generated. `roles:` and any other key the
  file carries are preserved as found, and so is every per-entry key on a
  pre-existing direct entry (`type`, `source`, `signatures`): pinning sets
  `version` and nothing else; a git entry already pinned to a tag or commit
  is not rewritten. An entry the scaffold cannot reach is left as found and
  reported rather than dropped, with the stated consequence that
  ansible-lint, which installs the manifest itself, stays red on that host.
- The skill gains a network dependency on Galaxy at scaffold time. Offline,
  a greenfield run writes the unpinned manifest; a retrofit leaves the
  existing file untouched rather than downgrade a lock. Both report that
  pinning was skipped.
- On a retrofit, rewriting a pre-existing `requirements.yml` is a change to
  pre-existing content and falls under the ADR 0005 ask.
