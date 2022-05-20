# Ansible-Project

**This role is part of the [ITC Automated Build Pods Project][]**
  - Installs Ansible AWX service

## Dependencies

This role can be run stand-alone to set up an AWX server, however it is intended to be used as part of the [ITC Automated Build Pods Project][] which requires additional Ansible roles and configurations.

[Docker-Ansible][] role is needed to deploy the AWX service containers.

## Requirements

The AWX installation requires a Docker container runtime environment.  The [Docker-Ansible][] role will install and configure this for you.

## Role Variables

### defaults/main.yml
```yaml
awx_rabbitmq_version: "3.7.4"
awx_rabbitmq_image: "ansible/awx_rabbitmq:{{ awx_rabbitmq_version }}"
awx_rabbitmq_default_vhost: "awx"
awx_rabbitmq_erlang_cookie: "cookiemonster"
awx_rabbitmq_host: "rabbitmq"
awx_rabbitmq_port: "5672"
awx_rabbitmq_user: "guest"
awx_rabbitmq_password: "guest"

awx_postgresql_version: "10"
awx_postgresql_image: "postgres:{{ awx_postgresql_version }}"


awx_memcached_image: "memcached"
awx_memcached_version: "alpine"
awx_memcached_host: "memcached"
awx_memcached_port: "11211"

awx_dockerhub_base: ansible
awx_dockerhub_version: 9.0.0
awx_create_preload_data: True

awx_project_data_dir: /var/lib/awx
awx_postgres_data_dir: /var/lib/pgawx
awx_host_port: 8080

awx_docker_compose_dir: /var/lib/awxcompose
awx_pg_username: awx
awx_pg_password: awxpass
awx_pg_admin_password: postgrespass
awx_pg_database: awx
awx_pg_port: 5432

awx_secret_key: awxsecret
awx_admin_user: admin
awx_admin_password: password

awx_web_url: "http://{{ ansible_default_ipv4.address }}:{{ awx_host_port }}"
awx_web_hostname: awxweb
awx_task_hostname: awx

```

## Example Playbook
```yaml
- name: "Deploy Foreman Server"
  hosts: buildhost
  remote_user: root
  vars_files:
    - vault.yml
  tasks:

    - name: Wait for server to come online
      wait_for_connection:
        delay: 60
        sleep: 10
        connect_timeout: 5
        timeout: 900

    - include_role:
        name: common
      tags:
        - common

    - include_role:
        name: isc_dhcp_server
        public: true
        apply:
          tags:
            - dhcp
      when: foreman_proxy_dhcp
      tags:
        - dhcp

    - include_role:
        name: tftp
        public: true
        apply:
          tags:
            - tftp
      when: foreman_proxy_tftp
      tags:
        - tftp

    - include_role:
        name: nginx
        public: true
        apply:
          tags:
            - nginx
      tags:
        - nginx

    - include_role:
        name: awx
        public: true
        apply:
          tags:
            - awx
      tags:
        - awx

    - include_role:
        name: docker
        public: true
        apply:
          tags:
            - docker
      tags:
        - docker

    - include_role:
        name: awx
        tasks_from: container-tasks.yml
        public: true
        apply:
          tags:
            - awx
      tags:
        - awx

    - include_role:
        name: foreman
        public: true
      tags:
        - install
        - configure
        - foreman
        - smartproxy
        - customize

    - include_role:
        name: ansible-project
        public: true
      tags:
        - never
        - project
        - projectimport
        - projectclone
```

## License

MIT

## Author Information

Created by Alan Janis

[itc automated build pods project]:  https://github.com/ajanis/itc-build-pods.git
[docker-ansible]: https://github.com/ajanis/docker-ansible.git
