# Contributing

Hand-editing Supervisord's main config and per-program `.ini` files across a fleet doesn't
scale — this role generates both instead.

## Development setup

See [AGENTS.md](AGENTS.md#commands) for verification commands — not repeated here.

## Opening a PR

Use this repo's [PR template](.github/PULL_REQUEST_TEMPLATE.md) — link the ticket (or state why
there isn't one) and describe how the change was verified.

## Before you open a PR

- [ ] Verified manually against a playbook that includes the role — no automated test suite
      exists (see [AGENTS.md](AGENTS.md#commands))
- [ ] Lint/static analysis passes, if you ran one locally (none is wired into this repo)
- [ ] Docs (`README.md`/`AGENTS.md`) updated if this changes role variables, templates, or
      behavior
