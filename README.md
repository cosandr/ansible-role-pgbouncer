# xait_database_pgbouncer

Installs and configures pgbouncer on EL8 or later.

## Usage

[Offical Ansible documentation](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#installing-multiple-roles-from-a-file).

### TL;DR

Install manually
```sh
# If roles_path is set in ansible.cfg
ansible-galaxy install git+ssh://gitlab.xait.no/ansible-roles/xait_database_pgbouncer.git
# If not, assuming you want to install to ./roles
ansible-galaxy install -p roles git+ssh://gitlab.xait.no/ansible-roles/xait_database_pgbouncer.git
```

With requirements.yml
```yml
- src: git@gitlab.xait.no:ansible-roles/xait_database_pgbouncer.git
  scm: git
```
```sh
# Same caveat about roles_path applies
ansible-galaxy install -r requirements.yml
```

## Requirements

SSL certificates must be copied before running this role if `pgbouncer_ssl_enable` is true.

A user_lookup function is required in all databases to be accessed by pgbouncer, [for example](https://www.pgbouncer.org/config.html#example):

```sql
CREATE SCHEMA AUTHORIZATION pgbouncer;
-- https://www.enterprisedb.com/postgres-tutorials/pgbouncer-authquery-and-authuser-pro-tips
CREATE OR REPLACE FUNCTION pgbouncer.user_lookup(username text) RETURNS TABLE(usename name, passwd text) AS 'SELECT usename, passwd FROM pg_catalog.pg_shadow WHERE usename = username' LANGUAGE sql SECURITY DEFINER;
REVOKE ALL ON FUNCTION pgbouncer.user_lookup(text) FROM public, pgbouncer;
GRANT EXECUTE ON FUNCTION pgbouncer.user_lookup(text) TO pgbouncer;
```

The queries can be run on a template, then any databases created from them will inherit the `pgbouncer` schema and `user_lookup` function.

Don't forget to configure `pg_hba.conf` as well:

```conf
# UNIX socket on the same host
local all pgbouncer peer
# TCP from another host (10.0.10.20 is the pgbouncer host)
host all pgbouncer 10.0.10.20 md5
```

## Role Variables

- `postgresql_version` determines which PG repository is installed, should match PG server(s)
- `pgbouncer_auth_type` defaults to md5
- `pgbouncer_auth_user` determines user that runs the auth query, defaults to `pgbouncer`
- `pgbouncer_auth_users` key-value pair of usernames and passwords for pgbouncer's internal database, should contain `pgbouncer_auth_user`

  Defaults to `pgbouncer` with an empty password (uses peer authentication if the PG server is on the same host)
- `pgbouncer_allow_sources` list of IPs allowed to connect to pgbouncer, all will be allowed if not specified
- `pgbouncer_databases` list of databases, defaults to localhost on unix socket
- `pgbouncer_default_pool_size` defaults to 1000
- `pgbouncer_instances` how many pgbouncer instances to run, defaults to number of CPUs with a fallback of 3
- `pgbouncer_listen` defaults to `*`
- `pgbouncer_max_client_conn` defaults to 2048
- `pgbouncer_pool_mode` defaults to `session`
- `pgbouncer_port` defaults to 6432
- `pgbouncer_reserve_pool_size` defaults to 100
- `pgbouncer_ssl_enable` whether or not to enable SSL, requires the following files to be present on the remote node:
  - `pgbouncer_ssl_ca_path` defaults to `/etc/pki/tls/certs/pg-ca.crt`
  - `pgbouncer_ssl_cert_path` defaults to `/etc/pki/tls/certs/pgbouncer.crt`
  - `pgbouncer_ssl_private_key_path` defaults to `/etc/pki/tls/private/pgbouncer.key`

## Example Playbook

```yml
- hosts: servers
  roles:
    - role: xait_database_pgbouncer
      vars:
        pgbouncer_instances: 5
        pgbouncer_databases:
          - name: '*'
            # See https://www.pgbouncer.org/config.html#section-databases
            opts: 'host=10.0.10.10'
        # See https://www.pgbouncer.org/config.html#authentication-file-format
        pgbouncer_users:
          # Generate with echo -n "md5"; echo -n "<pass><user>" | md5sum
          pgbouncer: "md50722e9d2276ab1b417482b89315eed00" # echo -n "md5"; echo -n "securepgbouncer" | md5sum
```

## Author Information

Andrei Costescu
