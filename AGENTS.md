# ansible-role-supervisord

Ansible role that installs Supervisord and generates its process-control configs on EL7 hosts.

## Architecture in a paragraph

Supervisord needs a main config plus one `[program:x]` block per managed process, and
hand-editing those across a fleet doesn't scale — this role generates both from a single
`supervisord_programs` list variable. It follows the standard single-role layout:
`tasks/main.yml` installs the `supervisor` package, renders `/etc/supervisord.conf` from
`templates/supervisord.conf.j2`, then loops `supervisord_programs` through
`templates/program.ini.j2` to write one `.ini` file per program under `supervisord_conf_d`,
notifying the `restart supervisord` handler on change. `meta/main.yml` declares no Galaxy role
dependencies and targets EL 7 only; the README's mention of `nexcess.repo-epel`/`nexcess.server`
is an undeclared prerequisite (EPEL supplies the `supervisor` package on EL), not a Galaxy
dependency, so a consuming playbook must add it itself. This is a leaf role with no dependents
inside the repo — it's pulled into host-level playbooks that need supervised background
processes.

## File map

```
defaults/
  main.yml              # role variables: package name, config paths, resource limits, supervisord_programs list
handlers/
  main.yml              # restart/reload/start/stop supervisord, notified by task changes
meta/
  main.yml              # Galaxy metadata: EL7 only, min_ansible_version 2.0, no declared dependencies
tasks/
  main.yml              # entry point: install package, render main + per-program configs, enable service at boot (does not start it)
templates/
  program.ini.j2        # one [program:x] block per supervisord_programs entry
  supervisord.conf.j2   # main supervisord.conf, populated from defaults/main.yml
README.md
```

## Commands

No CI, lint config, or Molecule tests are defined in this repo. Verify a change by running it
against a playbook that includes the role:

```
ansible-playbook --syntax-check -i <inventory> <playbook>.yml
ansible-playbook -i <inventory> <playbook>.yml --check --diff
```

## Conventions

- `README.md` states the license as `MIT / BSD`; `meta/main.yml` says `MIT`. Likely an unedited
  `ansible-galaxy role init` placeholder rather than a real dual-license — confirm before relying
  on either.
- Tasks use the legacy `key="value"` quoted-argument style throughout (e.g.
  `name="{{ supervisord_package }}"`), not the modern YAML-mapping form. Match it in
  `tasks/main.yml` and `handlers/main.yml` rather than mixing styles in the same file.
- Each `supervisord_programs` item maps to a conditional block in `program.ini.j2`. Adding a new
  per-program option means adding both the `{% if %}` block there and the key to
  `defaults/main.yml`/README.
- `meta/main.yml`'s `dependencies: []` is the source of truth for Galaxy dependencies — the
  README's EPEL/`nexcess.server` note is an operational prerequisite, not something to add there.
- `templates/supervisord.conf.j2`'s `[include]` path (`files = supervisord.d/*.ini`) is hardcoded,
  not templated from `supervisord_conf_d`. If you change `supervisord_conf_d`, update this
  template's include path too, or the generated per-program `.ini` files won't be loaded.

## See also

- [README.md](README.md) — install and usage
- [CONTRIBUTING.md](CONTRIBUTING.md) — PR process
