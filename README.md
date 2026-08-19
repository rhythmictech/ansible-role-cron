# Ansible Role: cron

[![CI](https://github.com/rhythmictech/ansible-role-cron/actions/workflows/ci.yml/badge.svg)](https://github.com/rhythmictech/ansible-role-cron/actions/workflows/ci.yml)
[![Ansible Galaxy](https://img.shields.io/badge/Ansible%20Galaxy-rhythmictech.cron-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/rhythmictech/cron/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

Manages scheduled jobs on a host using either classic **cron** entries (written
to `/etc/cron.d`) or **systemd timers** (with their backing service units).
Jobs can be defined once in shared group vars and selectively applied per
environment, and jobs you no longer want can be declaratively removed.

## Requirements

- Ansible (ansible-core) **2.17 or higher**.
- A systemd-based host for the timer features; the cron features work on any
  host with cron. Tested platforms: EL 9 / EL 10 (Rocky, Alma, and other RHEL
  rebuilds), Amazon Linux 2023, Fedora, and Debian 12 / 13.

The role's tasks require root; run the role with `become: true` (set it on the
play or role include — the role does not set `become` itself).

## Installation

```bash
ansible-galaxy role install rhythmictech.cron
```

Or in a `requirements.yml`:

```yaml
roles:
  - name: rhythmictech.cron
```

## The `env` variable

Crontabs and timers are commonly defined once in shared group vars but should
only be applied in some environments. Each item may carry an `envs` list, and
the role applies it only when the playbook-supplied `env` value is in that list:

```yaml
when: env in item.envs | default(cron_default_envs)
```

- `env` is **set by the playbook** (typically from inventory/group vars or
  `--extra-vars`) and must be defined when the role runs. It is not defaulted.
- If an item omits `envs`, it falls back to `cron_default_envs`
  (`['prod', 'ops', 'test', 'dev']` by default), so it applies in every
  recognized environment.

Note: removal tasks (`crontabs_remove`, `systemd_timers_remove`) are **not**
filtered by `env` — removals always run so an item can be cleaned up regardless
of the environment that created it.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `env` | _(unset, required)_ | Current environment name; gates which items apply. |
| `cron_default_envs` | `['prod','ops','test','dev']` | Envs an item applies to when its own `envs` is omitted. |
| `crontabs` | `[]` | Cron jobs to create. |
| `crontabs_remove` | `[]` | Cron jobs to remove. |
| `systemd_timers` | `[]` | systemd timers (and backing services) to create. |
| `systemd_timers_remove` | `[]` | systemd timers to remove. |

### `crontabs` item

| Key | Required | Description |
|-----|----------|-------------|
| `name` | yes | Unique name (used as the `/etc/cron.d/cron-<name>` file and comment marker). |
| `command` | yes | Command to run. |
| `user` | yes | User the job runs as. |
| `minute` / `hour` / `day` / `month` / `weekday` | no | Cron schedule fields (default `*` via `omit`). |
| `envs` | no | Environments this job applies to. |

### `systemd_timers` item

| Key | Required | Description |
|-----|----------|-------------|
| `name` | yes | Unit base name; creates `<name>.timer` and `<name>.service`. |
| `command` | yes | `ExecStart` for the backing service. |
| `description` | no | Unit description (defaults to `name`). |
| `user` | no | Service `User=` (default `root`). |
| `service_type` | no | Service `Type=` (default `simple`). |
| `envvars` | no | List of `{name, value}` rendered as `Environment=`. |
| `on_calendar` | no | **Raw** `OnCalendar=` expression (e.g. `daily`, `Mon *-*-* 00:00:00`). Overrides the component fields below. |
| `weekday`/`year`/`month`/`day`/`hour`/`minute`/`second` | no | Components used to build `OnCalendar=` when `on_calendar` is not set. |
| `persistent` | no | Sets `Persistent=` (run missed jobs after downtime). |
| `randomized_delay_sec` | no | Sets `RandomizedDelaySec=`. |
| `accuracy_sec` | no | Sets `AccuracySec=`. |
| `service_unit_custom_directives` | no | Extra raw lines in the service `[Unit]` section. |
| `service_custom_directives` | no | Extra raw lines in the service `[Service]` section. |
| `timer_unit_custom_directives` | no | Extra raw lines in the timer `[Unit]` section. |
| `timer_custom_directives` | no | Extra raw lines in the timer `[Timer]` section. |
| `envs` | no | Environments this timer applies to. |

`<name>_remove` items only need a `name`.

## Example Playbook

```yaml
- hosts: all
  become: true
  vars:
    env: prod
  roles:
    - role: rhythmictech.cron
      vars:
        crontabs:
          - name: nightly-backup
            user: root
            command: /usr/local/bin/backup.sh
            hour: "2"
            minute: "30"
            envs: ['prod', 'ops']

        systemd_timers:
          - name: cleanup
            command: /usr/local/bin/cleanup.sh
            description: Daily temp cleanup
            on_calendar: daily
            persistent: true
            randomized_delay_sec: 300

        crontabs_remove:
          - name: old-job

        systemd_timers_remove:
          - name: legacy-timer
```

## Local Development & Testing

The role is tested with [Molecule](https://ansible.readthedocs.io/projects/molecule/)
against systemd-enabled containers for each supported platform, and linted with
[ansible-lint](https://ansible.readthedocs.io/projects/lint/) (production
profile). CI runs both on every pull request.

```bash
pip install ansible-core ansible-lint molecule "molecule-plugins[docker]"

ansible-lint          # lint (includes yamllint)
molecule test         # full test cycle: create, converge, idempotence, verify, destroy

# Test a specific distro (default: rockylinux9)
MOLECULE_DISTRO=debian13 molecule test
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines.

## License

[MIT](LICENSE).

## About Rhythmic

This role is maintained by [Rhythmic Technologies](https://www.rhythmictech.com),
an AWS-focused managed services provider. We're always looking for good people,
practices, and tools — issues and pull requests are welcome.
