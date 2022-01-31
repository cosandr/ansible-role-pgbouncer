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

SSL certificates must be copied before running this role if `ssl_enable` is true.

## Role Variables

- `postgresql_version` determines which PG repository is installed, should match PG server(s)
- `pgbouncer_allow_sources` list of IPs allowed to connect to pgbouncer, all will be allowed if not specified
- `pgbouncer_databases` list of databases, defaults to localhost on unix socket
- `pgbouncer_default_pool_size` defaults to 1000
- `pgbouncer_instances` how many pgbouncer instances to run, defaults to 3
- `pgbouncer_listen` defaults to `*`
- `pgbouncer_max_client_conn` defaults to 2048
- `pgbouncer_pool_mode` defaults to `session`
- `pgbouncer_port` defaults to 6432
- `pgbouncer_reserve_pool_size` defaults to 100
- `ssl_enable` whether or not to enable SSL, requires the following files to be present on the remote node:
  - `ssl_ca_path` defaults to `/etc/pki/tls/certs/pg-ca.crt`
  - `ssl_cert_path` defaults to `/etc/pki/tls/certs/pgbouncer.crt`
  - `ssl_private_key_path` defaults to `/etc/pki/tls/private/pgbouncer.key`

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
```

## Author Information

Andrei Costescu
