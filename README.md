ansible-lxd
=========
[![Build Status](https://img.shields.io/travis/hispanico/ansible-lxd.svg?style=flat-square)](https://travis-ci.org/hispanico/ansible-lxd)
[![Galaxy](https://img.shields.io/badge/galaxy-hispanico.lxd-blue.svg?style=flat-square)](https://galaxy.ansible.com/hispanico/lxd/)

This role manages LXD Profiles and LXD Containers on a remote LXD Server. https://linuxcontainers.org/lxd/


Requirements
------------

* LXD 2.0 or greater must be installed on the LXD host and ansible server (it should already be installed by default on Ubuntu 16.04)

* LXD should already be setup on the remote host using `sudo lxd init`

* In order for Ansible to manage an LXD host remotely the following commands must be run ahead of time:

On the remote LXD host:

```
$ lxc config set core.https_address [::]:8443
$ lxc config set core.trust_password replace-this-with-a-secure-password
```

On the Ansible host:

```
$ lxc config set core.https_address [::]:8443
```

```
$ lxc remote add lxd4 lxd4.example.com
```
(replace lxd4.example.com with the hostname of your LXD host, 'lxd4' can be named whatever you want, you'll need to reference it in the inventory file)

* Tested on an LXD host and Ansible host both using Ubuntu 16.04 LTS (may work with other distros)

Role Variables
--------------

LXD Profile variables:
* `url:` unix:/var/snap/lxd/common/lxd/unix.socket (default)
* `state:` present (default), absent

LXD Containers variables:
* `url:` unix:/var/snap/lxd/common/lxd/unix.socket (default)
* `state:` started (default), stopped, restarted, absent, frozen
* `type:` image (default)
* `mode:` pull (default)
* `server:` https://cloud-images.ubuntu.com/releases (default)
* `protocol:` simplestreams (default)
* `alias:` "16.04" (default)
* `architecture:` x86_64 (default)
* `wait_for_ipv4_addresses:` true (default)
* `timeout:` 600 (default)

These variables are documented here:
* http://docs.ansible.com/ansible/latest/lxd_profile_module.html
* http://docs.ansible.com/ansible/latest/lxd_container_module.html

Additional Variables:
* `cloud_init:` cloud-init config (http://cloudinit.readthedocs.io/en/latest/index.html)

Default values:

```yaml
# LXD Profiles
lxd_profiles:
  - name: default
    description: "Default LXD Profile"
    status: present

# LXD Containers
LXCs:
  - name: mycontainer
    mode: pull
    server: https://cloud-images.ubuntu.com/releases
    alias: "16.04"
    protocol: simplestreams
    architecture: x86_64
    profile: default
    status: started

```

Dependencies
------------

None


Example
----------------

The following example will configure a default LXD profile with the cloud-init option to install python packages and configure ansible user then install 2 containers on the LXD server.

```yaml
  - hosts: all
    roles:
      - ansible-lxd
    vars:
      lxd_profiles:
        - name: default
          description: "Default LXD Profile"
          storage_pool: default
          network: lxdbr0
          cloud_init: |
            #cloud-config
            packages:
              - python
            users:
              - name: ansible
                group: sudo
                shell: /bin/bash
                sudo: ALL=(ALL) NOPASSWD:ALL
                ssh_authorized_keys:
                  - {{ lookup('file', '$HOME/.ssh/id_rsa.pub' ) }}
          status: present

      LXCs:
        - name: MyContainer01
          mode: pull
          server: https://cloud-images.ubuntu.com/releases
          alias: "16.04"
          protocol: simplestreams
          architecture: x86_64
          disk_space: 50GB          # Container's Disk limit
          memory: 2GB               # Container's Memory limit
          storage_pool: default
          network: lxdbr0
          ipv4address: 10.0.0.2     # static IPv4 address according with lxdbr0 subnet
          profile: default
          status: started

        - name: MyContainer02
          mode: pull
          server: https://cloud-images.ubuntu.com/releases
          alias: "16.04"
          protocol: simplestreams
          architecture: x86_64
          disk_space: 50GB          # Container's Disk limit
          memory: 2GB               # Container's Memory limit
          nesting: "true"           # Container's security.nesting enabled
          storage_pool: default
          network: lxdbr0
          ipv4address: 10.0.0.3     # static IPv4 address according with lxdbr0 subnet
          profile: default
          status: started
```


License
-------

Licensed under the GPLv3 License. See the LICENSE file for details.

Author Information
------------------

Hispanico
