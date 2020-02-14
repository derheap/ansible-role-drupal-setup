# ansible-role-drupal-setup

This role sets up files and folders for a Drupal 8 site – with a folder structure analog to Capistrano.

The folder structure is the prerequiste for a deployment of the Drupal site with Ansible.

## Folder structure

To enable deployments and configuration updates without affecting the running Drupal 8 site, a folder structure analog to [Capistrano](https://capistranorb.com/documentation/getting-started/structure/) is used.

~~~~
example.com/
├── current -> /srv/www/example.com/releases/20200214101735_master
├── releases
│   ├── 20200214095316_dummy
│   ├── 20200214095847_master
│   └── 20200214101735_master
└── shared
    └── sites
        └── default
            ├── files
            └── settings.php
~~~~

The shared directory holds the data to be shared with all releases. Mainly the `site`directory
The `site` directory is later symlinked into the current release.

### Permissions

All files are owned by the user in `drupal_core_owner` and by the group set in `drupal_deploy_group`. The files are writable by the group. All user who have the right to deploy must be in  that group.

Only `sites/default/files` is writable by the the webserver. The group is set to `drupal_server_group`.

The Git repository is writable for all members in the group defined in `drupal_deploy_group`.

### Settings.php

A `settting.php` file with the database connection is copied in place. The database itself is not created. If you want to use an newer or modified version of the `settings.php` you must provide one in the `templates` directory of your playbook. The template name is `settings.php.j2`.

### Dummy release

The Ansible role sets up a dummy release with a HTML file in `web`, so you can test the webserver without deploying a Drupal site.

## Prerequisites

* The user running the playbook should be in the `sudo` group.
* If you init a Git repository with this role, Git must be installed.

## Role Variables

When possible, the role uses variable names that are also used by other roles (mostly from Geerlingguy) to setup the server and database.

~~~~~
# Var from geerlingguy/ansible-role-drupal
drupal_domain: example.com

# Variables specific for this role
drupal_trusted_host_pattern: 'example\.com'
drupal_base_dir: "/srv/www/{{ drupal_domain }}"
drupal_config_folder: config/sync
drupal_deploy_create_repo: true
drupal_deploy_group: drupal
drupal_deploy_name: mydrupalproject
drupal_server_group: "www-data"
# To enable compatibility with some old installations
drupal_public_folder: "web"

# Vars from geerlingguy/ansible-role-drupal
drupal_deploy_repo: "/srv/repos/{{ drupal_deploy_name }}.git"
drupal_deploy_version: master
drupal_deploy_dir: "{{ drupal_base_dir }}/current"
drupal_core_path: "{{ drupal_deploy_dir }}/{{ drupal_public_folder }}"
drupal_core_owner: "{{ ansible_ssh_user | default(ansible_env.SUDO_USER, true) | default(ansible_env.USER, true) | default(ansible_user_id) }}"
drupal_core_owner_become: true

drupal_db_user: drupal
drupal_db_password: drupal
drupal_db_name: drupal
drupal_db_backend: mysql
drupal_db_host: "127.0.0.1"

drupal_install_profile : standard
~~~~~

## Example Playbook

~~~~
---
- hosts: myserver
  gather_facts: True
  become: true
  remote_user: myuser

  vars_files:
  - vars/drupal.yml

  roles:
    - { role: geerlingguy.git }
    - { role: derheap.drupal-setup }
~~~~

## Role Defaults

~~~~
# Var from geerlingguy/ansible-role-drupal
drupal_domain: example.com

# Variables specific for this role
# drupal_trusted_host_pattern: # Not set
drupal_base_dir: "/srv/www/{{ drupal_domain }}"
drupal_config_folder: config/sync
drupal_deploy_create_repo: true
drupal_deploy_group: drupal
drupal_deploy_name: mydrupalproject
drupal_server_group: "www-data"
# To enable compatibility with some old installations
drupal_public_folder: "web"

# Vars from geerlingguy/ansible-role-drupal
drupal_deploy_repo: "/srv/repos/{{ drupal_deploy_name }}.git"
drupal_deploy_version: master
drupal_deploy_dir: "{{ drupal_base_dir }}/current"
drupal_core_path: "{{ drupal_deploy_dir }}/{{ drupal_public_folder }}"
drupal_core_owner: "{{ ansible_ssh_user | default(ansible_env.SUDO_USER, true) | default(ansible_env.USER, true) | default(ansible_user_id) }}"
drupal_core_owner_become: true

# drupal_db_user: # Not set
# drupal_db_password: # Not set
# drupal_db_name: # Not set
drupal_db_backend: mysql
drupal_db_host: "127.0.0.1"

drupal_install_profile : standard
~~~~
