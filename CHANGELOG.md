# Changelog

## Unreleased
- Check mode: `--check` previews remediation on an unconfigured host; `apply_now` and the re-audit are skipped. New Molecule `check` scenario.
- Fail fast on OS families other than RedHat and Debian; gather the needed facts when the play sets `gather_facts: false`.
- Install `dnf-plugins-core` / `dnf5-plugins` so `dnf needs-restarting` works, and report them as findings when missing (a missing plugin used to read as "no reboot required").
- Validate `linux_hardening_updates_reboot_time` as HH:MM when automatic reboot is on.
- Platform variables moved to `vars/RedHat.yml` and `vars/Fedora.yml`; internal variables use the `__linux_hardening_updates_` prefix; sub-task names carry the file prefix (`name[prefix]` enabled).
- gitleaks pre-commit hook; `community.general >= 9.0.0`.

## 1.0.0
- Split out of the iamenr0s.linux_hardening collection; no functional change.
