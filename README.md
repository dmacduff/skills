# skills

Agent skills by [Douglas MacDuff](https://github.com/dmacduff). Install into
any project:

```bash
npx skills@latest add dmacduff/skills
```

## scaffold-ansible-env

Stands up a reproducible Ansible **environment** (not project structure) in
the current directory:

- **uv owns Python**: pinned interpreter, project-managed venv,
  `ansible-core` and lint tooling as locked dependencies.
- **One authoritative lockfile, two derived exports**: `uv.lock` is primary;
  a pip-compatible `requirements.txt` and a PEP 751 `pylock.toml` are
  regenerated from it and never hand-edited. The `requirements.txt` is what
  makes the result installable on a stock box that has never heard of uv.
- **A collection lock**: `requirements.yml` pins the full resolved
  collection graph, transitives included, to exact versions. Galaxy has no
  lockfile, so this file is it. See
  [ADR 0004](docs/adr/0004-collection-lock.md).
- **pre-commit from the first commit**: `ansible-lint`, `yamllint` (tuned to
  meet ansible-lint's requirements), ruff, and the stock hygiene hooks.
- **A decided execution-environment question**: a four-criterion rubric
  (shared control node, platform mismatch, multiple runners, unattended runs)
  applied as *infer → ask → default-no*. When it fires, the EE is one extra
  file, because `ansible-builder` consumes exactly the `requirements.txt` and
  `requirements.yml` the scaffold already emitted. When it doesn't, the
  rubric lands in your README so opting in later is deliberate, not
  archaeology.

It never emits `ansible.cfg`, inventory, roles, or playbooks
([ADR 0003](docs/adr/0003-environment-project-boundary.md)). On a retrofit it
asks once before pinning an existing `requirements.yml` or mechanically
normalizing pre-existing files (whitespace, line endings, `---` markers);
semantic ansible-lint findings it inherits are baselined in
`.ansible-lint-ignore`, never edited, so the repo ends committable
([ADR 0005](docs/adr/0005-retrofit-ask-normalize-baseline.md)).

### Questions it may ask

Every question is inferred from your request first. Put the answer in the
prompt and it is never asked.

| Question | Fires when | Answer it up front with |
|---|---|---|
| Normalize pre-existing files and pin the existing `requirements.yml`? | Retrofit only | "normalize existing files" / "leave existing files alone" |
| EE criterion 1: is the control node shared or someone else's? | Not inferable from request or repo | "runs on our shared jump host" / "runs on my laptop" |
| EE criterion 2: do collections need system-level deps the control node lacks? | Not inferable; a collection with system deps makes it true on its own | "control node has libssh" / name the collections |
| EE criterion 3: does more than one person or pipeline run it? | Not inferable | "team of four runs this" / "just me" |
| EE criterion 4: is it scheduled or unattended? | Not inferable | "from cron" / "interactive only" |
| Run `ansible-builder build` as a check? | EE scaffolded, container runtime present, image not explicitly requested | "build the image" / "skip the build" |

### Example

```
> Set up the environment for this Ansible project. It'll run unattended
> from cron on our shared jump host.

[survey] greenfield; cisco.ios collection requested; no Python source planned
[uv]     pinned Python, added ansible-core; dev: ansible-lint yamllint pre-commit
[lock]   uv.lock written; exported requirements.txt, pylock.toml
[hooks]  .pre-commit-config.yaml, .yamllint, .gitignore; all-files run: passed
[galaxy] requirements.yml: cisco.ios 11.5.1; transitive ansible.netcommon 8.6.2, ansible.utils 6.1.0
[EE]     criteria 1 (shared host) and 4 (unattended) true → execution-environment.yml
[readme] recreate-the-env, derived-artifact rule, EE decision recorded
```

## Design notes

Vocabulary in [CONTEXT.md](CONTEXT.md); decisions in [docs/adr/](docs/adr/).
Skills here are developed with Claude Code: designed in recorded grill
sessions (the ADRs are their output), with every change reviewed by me
before it lands.

## License

[MIT](LICENSE)
