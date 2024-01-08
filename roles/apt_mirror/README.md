Role `apt_mirror`
=================

Install and configure `apt-mirror` to be an APT repository mirror.

As of the time this role was created, the `apt-mirror` script installed from the Ubuntu APT repository has a bug in it and the software is without a current developer working on it. A patched version of the Perl script is located in the `files` directory and can be installed by setting `apt_mirror_install_patched_executable: true`. If the software is patched at some future date, simply remove that and allow the default of `false` to no longer install that version of the script.

To update the mirror contents, run the following commands.

```bash
# When logged in as root, like often done with an LXC container
su apt-mirror -c apt-mirror
/apt-mirror/var/clean.sh
# When logged in as a user with sudo permissions
sudo -u apt-mirror apt-mirror
sudo /apt-mirror/var/clean.sh
```

Requirements
------------

The `apt_mirror_base_path` must already exist and be on a file system with enough free space to hold all the APT repositories listed in `apt_mirror_list`. This can very in size, but as of the time this role was created, the default list it taking `522 GB` of space. Make sure there is also enough free space for updates to download new files before removing old ones.

Role Variables
--------------

### Required variables.

The default values for the following variables will very likely not be desired or possibly even work.

At a minimum, set the following to desired values.

`bind9_servers_zones`

### All variables.

- `apt_mirror_base_path` - The set `base_path` value in the `mirror.list` file (default `/apt-mirror`).
- `apt_mirror_install_patched_executable` - When set to `true`, install the included patched script over top of the one installed from the APT repository (default `false`).
- `apt_mirror_list` - A list of lines to add in the `mirror.list` file (default is a list of `deb` packages).
- `apt_mirror_nginx_add_site` - When true, NGINX will be installed and a site file created (default `false`).
- `apt_mirror_nginx_default_indexes` - A list of index files under `apt_mirror_nginx_server_root` to be removed (default `[ index.nginx-debian.html ]`).
- `apt_mirror_nginx_default_server` - Adds `default_server` to the `listen` directives in the site file (default `true`).
- `apt_mirror_nginx_disable_defaults` - When true, will disable the `default` site installed by NGINX (default `false`).
- `apt_mirror_nginx_locations` - A list of location/alias blocks to add to the site file (default contains `/ubuntu` and `/ubuntu-ports`).
- `apt_mirror_nginx_server_index` - The `index` directive value for the site (default `"index.html index.htm"`).
- `apt_mirror_nginx_server_name` - The `server_name` directive value for the site (default `_`).
  The `_` value is an NGINX catch_all value.
- `apt_mirror_nginx_server_root` - The `root` directive value for the site (default `/var/www/html`).
- `apt_mirror_nginx_site_name` - The name of the `sites-available` and `sites-enabled` files (default `primer`).
- `apt_mirror_nthreads` - The `threads` value in the `mirror.list` file (default `40`).

Example Playbook
----------------


```yaml
---
- name: Configure 'apt-mirror' with patch and NGINX for its use.
  become: true
  hosts: host_group

  roles:
    - role: cloudcodger.ubuntu.apt_mirror
      apt_mirror_install_patched_executable: true
      apt_mirror_nginx_disable_defaults: true
      apt_mirror_nginx_add_site: true
```
