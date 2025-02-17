# Ansible Collection - cloudcodger.ubuntu

The 'ubuntu` collection contains roles written specifically for the Ubuntu OS.

These roles may or may not work on other Debian based operating systems.

# Roles

See the individual `README.md` files under each role for more information.

- `apt-mirror` - Configures a host to run the `apt-mirror` application.
- `cloud_init_reboot` - Use with Cloud-Init systems for an initial reboot.
- `initial_apt_update` - Performs an initial `apt-get upgrade` and reboot.
- `keepalived` - Install and configure `keepalived`.
- `minio` - Install and configure [MinIO](https://min.io/).
- `netboot_xyz` - Configure a system to be a TFTP server with netboot.xyz files.
- `nginx` - Install NGINX and configure it for various purposes.

The `cloud_init_reboot` role differs from `initial_apt_update` in that it waits for the `cloud-init` to finish before doing anything else and it does not perform the `apt-get upgrade`, since `cloud-init` performs that during its execution.
