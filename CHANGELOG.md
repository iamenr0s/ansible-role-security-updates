# Changelog

## Unreleased
First release; becomes 1.0.0 when `v1.0.0` is tagged.
- Split out of the iamenr0s.linux_hardening collection.
- Check mode: `--check` previews remediation on an unconfigured host; `apply_now` and the re-audit are skipped. New Molecule `check` scenario.
- Fail fast on OS families other than RedHat and Debian; gather the needed facts when the play sets `gather_facts: false`.
- Install `dnf-plugins-core` / `dnf5-plugins` so `dnf needs-restarting` works, and report them as findings when missing (a missing plugin used to read as "no reboot required").
- Validate `linux_hardening_updates_reboot_time` as HH:MM when automatic reboot is on.
- Platform variables moved to `vars/RedHat.yml` and `vars/Fedora.yml`; internal variables use the `__linux_hardening_updates_` prefix; sub-task names carry the file prefix (`name[prefix]` enabled).
- Audit checks are written as data (one `ok` + `finding` per check) instead of Jinja loops; findings are a native list.
- One finding per kind of problem: `packages`, `apt timers` and `allowed origins` list what is wrong in `actual`, instead of one finding per package, timer or origin. Finding fields are unchanged; finding keys changed.
- Desired settings are defined once in `vars/main.yml` and shared by configure, audit and the apt template.
- `apply_now` on the RedHat family uses `ansible.builtin.dnf` (`security: true`, `update_only: true`) instead of `dnf upgrade --security`.
- Removed `linux_hardening_updates_dropin_name`; the apt drop-in is always `/etc/apt/apt.conf.d/52linux-hardening-updates`. A value set for it is now ignored.
- gitleaks pre-commit hook, also run in CI: the lint job now runs `pre-commit run --all-files`.
- No collection dependencies: `dnf automatic` is configured with `ansible.builtin.lineinfile` instead of `community.general.ini_file`, which current community.general releases can no longer run on EL 8 (Python 3.6 targets).
- Debian family installs with `ansible.builtin.apt`; RedHat drops the undocumented `update_cache` passthrough of `ansible.builtin.package`.
