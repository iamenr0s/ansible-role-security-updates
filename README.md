[![Molecule](https://github.com/iamenr0s/ansible-role-security-updates/actions/workflows/molecule.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-security-updates/actions/workflows/molecule.yml) ![Ansible Role](https://img.shields.io/ansible/role/d/iamenr0s/security_updates) [![CodeFactor](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-security-updates/badge)](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-security-updates)

# Ansible Role: Security Updates

This role audits and enables unattended security updates on Debian/Ubuntu, RHEL/AlmaLinux/RockyLinux and Fedora servers, and can optionally run a security-only update once. It never reboots: it reports `linux_hardening_updates_reboot_required` and leaves the reboot to your playbook.

Features
--------
- Audit mode by default: read-only, and reports every deviation. Changes need `linux_hardening_confirm_remediation: true`.
- Configures `dnf-automatic` / `dnf5-automatic` on the RedHat family and `unattended-upgrades` on the Debian family, and enables their timers.
- Security-only updates by default; `linux_hardening_updates_apply: false` only downloads and notifies.
- Optional one-shot security update (`linux_hardening_updates_apply_now`), gated on free disk space.
- Backs up the config file before changing it.
- Fails fast on unsupported OS families and gathers the facts it needs when the play does not.
- The only role in the `iamenr0s.linux_hardening` collection that manages automatic updates; for a full upgrade of all packages plus reboot handling, keep using `ansible-role-upgrade`.

## What it enforces
| Family | Mechanism | Settings |
|---|---|---|
| RedHat (EL 8-10) | `dnf-automatic` + `dnf-automatic.timer` (`dnf-plugins-core` for the reboot check) | `upgrade_type = security`, `apply_updates = yes` in `/etc/dnf/automatic.conf` |
| Fedora | `dnf5-plugin-automatic` + `dnf5-automatic.timer` (`dnf5-plugins` for the reboot check) | same keys |
| Debian, Ubuntu | `unattended-upgrades`, `apt-daily*.timer` | drop-in `/etc/apt/apt.conf.d/52linux-hardening-updates`: package lists daily, unattended upgrade on, `Automatic-Reboot "false"` |

On the Debian family the allowed origins come from the distro's `50unattended-upgrades` and are **not overridden**; any allowed origin without "security" in its name is reported (not fixed) when `linux_hardening_updates_security_only` is true.

## Variable names
Variables use the collection-wide `linux_hardening_*` prefix (shared with sibling roles) rather than `security_updates_*`. Full list: `defaults/main.yml` and `meta/argument_specs.yml`.

## Modes
`audit` (default) is read-only and sets `linux_hardening_updates_findings` (`key, expected, actual, fixable`) and `linux_hardening_updates_reboot_required`. `remediate` needs `linux_hardening_confirm_remediation: true`. After remediation only fixable findings fail the run.

## Safety
- `linux_hardening_updates_apply_now: true` (remediate only) runs `dnf upgrade --security` or `unattended-upgrade -v` once, after checking that `/`, `/var` and `/boot` are below `linux_hardening_updates_max_filesystem_usage` percent.
- Once enabled, packages change without a run of your playbook, and services restart when their libraries are updated. Use `linux_hardening_updates_apply: false` to only download and notify.
- The role never reboots; `linux_hardening_updates_automatic_reboot` (Debian family only) is `false` by default.
- `apply_now` is not idempotent: a later run may find new updates. Everything else is (Molecule checks idempotence).
- Check mode: `--check` previews the configuration change; `apply_now` and the re-audit are skipped.

## Example
```yaml
- hosts: all
  become: true
  roles:
    - role: iamenr0s.security_updates
      vars:
        linux_hardening_mode: remediate
        linux_hardening_confirm_remediation: true
  post_tasks:
    - name: Reboot if the updates need it
      ansible.builtin.reboot:
      when: linux_hardening_updates_reboot_required | bool
```

## Verification
Molecule `default` (remediate without gathered facts, idempotence, re-audit, settings read back), `check` (`--check` on an unconfigured host installs nothing), `audit`, `refuse`: AlmaLinux 9, Debian 13, Ubuntu 24.04, Fedora 44. `apply` (real security-only update): AlmaLinux 9, Debian 13, Fedora 44. CI matrix: AlmaLinux 8–10, Rocky 8–10, Fedora 42–44, Debian 12–13, Ubuntu 22.04/24.04.

## Rollback
RedHat: set `apply_updates = no` or disable the timer. Debian family: delete the drop-in and disable `apt-daily-upgrade.timer`. Installed updates are not rolled back.

## Known gaps
- Containers have no real timer firing or reboot; the timers are checked as enabled, not as having run.
- No package holds/versionlocks (the old role had them), no uptime or load gates, no reboot execution.
- dnf-automatic cannot reboot, so `automatic_reboot` applies to the Debian family only.
- Pending-update counts are not reported. Not tested: EL 8 and 10, Rocky, CentOS Stream, Debian 12, Ubuntu 22.04. CIS IDs not mapped.

CI & Release (maintainers)
----------------------------
A single workflow (`.github/workflows/molecule.yml`) runs lint and the full Molecule distro matrix on pushes to `main`, PRs, and `v*` tags. On `v*` tags, a `release` job publishes to Ansible Galaxy after all tests pass.

One-time setup:

```bash
# Galaxy publishing key (environment-scoped, get it from galaxy.ansible.com/ui/token)
gh secret set GALAXY_API_KEY --repo iamenr0s/ansible-role-security-updates

# Code scanning notifications (Slack webhook URL; for Discord append /slack to the webhook URL)
gh secret set SECURITY_ALERT_WEBHOOK --repo iamenr0s/ansible-role-security-updates
```

`.github/workflows/code-scanning-notify.yml` polls the code-scanning API every 6 hours and posts new or updated open alerts to that webhook (GitHub Actions cannot trigger on `code_scanning_alert` directly).

To release: tag a commit `vX.Y.Z` and push the tag — CI gates the Galaxy publish.

## Contributing

Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md) for the
local pipeline commands and pull request checklist. This project follows the
[Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md).

## Security

See [SECURITY.md](SECURITY.md) — GitHub private vulnerability reporting, no
public issues for security bugs.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and release notes.

## License

This project is licensed under the [MIT License](LICENSE).

## Author Information

Author: iamenr0s

Galaxy: `iamenr0s.security_updates`
