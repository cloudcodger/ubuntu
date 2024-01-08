Role `cloud_init_reboot`
========================

Check for the completion of Cloud-Init and reboot the system if required.

After the first boot of a system configured using Cloud-Init, there is a one time execution of the equivelant to what `apt-get upgrade` does. It does not however reboot the system when it is required from upgrading packages.

Using a flag file to make sure the reboot is only performed on the first execution of the role. This waits for the Cloud-Init process to complete (which can take a whiile on that first boot), then checks for `/var/run/reboot-required` and reboots the system when the file exists.

If a reboot is not required, the flag file is still created and this role will not perform a reboot. After the first time the role has run, the flag file prevents any subsequent reboot, even if `/var/run/reboot-required` exists.

Caution: Adding this role to a group of hosts that have been configured and running for some time may cause them all to be rebooted at the same time and could cause any services they provide to be unavailable during the reboots. It is recommended to call this role before any other roles in a playbook and to run the playbook right after the system has been created. Other use cases may not provide any benefit.

Role Variables
--------------

### All variables.

- `cloud_init_reboot_flag_file` - The name of the flag file used to make sure the reboot only happens once (default: `"{{ ansible_facts.env.PWD }}/.ansible/cloud_init_reboot_check"`).

Example Playbook
----------------

```yaml
---
- name: Reboot system after initial install via cloud-init.
  become: true
  hosts: host_group

  roles:
    - role: cloudcodger.ubuntu.cloud_init_reboot
```
