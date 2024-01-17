# Ansible role "papanito.ttrss"  <!-- omit in toc -->

[![Ansible Role](https://img.shields.io/ansible/role/d/papanito/ttrss)](https://galaxy.ansible.com/papanito/ttrss) [![GitHub issues](https://img.shields.io/github/issues/papanito/ansible-role-ttrss)](https://github.com/papanito/ansible-role-ttrss/issues) [![GitHub pull requests](https://img.shields.io/github/issues-pr/papanito/ansible-role-ttrss)](https://github.com/papanito/ansible-role-ttrss/pulls)

Installs an dedicated instance of Tiny Tiny RSS and all it's dependencies (php, PostgreSQL, Lighttpd).

Some restriction/caveats:

- The role does currently install and use lighttpd
- The role is not yet intended to install side-by-side of another app
- Very limited lighttpd configuration e.g. no ssl}

I recommend to use the role on a dedicate cloud instance - see below for an example on how I use it.

## Requirements

TTRSS requires now PHP > 7.4

Also be [aware of this](https://tt-rss.org/wiki/UpdatingFeeds):

> Host installations are not supported starting 2021.

## Role Variables

### ttrss-specifc

These variables are specific for the tt-rss config incl. lighttpd

|Variable|Description|Default Value|
|--------|-----------|-------------|
| `ttrss_hostname` | Hostname of the url on which the site can be reached | `{{ ansible_default_ipv4.address }}` |
| `ttrss_url_prefix` | url-prefix, mainly used for default value of `ttrss_url` | `https://` |
| `ttrss_url` | `SELF_URL_PATH` Full URL of your tt-rss installation. This should be set to the location of tt-rss directory, e.g. http://example.org/tt-rss/<br> You need to set this option correctly otherwise several features including PUSH, bookmarklets and browser integration will not work properly. | `"{{ ttrss_url_prefix }}{{ ttrss_hostname }}` |
| `ttrss_init_system` | Service backend for [update daemon](https://tt-rss.org/wiki/UpdatingFeeds). `systemd` or `system-v` | `systemd` |
| `ttrss_db_user` | Database user | `ttrss` |
| `ttrss_url` | Public url of ttrss | `http://{{ ansible_default_ipv4.address }}` |
| `ttrss_db_password` | Database password, please change when using the role | `ttrss`|
| `ttrss_db_name` | Database name | `ttrss` |
| `ttrss_db_encoding` | Database encoding | `UTF-8`|
| `ttrss_db_collate` | Database collation, defines sort order and character classification | `en_US.UTF-8` |
| `ttrss_db_ctype` | Database ctype, defines character classification | `en_US.UTF-8` |
| `ttrss_git_repo` | URL of the Tiny Tiny RSS git repository, change when you want to use a fork | `https://tt-rss.org/gitlab/fox/tt-rss.git` |
| `ttrss_install_path` | Path to the folder, where Tiny Tiny RSS will be installed | `/var/www/rss` |
| `ttrss_url_path` | `URL PATH` of the Tiny Tiny RSS installation, change only when you know what you do | `{{ ttrss_install_path | basename }}` |
| `ttrss_enable_gzip` | Enable gzip out to improve wire performance, This requires PHP Zlib extension on the server | `true` |
| `ttrss_log_destination` | Sets log destination <ul><li>`syslog` - logs to system log</li><li>`sql` - logs to database, can be seen in `Preferences -> System`</li><li>`''` - uses PHP logging, usually the http server error log</li></ul>| `syslog` |
| `ttrss_hostname` | Hostname (FQDN) of the server | `''` |
| `ttrss_smtp_from_address`| address and subject for sending outgoing mail | `noreply@your.domain.dom` |
| `ttrss_smtp_server`|Hostname:port combination to send outgoing mail (i.e. localhost:25). Blank - use system MTA| `''` |
| `ttrss_smtp_login`| Username for SMTP authentication when sending outgoing mail | `''` |
| `ttrss_smtp_password`| Password for SMTP authentication when sending outgoing mail | `''` |
| `ttrss_smtp_secure`| Select a secure SMTP connection. Allowed values: `ssl`, `tls` or empty. | `''` |
| `ttrss_plugins_git` | List of git-urls to plugins to install | feediron |
| `ttrss_php_executable`| Path to cli binary | `/usr/bin/php7.4` |

The `ttrss_plugins_git` is a list wherase the `name`is also the target folder e.g. `{ttrss_install_path}/plugins.local/feediron`:

```yaml
- name: feediron
  url: https://github.com/feediron/ttrss_plugin-feediron.git
```

### php-specific

> The [role used for php](https://github.com/geerlingguy/ansible-role-php) offers much more configuration parameters. Currently we only pass a few which we consider useful

see link above

### postgres-specific

These parameters are specific to the postgres installation

> The [role used for postgres](https://github.com/geerlingguy/ansible-role-postgresql) offers much more configuration parameters. Currently we only pass a few which we consider necessary

|Variable|Description|Default Value|
|--------|-----------|-------------|
| `ttrss_postgresql_user` | Postgres admin user | `postgres` |
| `ttrss_postgresql_group` | User group for admin user | `postgres` |
| `ttrss_postgresql_locales` | Debian only. Used to generate the locales used by PostgreSQL databases | en_US.UTF-8' |
| `ttrss_postgres_users_no_log` | Whether to output user data when managing users | `true` |

## Dependencies

- [Collection `geerlingguy.php_roles`](https://galaxy.ansible.com/geerlingguy/php_roles) for installation of php
- [Role `geerlingguy.postgresql`](https://galaxy.ansible.com/geerlingguy/postgresql) for installation of postgresql

## Example Playbook

The following playbook creates a server (cx11) on [Hetzner Cloud](https://hetzner.cloud) - the instance [costs about 2.68€ per month](https://www.hetzner.com/cloud) which is quite a nice price:

```yml
- hosts: localhost
  gather_facts: no

  pre_tasks:
  - name: Add ssh-keys to hcloud
    hcloud_ssh_key:
      name: "{{ item.key }}"
      public_key: "{{ item.value }}"
      state: present
    with_dict: "{{ ssh_keys }}"
    delegate_to: localhost
    tags:
      - ansible
      - authorised_key


  - name: Create a basic server for ttrss
    hcloud_server:
      name: ttrss
      server_type: cx11
      image: debian-10
      state: present
      ssh_keys: 
        - ansible-user
    delegate_to: localhost
    register: result

  - name: Store ipv4 of host
    set_fact:
      ttrss_server_ipv4: "{{ result.hcloud_server.ipv4_address }}"

- hosts: ttrss
  vars:
  roles:
    { role: papanito.ttrss, ttrss_db_password: secret }
```

## License

This is Free Software, released under the terms of the Apache v2 license.

## Author Information

Written by [Papanito](https://wyssmann.com) - [Gitlab](https://gitlab.com/papanito) / [Github](https://github.com/papanito)
