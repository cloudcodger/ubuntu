Role `nginx`
============

Install NGINX and configure it from the following optional items.

- Disable the `default` site, installed with the package
- Internal Only site
- Custom sites
- Load Balancer sites
- Cloud-Config sites

If none of these options are desired, using `ansible.builtin.apt` to install `nginx` would be faster.

Requirements
------------

No special requirements.

Variables
---------

- `nginx_base` - The base directory to hold all site files (default: `/var/www`).
- `nginx_disable_default_site` - Disable the `default` site, installed with the NGINX package, which removes the `/etc/nginx/sites-enabled/default` sym-link (default: `true`).

Internal Site
=============

The role can create an "internal only site" named after the hostname and with an `index.html` that contains a message indicating the site is for internal use only. The default messsage is "This site is not intended to be accessed via a web browser."

This is useful for a web server where the base URL for the host is not used and only other host names (for example from a CNAME record) are the primary use case of the server.

Internal Site Variables
-----------------------

- `nginx_internal_site_apply` - Create the Internal Site or not,  (default: `false`).
- `nginx_internal_site_is_default_site` - Should the Internal Site be the `default` site or not. (default: `true`).
- `nginx_internal_site_message` - Change the default message for the Internal Site inside `index.html` (default: see above).
- `nginx_internal_site_server_index` - the value for `index` in the site file (default: `"index.html index.htm"`).
- `nginx_internal_site_server_name` - the value for `server_name` in the site file (default: `_`, which is the catch-all server).

Custom Sites
============

For creating custom sites that contain a single `server` directive. The expectation is that every site file will only contain one `server` section and multiple sites will be used otherwise.

Unlike some other parts of this role, the `server` does not include a `listen` or `server_name` and these need to be included in the `directives`.

Custom Variables
----------------

- `nginx_custom_sites` - a list of custom sites to create (default: `[]`).

Each list item must contain the following keys:

- `name` - the site name.
- `directives` - a multi-line string of the lines for the `server` level settings.
- `locations` - a list of `location` sections.
  - `uri` - the uri for the location (default: `/`).
  - `contents` - a multi-line string of the lines for the `location`.

Each list item may contain the following optinonal keys:

- `upstreams` - a list of `upstream` sections.
  - `name` - required for each upstream.
  - `contents` - a multi-line string of the lines for the `upstream`.

Load Balancer Sites
===================

The role can create load balancer sites for NGINX to be configured as a reverse proxy. This only configures one `location /` and may not work for more complex site configurations.

Load Balancer Variables
-----------------------

- `nginx_balancer_sites` - a list of load balancer sites to create (default: `[]`).

Each list item must contain the following keys:

- `name` - the site name.
- `backends` - a list of backend servers and ports.

Each list item may contain the following optinonal keys:

