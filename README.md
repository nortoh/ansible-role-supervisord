# Ansible Role: Supervisord

Hand-editing Supervisord's main config and per-program `.ini` files across a fleet doesn't
scale. This role generates both from a single `supervisord_programs` variable and restarts the
service when they change.

## Requirements

- EL 7 (per `meta/main.yml`)
- EPEL enabled first — via `nexcess.repo-epel` or `nexcess.server` — so the `supervisor` package
  is installable. This isn't a Galaxy dependency (`meta/main.yml` declares none); add it to your
  playbook yourself.

## Role Variables

See [`defaults/main.yml`](defaults/main.yml) for the full list. The one that matters is
`supervisord_programs`, a list of program definitions that drives every generated
`[program:x]` block in [`templates/program.ini.j2`](templates/program.ini.j2).

| Variable | Default | Purpose |
|---|---|---|
| `supervisord_package` | `supervisor` | Package installed |
| `supervisord_conf_d` | `/etc/supervisord.d` | Where per-program `.ini` files land — see caveat below |
| `supervisord_minfds` | `1024` | `minfds` in the main config |
| `supervisord_minprocs` | `200` | `minprocs` in the main config |
| `supervisord_umask` | `022` | `umask` in the main config |
| `supervisord_directory` | `/tmp` | `directory` in the main config |
| `supervisord_programs` | `[]` | Programs to configure — see `program.ini.j2` for supported keys |

> **Caveat:** `templates/supervisord.conf.j2`'s `[include]` section hardcodes `files =
> supervisord.d/*.ini` (relative to `/etc`) rather than templating `supervisord_conf_d`. If you
> override `supervisord_conf_d` away from its default `/etc/supervisord.d`, per-program `.ini`
> files land in the new path but supervisord never loads them.

## Add to Requirements

```yaml
- src: https://github.com/nexcess/ansible-role-supervisord.git
  name: nexcess.supervisord
```

## Example Playbook

```yaml
- hosts: server
  roles:
    - nexcess.supervisord
  vars:
    supervisord_programs:
      - name: myapp
        command: /usr/bin/myapp --run
        numprocs: 1
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for the PR process. Architecture, the file layout, and
verification commands live in [AGENTS.md](AGENTS.md).

## License

MIT / BSD
