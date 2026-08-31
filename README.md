# skills

Agent skills by [Douglas MacDuff](https://github.com/dmacduff). Install into
any project:

```bash
npx skills@latest add dmacduff/skills
```

## scaffold-ansible-env

Stands up a reproducible Ansible **environment** — not project structure — in
the current directory:

- **uv owns Python**: pinned interpreter, project-managed venv,
  `ansible-core` and lint tooling as locked dependencies.
- **One authoritative lockfile, two derived exports**: `uv.lock` is primary;
  a pip-compatible `requirements.txt` and a PEP 751 `pylock.toml` are
  regenerated from it and never hand-edited. The `requirements.txt` is what
  makes the result installable on a stock box that has never heard of uv.
- **pre-commit from the first commit**: `ansible-lint`, `yamllint` (tuned to
  not fight ansible-lint), ruff, and the stock hygiene hooks.
- **A decided execution-environment question**: a four-criterion rubric
  (shared control node, platform mismatch, multiple runners, unattended runs)
  applied as *infer → ask → default-no*. When it fires, the EE is one extra
  file — `ansible-builder` consumes exactly the `requirements.txt` and
  `requirements.yml` the scaffold already emitted. When it doesn't, the
  rubric lands in your README so opting in later is deliberate, not
  archaeology.

It never emits `ansible.cfg`, inventory, roles, or playbooks, and never
touches existing Ansible content — see
[ADR 0003](docs/adr/0003-environment-project-boundary.md).

### Example

```
> Set up the environment for this Ansible project. It'll run unattended
> from cron on our shared jump host.

[survey] greenfield; cisco.ios collection requested; no Python source planned
[uv]     pinned Python, added ansible-core; dev: ansible-lint yamllint pre-commit
[lock]   uv.lock written; exported requirements.txt, pylock.toml
[galaxy] requirements.yml: cisco.ios
[hooks]  .pre-commit-config.yaml, .yamllint, .gitignore; all-files run: passed
[EE]     criteria 1 (shared host) and 4 (unattended) true → execution-environment.yml
[readme] recreate-the-env, derived-artifact rule, EE decision recorded
```

## Design notes

Vocabulary in [CONTEXT.md](CONTEXT.md); decisions in [docs/adr/](docs/adr/).

## License

[MIT](LICENSE)
