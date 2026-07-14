My First Ansible Playbook

#milestone for a aspiring Network Technician.

What this is

This marks the first Ansible playbook I wrote and successfully ran end-to-end — configuring a local Linux VM by installing and starting nginx. It's a small task, but it exercises the full basic Ansible workflow: inventory, privilege escalation, package management, and service management.

Goal---

Use Ansible to automatically:


Install the nginx web server
Ensure the nginx service is running


...on a target host (in this case, localhost), instead of doing it manually.

Setup


Inventory file: inventory.ini
Playbook file: site.yml
Command used to run it:


bash  ansible-playbook -i inventory.ini site.yml --ask-become-pass

(--ask-become-pass prompts for the sudo password needed to install/start services)

What the playbook does

The playbook (site.yml) targets the local VM and runs three tasks:


Gather Facts — Ansible's automatic first step, collecting system info about the target (OS, IP, hardware, etc.) so later tasks can use it if needed.
Install nginx — uses Ansible's package module to install the nginx web server if it isn't already present.
Start nginx — ensures the nginx service is running (and would keep it running/enabled on future runs, since Ansible tasks are idempotent — running the same playbook again won't break or duplicate anything).


Result

PLAY RECAP
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


ok=3 — all three tasks completed successfully
changed=1 — nginx installation actually made a change (it wasn't installed before); the other two tasks (Gather Facts, Start nginx) found nothing needed changing, which is expected and correct behavior for idempotent automation


Confirmed working by checking:

bashsudo systemctl status nginx

and loading http://localhost in a browser, which showed the nginx welcome page.

What I learned


-How Ansible inventories and playbooks are structured at a basic level
-What privilege escalation (become) means and how Ansible handles it
-That Ansible tasks are meant to be idempotent — designed to be safely re-run without unintended side effects
-Hit and diagnosed a real environment bug along the way (see: ansible-sudo-rs-become-timeout.md in this repo) before getting this playbook working


Next steps


Expand this playbook to configure more than just nginx (e.g. firewall rules, a custom index page, additional packages)
Move from a single flat playbook toward using Ansible roles for better structure
Start tying this into a proper lab environment (EVE-NG/GNS3/Containerlab) instead of just localhost
