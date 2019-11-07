# Ansible-Project

**This role is part of the [ITC Automated Build Pods Project][]**
  - Installs Ansible AWX service

[TOC]

## Dependencies

This role can be run stand-alone to set up an AWX server, however it is intended to be used as part of the [ITC Automated Build Pods Project][] which requires additional Ansible roles and configurations.

[Docker-Ansible][] role is needed to deploy the AWX service containers.

## Requirements

The AWX installation requires a Docker container runtime environment.  The [Docker-Ansible][] role will install and configure this for you.

## Role Variables

### defaults/main.yml
```yaml
rabbitmq_version: "3.7.4"
rabbitmq_image: "ansible/awx_rabbitmq:{{rabbitmq_version}}"
rabbitmq_default_vhost: "awx"
rabbitmq_erlang_cookie: "cookiemonster"
rabbitmq_host: "rabbitmq"
rabbitmq_port: "5672"
rabbitmq_user: "guest"
rabbitmq_password: "guest"

postgresql_version: "10"
postgresql_image: "postgres:{{postgresql_version}}"


memcached_image: "memcached"
memcached_version: "alpine"
memcached_host: "memcached"
memcached_port: "11211"

dockerhub_base: ansible
dockerhub_version: 9.0.0
create_preload_data: True

project_data_dir: /var/lib/awx
postgres_data_dir: /var/lib/pgawx
host_port: 8080

docker_compose_dir: /var/lib/awxcompose
pg_username: awx
pg_password: awxpass
pg_admin_password: postgrespass
pg_database: awx
pg_port: 5432

secret_key: awxsecret
admin_user: admin
admin_password: password

awx_web_hostname: awxweb
awx_task_hostname: awx

```

## Example Playbook
```yaml
- name: "Deploy Build-Pod Environment"
  hosts: all
  remote_user: root
  tasks:

    - include_role:
        name: common
      tags:
        - common
        - packages
        - cloudinit
        - ssh
        - sol
        - sudo
        - autoupdate
        - hostfile
        - timezone

    - include_role:
        name: isc_dhcp_server
        public: yes
      when: foreman_proxy_dhcp

    - include_role:
        name: tftp
        public: yes
      when: foreman_proxy_tftp

    - include_role:
        name: nginx
        public: yes

    - include_role:
        name: awx
        public: yes

    - include_role:
        name: docker
        public: yes

    - include_role:
        name: awx
        tasks_from: update_ca.yml
        public: yes

    - include_role:
        name: foreman
        public: yes
      tags:
        - install
        - configure
        - foreman
        - smartproxy
```

## License

MIT

## Author Information

Created by Alan Janis

[itc automated build pods project]:  https://github.com/ajanis/itc-build-pods.git
