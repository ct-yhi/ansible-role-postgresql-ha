## ANXS - PostgreSQL [![Build Status](https://travis-ci.org/ANXS/postgresql.svg?branch=master)](https://travis-ci.org/ANXS/postgresql)

---
Help Wanted! If you are able and willing to help maintain this Ansible role then please open a GitHub issue. A lot of people seem to use this role and we (quite obviously) need assistance!
💖
---

Ansible role which installs and configures PostgreSQL, extensions, databases and users.


#### Installation

This has been tested on Ansible 2.4.0 and higher.

To install:

```
ansible-galaxy install ANXS.postgresql
```

#### Example Playbook

Including an example of how to use your role:

    - hosts: postgresql-server
      become: yes
      roles:
         - { role: anxs.postgresql }

#### Compatibility matrix

| Distribution / PostgreSQL | <= 9.3 | 9.4 | 9.5 | 9.6 | 10 | 11 | 12 |
| ------------------------- |:---:|:---:|:---:|:---:|:--:|:--:|:--:|
| Ubuntu 14.04 | :no_entry: | :no_entry:| :no_entry:| :no_entry:| :no_entry:| :no_entry:| :no_entry:|
| Ubuntu 16.04 | :no_entry: | :white_check_mark:| :white_check_mark:| :white_check_mark:| :white_check_mark:| :white_check_mark:| :white_check_mark:|
| Debian 8.x | :no_entry: | :white_check_mark:| :white_check_mark:| :white_check_mark:| :white_check_mark:| :white_check_mark:| :white_check_mark:|
| Debian 9.x | :no_entry: | :white_check_mark:| :white_check_mark:| :white_check_mark:| :white_check_mark:| :white_check_mark:| :white_check_mark:|

- :white_check_mark: - tested, works fine
- :warning: - Not for production use
- :grey_question: - will work in the future (help out if you can)
- :interrobang: - maybe works, not tested
- :no_entry: - Has reached End of Life (EOL)


#### Replication with repmgr

There is initial support for setting up and running with replication managed by [repmgr](https://repmgr.org/). In it's current state it has only been tested with repmgr-4.2 on Centos 7 and requires Systemd.


#### Variables

```yaml
# Basic settings
postgresql_version: 12
postgresql_encoding: "UTF-8"
postgresql_locale: "en_US.UTF-8"
postgresql_ctype: "en_US.UTF-8"

postgresql_admin_user: "postgres"
postgresql_default_auth_method: "peer"

postgresql_service_enabled: false # should the service be enabled, default is true

postgresql_cluster_name: "main"
postgresql_cluster_reset: false

# List of databases to be created (optional)
# Note: for more flexibility with extensions use the postgresql_database_extensions setting.
postgresql_databases:
  - name: foobar
    owner: baz          # optional; specify the owner of the database
    hstore: yes         # flag to install the hstore extension on this database (yes/no)
    uuid_ossp: yes      # flag to install the uuid-ossp extension on this database (yes/no)
    citext: yes         # flag to install the citext extension on this database (yes/no)
    encoding: "UTF-8"   # override global {{ postgresql_encoding }} variable per database
    lc_collate: "en_GB.UTF-8"   # override global {{ postgresql_locale }} variable per database
    lc_ctype: "en_GB.UTF-8"     # override global {{ postgresql_ctype }} variable per database

# List of database extensions to be created (optional)
postgresql_database_extensions:
  - db: foobar
    extensions:
      - hstore
      - citext

# List of users to be created (optional)
postgresql_users:
  - name: baz
    pass: pass
    encrypted: yes  # if password should be encrypted, postgresql >= 10 does only accepts encrypted passwords

# List of schemas to be created (optional)
postgresql_database_schemas:
  - database: foobar           # database name
    schema: acme               # schema name
    state: present

  - database: foobar           # database name
    schema: acme_baz           # schema name
    owner: baz                 # owner name
    state: present

# List of user privileges to be applied (optional)
postgresql_user_privileges:
  - name: baz                   # user name
    db: foobar                  # database
    priv: "ALL"                 # privilege string format: example: INSERT,UPDATE/table:SELECT/anothertable:ALL
    role_attr_flags: "CREATEDB" # role attribute flags

# Manage replication with repmgr (optional)
repmgr_target_group: "postgresql-db"
postgresql_ext_install_repmgr: yes
repmgr_user: repmgr
repmgr_database: repmgr
postgresql_wal_level: "replica"
postgresql_max_wal_senders: 10
postgresql_max_replication_slots: 10
postgresql_wal_keep_segments: 100
postgresql_hot_standby: on
postgresql_archive_mode: on
postgresql_archive_command: "test ! -f /tmp/%f && cp %p /tmp/%f"
postgresql_shared_preload_libraries:
  - repmgr

postgresql_users:
  - name: "{{repmgr_user}}"
    pass: "password"

postgresql_databases:
  - name: {{repmgr_database}}
    owner: "{{repmgr_user}}"
    encoding: "UTF-8"

postgresql_user_privileges:
  - name: "{{repmgr_user}}"
    db: {{repmgr_database}}
    priv: "ALL"
    role_attr_flags: "SUPERUSER,REPLICATION"
```

There's a lot more knobs and bolts to set, which you can find in the [defaults/main.yml](./defaults/main.yml)

## Install requirements

```bash
ansible-galaxi install -r requirements.yml -p ~/.roles
```

## After setup : verifying cluster functionality using Ansible ad-hoc command 

```bash
ansible postgres_cluster -b --become-user postgres -m shell -a "repmgr -f /etc/postgresql/11/main/repmgr.conf cluster show"
```