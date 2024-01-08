Role `netboot_xyz`
==================

Configure a system to be a TFTP server providing the netboot.xyz files.

Requirements
------------

No special requirements.

Role Variables
--------------

- `netboot_xyz_tftp_base_dir` - Base directory in which to place the files served by TFTP (default `/srv/tftp`).
- `netboot_xyz_files` - List of file names and URLs to download into the base directory (default are two files).

See `defaults/main.yml` file for full default values. The default installs `nnetboot.xyz-arm64.efi`, `netboot.xyz.efi` and `netboot.xyz.kpxe` that use built-in iPXE NIC drivers from `https://boot.netboot.xyz/ipxe/` so they are served out by the TFTP server.

Example Playbook
----------------

```yml
---
- name: Configure TFTP server Hosts for netboot_xyz.
  become: true
  hosts: netboot_xyz_host

  roles:
    - role: cloudcodger.ubuntu.netboot_xyz
```