- `disable_chunked_encoding` - Set `chunked_transfer_encoding off;` (default: no).
- `headers` - a list of `proxy_set_header` values to add (default: don't add any).
- `http_version` - the `proxy_http_version` value (default: don't add).
- `listen_port` - the port on which to listen (default: `80`).
- `server_name` - the `server_name` for the site (default: the system hostname).

Cloud-Config
============

Prior to Ubuntu 20.04 a `preseed` File was used for unattended installs, and is out of scope for this role.

Starting with Ubuntu 20.04 `cloud-init` must be used for unattended installs. How to use the `autoinstall` feature requires a webserver that can server up two HTML pages, `meta-data` and `user-data`. The role can create these for one or more sites.

When installing Ubuntu inside a VM on [Proxmox PVE](https://proxmox.com/), a much better approach is to use the `cloudcodger.proxmox_client.cloud_init` Ansible Role, found in [Ansible Galaxy](https://galaxy.ansible.com/ui/).

See the [Introduction to autoinstall](https://canonical-subiquity.readthedocs-hosted.com/en/latest/intro-to-autoinstall.html) for a better understanding of how all this works. The details of which are out of the scope of this readme.

Cloud-Config variables
----------------------

- `nginx_cloud_config_sites` - a list of key/values pairs for the NGINX sites to configure (default: `[]`).
- `nginx_user_data` - a multi-line string that gets added as the `user_data:` lines in the `user-data` html file (default: not added).
- `nginx_user_data_ssh_allow_pw` - sets `allow-pw` under the `ssh` section (default: `true`).
- `nginx_user_data_ssh_authorized_keys` - a nulti-line string of public SSH keys to be placed in the `root` `authorized_keys` file (default: not added).
- `nginx_user_data_late_commands` - a multi-line string that gets added as the `late-commands` section (default: see `defaults/main.yml`).
- `nginx_user_data_network:` - a multi-line string that gets added as the `network` section (default: see `defaults/main.yml`).
- `nginx_user_data_packages` - a nulti-line string of packages tthat gets added as the `packages` section (default: not added).
- `nginx_user_data_storage_config` - a multi-line string that gets added as the `config` section under the `storage` section (default: see `defaults/main.yml`).

Dependencies
------------

No dependencies.

Example Playbook
----------------

```yml
---
- name: |
    Install NGINX, remove the default site and configure
    reverse proxy load balancers for loki, and mimir.
  become: true
  hosts: nginx_servers # must be two hosts with one in the `keepalived_primary` group.

  vars:
    nginx_balancer_sites:
      - name: loki
        backends:
          - "backend1:3100"
          - "backend2:3100"
          - "backend3:3100"
        headers:
          - 'Connection ""'
          - Host $http_host
          - X-Scope-OrgID default
          - X-Forwarded-For $proxy_add_x_forwarded_for
          - X-Forwarded-Proto $scheme
          - X-Real-IP $remote_addr
        listen_port: "3100"
        server_name: "_"
      - name: mimir
        backends:
          - "backend1:8080"
          - "backend2:8080"
          - "backend3:8080"
        headers:
          - 'Connection ""'
          - Host $http_host
          - X-Scope-OrgID default
          - X-Forwarded-For $proxy_add_x_forwarded_for
          - X-Forwarded-Proto $scheme
          - X-Real-IP $remote_addr
        listen_port: "9009"
        server_name: "_"

  roles:
    - role: cloudcodger.ubuntu.nginx
```

```yml
---
- name: |
    Install NGINX and configure it using 'custom_sites' as a reverse
    proxy load balancer for MinIO and the MinIO UI.

  roles:
    - role: cloudcodger.ubuntu.nginx
      nginx_custom_sites:
        - name: minio
          directives: |
            listen 80 default_server;
            listen [::]:80 default_server;
            server_name "_";

            # Allow special characters in headers
            ignore_invalid_headers off;
            # Allow any size file to be uploaded.
            # Set to a value such as 1000m; to restrict file size to a specific value
            client_max_body_size 0;
            # Disable buffering
            proxy_buffering off;
            proxy_request_buffering off;
          upstreams:
            - name: minio_s3
              contents: |
                least_conn;
                server minio1:9000;
                server minio2:9000;
                server minio3:9000;
            - name: minio_console
              contents: |
                least_conn;
                server minio1:9001;
                server minio2:9001;
                server minio3:9001;
          locations:
            - contents: |
                proxy_set_header Host $http_host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;

                proxy_connect_timeout 300;
                # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
                proxy_http_version 1.1;
                proxy_set_header Connection "";
                chunked_transfer_encoding off;

                proxy_pass http://minio_s3; # This uses the upstream directive definition to load balance
            - uri: /minio/ui/
              contents: |
                rewrite ^/minio/ui/(.*) /$1 break;
                proxy_set_header Host $http_host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
                proxy_set_header X-NginX-Proxy true;

                # This is necessary to pass the correct IP to be hashed
                real_ip_header X-Real-IP;

                proxy_connect_timeout 300;

                # To support websockets in MinIO versions released after January 2023
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";

                chunked_transfer_encoding off;

                proxy_pass http://minio_console; # This uses the upstream directive definition to load balance

```

```yml
---
- name: |
    Install NGINX and configure it to server out 'meta-data' and 'user-data'
    for an Ubuntu Unattended Install.
    Overriding the URI for the jammy site.

  roles:
    - role: cloudcodger.ubuntu.nginx
      nginx_cloud_config_sites:
        - name: focal
          server_name: prime20 prime20.lab.codger.site
        - name: jammy
          server_name: prime22 prime22.lab.codger.site
          uri: http://prime7/ubuntu
      nginx_user_data_ssh_authorized_keys: |-
        - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJwg1ybdTSPF2xFm+TS3bjelKO7VQgGpbyvr+b9qJBzr cloudcodger@showcase
        - {{ lookup('file', lookup('env', 'HOME') + '/.ssh/showcase.pub') }}
      nginx_cloud_conf_user_data_packages: |-
        - htop
        - tcpdump
        - top
```
