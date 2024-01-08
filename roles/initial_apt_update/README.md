Role `initial_apt_update`
=========================

Perform an initial `apt-get upgrade` and reboot the system if required.

Using a flag file to make sure the reboot is only performed on the first execution of the role. This does the same thing as an initial execution of `apt-get update && apt-get dist-upgrade`, then checks for `/var/run/reboot-required` and reboots the system when the file exists.

If a reboot is not required, the flag file is still created and this role will not perform a reboot. After the first time the role has run, the flag file prevents any subsequent reboot, even if `/var/run/reboot-required` exists.

Caution: Adding this role to a group of hosts that have been configured and running for some time may cause them all to be rebooted at the same time and could cause any services they provide to be unavailable during the reboots. It is recommended to call this role before any other roles in a playbook and to run the playbook right after the system has been created. Other use cases may not provide any benefit.

Role Variables
--------------

- `initial_apt_update_reboot_flag_file` - The flag file used to prevent multiple executions (default `"{{ ansible_facts.env.PWD }}/.ansible/initial_apt_update_flag"`).

Example Playbook
----------------

```yml
---
- name: Update APT packages on Hosts once and only once.
  become: true
  hosts: new_server

  roles:
    - role: cloudcodger.ubuntu.initial_apt_update
```
