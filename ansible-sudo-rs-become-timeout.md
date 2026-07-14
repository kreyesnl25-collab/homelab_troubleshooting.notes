# Fix: Ansible "Timed out waiting for become success" on Ubuntu (sudo-rs conflict)

## Problem

Running any Ansible playbook or ad-hoc command with privilege escalation (`become: true` / `-b`) failed with:

[ERROR]: Task failed: Timed out waiting for become success or become password prompt.
>>> Standard Error
[sudo: [sudo via ansible, key=...] password:] Password:
fatal: [localhost]: UNREACHABLE! => {"changed": false, "msg": "Task failed: Timed out waiting for become success or become password prompt...", "unreachable": true}

This happened whether the password was supplied via `--ask-become-pass` (interactive prompt) or via `-e "ansible_become_password=..."` (extra var). Sudo itself worked fine outside of Ansible (`sudo whoami` returned `root` with no issue).

## Environment

- OS: Ubuntu Server (VMware VM)
- Ansible: `ansible-core 2.20.1`
- Python: `3.14.4`
- Connection type: local (running against `localhost`)

## Diagnosis Steps

Ruled out in order:

1. **Wrong password** — confirmed correct via direct `sudo whoami` test (worked, returned `root`).
2. **`requiretty` in sudoers** — checked with `sudo grep -i requiretty /etc/sudoers /etc/sudoers.d/*`; not present.
3. **Ansible-level password passing** — tested with `-e "ansible_become_password=..."` to bypass the interactive prompt entirely; still timed out, ruling out prompt/typing issues.
4. **Verbose debug (`-vvv`)** — showed Ansible correctly building and issuing the sudo command, ruling out command construction as the cause.

## Root Cause

Ubuntu had `sudo-rs` (a Rust reimplementation of sudo) set as the default `sudo` binary via `update-alternatives`. Sudo-rs handles the password prompt/response cycle differently enough from traditional sudo that Ansible's become mechanism hangs indefinitely waiting for a signal that never arrives in the expected form.

## Fix

Switch the system default back to traditional sudo:

sudo update-alternatives --config sudo

Select the option pointing to `/usr/bin/sudo.ws` (traditional sudo) rather than `/usr/lib/cargo/bin/sudo` (sudo-rs).

Verify the fix:
ansible localhost -b -m command -a "whoami" -e "ansible_become_password=YOURPASSWORD"

Expected output: `root`

## Prevention / Notes

- System updates may reset `update-alternatives` back to `sudo-rs` as the default — if this resurfaces after an update, check `sudo update-alternatives --display sudo` first.
- Never leave real passwords in `-e "ansible_become_password=..."` beyond one-off local debugging.
- This is a general local-connection/become quirk, not specific to any one playbook.
