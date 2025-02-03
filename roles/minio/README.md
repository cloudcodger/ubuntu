Role `minio`
============

Install [MinIO](https://min.io/) on servers and configure it.

The recommendation is to place the data directory on XFS formatted file systems. This role can take a device name and make sure it is formatted as XFS and mounted on `/data`.

Requirements
------------

Either `minio_volume_hosts` or `minio_volumes` must be specified.

Role Variables
--------------

- `minio_data_base_dir`
  - The base directory under which the data directory will be created.
  - Default: `/data`.
- `minio_data_dir`
  - The data directory.
  - Default: `"{{ minio_data_base_dir }}/minio"`.
- `minio_data_drive`
  - An optional disk drive (for example: `/dev/sdb`) to be formatted as an XFS filesystem and mounted at `minio_data_base_dir`.
- `minio_opts`
  - The `defaults` value for MINIO_OPTS.
  - Explicitly sets the MinIO Console listen address to port `9001` on all network interfaces.
  - (default: `"--console-address :9001"`).
- `minio_root_user`
  - The `defaults` value for MINIO_ROOT_USER.
  - Default: `admin`.
- `minio_system_user`
  - The Linux system user for MinIO.
  - Default: `minio-user`.
- `minio_volume_hosts`
  - The host portion of `minio_volumes`.
  - Must be specified if `minio_volumes` was not.
  - Example: `minio{1...3}.example.com`.
  - Default: none.
- `minio_volumes`
  - The `defaults` value for MINIO_VOLUMES.
  - Must be specified if `minio_volume_hosts` was not.
  - Default: `http://{{ minio_volume_hosts }}:9000/data/minio`.

Dependencies
------------

No special dependencies.

Example Playbook
----------------

```yml
---
- name: Install MinIO and configure it.
  become: true
  hosts: minio_servers

  vars:
    minio_data_drive: /dev/sdb
    minio_root_password: "{{ lookup('file', '~/.secrets/minio/root_password') }}"
    minio_volume_hosts: "minio{1..3}.example.com"

  roles:
    - role: cloudcodger.ubuntu.keepalived
```
