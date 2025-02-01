Role `keepalived`
=================

Configure two or more systems with `keepalived`. Best practice is to do this with two systems where one is the primary and the others are secondary.

Requirements
------------

- the number of play hosts must be two or more.
- the ansible group `keepalived_primary` must be defined and contain 1 host.
- a VIP must be specified and must be an IP address.

Role Variables
--------------

- `keepalived_password` - the password to use (default: a randomly generated string of 8 characters).
- `keepalived_vip` - A required IP address for the VIP shared by the hosts (default: none).
- `keepalived_virtual_id` - the `virtual_router_id` (default: the last octet of the VIP). If the VIP is IPv6, the default will not be calculated properly and must instead be specified. It must also be within the range of 0 to 255.

Dependencies
------------

No special dependencies.

Example Playbook
----------------

```yml
---
- name: Install keepalived and configure it.
  become: true
  hosts: keepalived_servers # must contain at least two hosts with one in the `keepalived_primary` group.

  vars:
    keepalived_vip: 192.168.1.100

  roles:
    - role: cloudcodger.ubuntu.keepalived
```
