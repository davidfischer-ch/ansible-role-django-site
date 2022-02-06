# Ansible Role Django Site

Library of Ansible plugins and roles for deploying various services.
See [ansible-roles](https://github.com/davidfischer-ch/ansible-roles) for additional documentation.

This repository hosts the role Django Site and may depend of other roles and plugins of the library.

## Status

Not (yet) implemented:

* Role actions such as configure, ...
* Mounting the data directory. I think this roles should not take this responsibility. When deploying more than one web host, make sure your PlayBook mount the data directory prior to calling this role.

## Dependencies

See [meta](meta/main.yml).

## Variables

TODO VARIABLES

## Example

### Code Repository

Structure:

```
website$ tree -L 3
.
├── config.yml
└── src
    └── myproject
        ├── myproject
        ├── contact
        ├── locale
        ├── manage.py
        ├── news
        ├── offices
        ├── pages
        ├── publications
        ├── requirements.install.pip2
        └── templates

8 directories, 3 files
```

Settings are templates that will be rendered with the Ansible `template` module:

```
website$ tree src/myproject/myproject/settings/
src/myproject/myproject/settings/
├── base.py.j2
├── develop.py.j2
├── __init__.py.j2
└── production.py.j2

0 directories, 4 files
```

Automation configuration file `config.yml`:

```
configuration_templates:
  - settings/__init__.py
  - settings/base.py
  - settings/develop.py
  - settings/production.py
custom_applications:
  - contact
  - news
  - offices
  - pages
  - publications
python_version: '3.6'
source_path: src/myproject
```

### PlayBook

```
---

# Management

- hosts:
    - localhost
  roles:
    - register-ssh-hosts

# Common Services

- hosts:
    - all:!localhost
  roles:
    - bootstrap
    - fail2ban
    - ufw
    - miscellaneous
    - kernel
    - python
    - mounts
    - rsync
  vars:
    rsync_version: 3.1.3  # 10/12/2108

# Application Services

- hosts:
    - my-site-cache-group
  roles:
    - redis

- hosts:
    - my-site-db-group
  roles:
    - postgresql

- hosts:
    - my-site-storage-group
  roles:
    - nfs

# TODO: Mount shared storage

- hosts:
    - my-site-web-group
  roles:
    - django-site
    - nginx
    - uwsgi
  vars:
    supervisor_daemon_mode: systemv
    supervisor_username: kikouman
    supervisor_password: kikoupass
    supervisor_version: 3.3.4  # 10/12/2018

    djsite_role_action: setup

    djsite_app_directory: '/var/app/{{ djsite_instance_name }}'
    djsite_bower_enabled: yes
    djsite_celery_workers:
      default:
        config_file: example.celery.worker.conf.j2
        name: default
        queues:
          - default
        type: worker
      beat:
        config_file: example.celery.worker.conf.j2
        name: beat
        type: beat
    djsite_compress_enabled: yes
    djsite_compress_offline: yes
    djsite_coverage_minimum: 62
    djsite_coverage_options:
      - --branch
      - --include=myapp/*
      - --omit=*/migrations/*,*/tests/*
    djsite_database_host: 127.0.0.1
    djsite_database_name: myapp
    djsite_database_port: '{{ postgresql_port|int }}'
    djsite_database_user: admin
    djsite_database_password: dbpass
    djsite_debug_enabled: yes
    djsite_domains:
      - www.myapp.net
      - '{{ ansible_host }}'
      - 127.0.0.1
    djsite_instance_name: myapp
    djsite_max_releases: 3
    djsite_project: MyApp
    djsite_release_mode: normal
    djsite_repository_url: git://some-repo.git
    djsite_repository_version: master
    djsite_secret_key: this-is-not-a-good-secret-key
    djsite_settings_module: development
    djsite_ssh_key_private: "{{ lookup('env', 'HOME') }}/.ssh/id_rsa"
    djsite_ssh_key_public: "{{ lookup('env', 'HOME') }}/.ssh/id_rsa.pub"
    djsite_standalone: yes
    djsite_superuser:
      name: super.man
      email: super@myapp.net
      password: superpass
    djsite_test_options: --failfast --noinput

    nginx_daemon_mode: supervisor
    nginx_version: release-1.13.6  # 19/11/2017
    nginx_sites:
      site:
        name: '{{ djsite_instance_name }}'
        config_file: '{{ roles_directory }}/django-site/templates/example.nginx.config.conf.j2'
        debug: '{{ djsite_debug_enabled|bool }}'
        domains: '{{ djsite_domains }}'
        max_body_size: 100k
        redirect_ssl: yes
        with_dhparam: yes
        with_ssl: yes

    uwsgi_apps:
      application:
        name: '{{ djsite_instance_name }}'
        config_file: '{{ roles_directory }}/django-site/templates/example.uwsgi.app.xml.j2'
        user: '{{ djsite_daemon_user }}'
        group: '{{ djsite_daemon_group }}'
        path: '{{ djsite_app_directory }}/production'
        project: '{{ djsite_project }}'
        python_version: python
        limit_as: 2048
```

If you want to replace uWSGI by Uvicorn, simply disable uWSGI role and configuration, and add:

```
    djsite_process_config_file: example.uvicorn.app.conf.j2
    djsite_process_count: '{{ 2 * ansible_processor_count }}'
    djsite_process_enabled: yes
    djsite_process_options:
      - --interface asgi3
      - --log-level warning
```

If you need to execute additional tasks on certain events, just do:

```
djsite_configure_tasks_file: '{{ playbook_dir }}/../app_tasks/configure-hook.yml'
djsite_update_tasks_file: '{{ playbook_dir }}/../app_tasks/update-hook.yml'
djsite_virtualenv_tasks_file: '{{ playbook_dir }}/../app_tasks/virtualenv-hook.yml'
```

## License

See [LICENSE.rst](LICENSE.rst).

## Authors

See [AUTHORS](AUTHORS).

2014-2022 - David Fischer
