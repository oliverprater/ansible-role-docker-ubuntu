docker-ubuntu
=============

Installs Docker CE or EE according to the [Docker v17.06 Ubuntu guide](https://docs.docker.com/engine/installation/linux/docker-ee/ubuntu/) on:
- Xenial 16.04 (LTS)
- Trusty 14.04 (LTS)

As the Docker ecosystem moves fast the role exclusively supports the two latest Ubuntu LTS versions.

> Please confirm for yourself that the Ansible role still follows the best practices in the Docker installation guide!

Example Plays
-------------

To install Docker CE and manage the host with Ansible:

    - hosts: all
      roles:
         - role: oliverprater.docker-ubuntu

The role provisions `python` and `pip` in accordance with the `ansible_python_interpreter`.

The pip packages `docker` and `docker-compose` ensure that users of this role can use Ansible to build Docker images and manage Docker containers.
This behavior can be configured, but the role supports out of the box:

    - hosts: server
      roles:
         - role: oliverprater.docker-ubuntu
           docker_version: "17.06" # fix docker version for idempotent install behavior
      tasks:
        docker_service:
          project_src: hello_world # assumes a docker-compose file is present
          build: yes

The default behavior on Ubuntu 14.04 is to install AuFS support for Docker.
To override the default configuration as well as use the older `devicemapper` storage engine:

    - hosts: server
      roles:
         - role: oliverprater.docker-ubuntu
           docker_aufs_enabled: no
           docker_config:
             storage-driver: devicemapper

The example above uses the flexible `docker_config` variable to write an `/etc/docker/daemon.json` as described in
[the dockerd reference](https://docs.docker.com/engine/reference/commandline/dockerd/#linux-configuration-file).
The advantage of the flexibe `docker_config` variable becomes apparrent, when creating a more advanced setup:

    - hosts: server
      vars:
        port_dockerd: 2376
        tls_cacert_dest: "~/.docker/ca.pem"
        tls_cert_dest: "~/.docker/cert.pem"
        tls_key_dest: "~/.docker/key.pem"
      roles:
         - role: oliverprater.docker-ubuntu
           docker_version: "17.06" # fix docker version for idempotent install behavior
           docker_opts: "-H tcp://{{ ansible_default_ipv4['address'] }}:{{ port_dockerd }}"
           docker_config:
             tls: yes
             tlscacert: "{{ tls_cacert_dest }}"
             tlscert: "{{ tls_cert_dest }}"
             tlskey: "{{ tls_key_dest }}"
             tlsverify: yes

This advanced example has docker daemon listen on port 2376 of the host machine via `docker_opts` and configures Docker TLS usage via `docker_config`.
The example makes several assumptions about the host machine setup, which are out of the scope of this roles configuration space.

> IMPORTANT: When the `docker_config` contains the keyword `hosts` then `-H fd://` is removed from the docker.service file!

Role Variables
--------------

Available variables are listed below for default values see `defaults/main.yml` as well:

### `docker_apt_cache_valid_time`

Global setting for Ansible's `cache_valid_time`, which determines how long apt cache is valid.

### `docker_aufs_enabled`

Determines whether to install AuFS driver on Ubuntu 14.04 LTS.

### `docker_apt_key_url` and `docker_apt_key_sig`

The official key information from the Docker guide.

### `docker_apt_release_channel` and `docker_architecture`

Set the release channel and host architecture seperately:

- `docker_apt_release_channel`: **stable** supports 'edge' and 'testing'
- `docker_architecture`: **amd64** supports amd64, armhf, and s390x

### `docker_apt_repository`

Sets the apt repository URL according to the information of `docker_apt_release_channel`, `docker_architecture`, and Ansible's `set_facts` module.
Set to a specific repository URL to override the automatic URL generation.

> Override will ignore `docker_apt_release_channel` and `docker_architecture`

### `docker_edition` and `docker_version`

Set the Docker edition and version seperately:

- `docker_edition`: **ce** supports 'ce' and 'ee'
- `docker_version`: **latest** supports any string e.g. '17.03'

### `docker_package`

Sets the docker package name according to `docker_edition` and `docker_version`, which is sent to the Ansible `apt` module.

> Override will ignore `docker_edition` and `docker_version`

### `docker_config` and `docker_opts`

Please see [the docs for dockerd](https://docs.docker.com/engine/reference/commandline/dockerd/) for a full list of available options.

- The dict `docker_config` is written to `/etc/docker/daemon.json`.
- The string `docker_opts` sets the `$DOCKER_OPTS` environment variable.

The options set in the configuration file via `docker_config` must not conflict with options set via flags in `docker_opts`.

> The docker daemon fails to start if an option is duplicated between the file and the flags, regardless their value.

The failure to start the daemon due to duplication between the file and the flags is especially prevalent for the `hosts` keyword in the `daemon.json` file.
It is possible to configure the entire daemon via `daemon.json`, but remember to also include the UNIX socket as a host:

```
  docker_config:
    hosts: ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]
```

> IMPORTANT: When the `docker_config` contains the keyword `hosts` then `-H fd://` is removed from the docker.service file!

Requirements
------------

Requires python-pycurl for apt modules.

Dependencies
------------

None.

License
-------

MIT

Author Information
------------------

The motivation for this role came from [angstwad.docker_ubuntu](https://github.com/angstwad/docker.ubuntu), as well as [geerlingguy.docker](https://github.com/geerlingguy/ansible-role-docker) and [mongrelion.docker](https://github.com/mongrelion/ansible-role-docker).

This role was created in 2017 by Oliver Prater.
