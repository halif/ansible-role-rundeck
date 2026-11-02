# Ansible Role: Rundeck

[![CI](https://github.com/halif/ansible-role-rundeck/actions/workflows/ci.yml/badge.svg)](https://github.com/halif/ansible-role-rundeck/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Ansible role for installing and configuring [Rundeck](https://www.rundeck.com/) on Linux servers.

Rundeck is a powerful platform for job scheduling, workflow orchestration, runbook automation, and infrastructure operations. It provides a convenient web interface for managing operational tasks and automating routine activities.

This role aims to make Rundeck deployment reproducible, configurable, and easy to maintain.

## Features

- Automated Rundeck installation from official package repositories.
- Support for Debian-based and RedHat-based Linux distributions.
- Automatic Java installation (OpenJDK 17 by default).
- Configurable Rundeck version.
- Database configuration:
  - H2 (default)
  - MySQL
  - PostgreSQL
- Automatic JDBC driver installation for external databases.
- Optional database creation.
- Rundeck configuration management.
- Service management and automatic startup configuration.
- Molecule tests and GitHub Actions CI pipeline.

## Supported platforms

The role supports Debian and RedHat OS families.

The following platforms are currently covered by CI testing:

| Distribution | Version | CI scenario |
|---|---|---|
| Ubuntu | 22.04 | default |
| Rocky Linux | 9 | rocky9 |

Other compatible Debian- and RedHat-based distributions may work but have not necessarily been tested.

## Requirements

Ansible must be installed on the control node.

The role uses the following Ansible collections:

- `ansible.mysql`
- `community.postgresql`

Install the required collections:

```bash
ansible-galaxy collection install -r requirements-collections.yml
```

For external database management, the appropriate database Python drivers and administrative permissions must also be available on the delegated database host.

The target server must have access to the Rundeck package repositories and the required package repositories for Java.

## Installation

Install the role from Ansible Galaxy:

```bash
ansible-galaxy role install halif.rundeck
```

Install the role's collection dependencies:

```bash
ansible-galaxy collection install -r requirements-collections.yml
```

## Usage

Create a playbook, for example `rundeck.yml`:

```yaml
---
- name: Install and configure Rundeck
  hosts: rundeck
  become: true

  roles:
    - role: halif.rundeck
```

Run the playbook:

```bash
ansible-playbook -i inventory rundeck.yml
```

By default, the role installs Rundeck with Java 17 and the embedded H2 database.

## Database configuration

The role supports three database types.

### H2

H2 is the default database and can be used for testing, evaluation, and non-production environments.

```yaml
rundeck_database_type: h2
```

**Important:** H2 is not recommended or supported for production use. Rundeck recommends an external database because an ungraceful shutdown can result in database corruption or data loss.

See the [official Rundeck database documentation](https://docs.rundeck.com/docs/administration/configuration/database/).

### MySQL

Example configuration:

```yaml
rundeck_database_type: mysql

rundeck_database_host: "mysql.example.com"
rundeck_database_port: 3306

rundeck_database_name: "rundeck"
rundeck_database_username: "rundeck"
rundeck_database_password: "{{ vault_rundeck_database_password }}"

rundeck_database_manage_db: false
```

The role automatically downloads the configured MySQL JDBC driver.

### PostgreSQL

Example configuration:

```yaml
rundeck_database_type: postgresql

rundeck_database_host: "postgres.example.com"
rundeck_database_port: 5432

rundeck_database_name: "rundeck"
rundeck_database_username: "rundeck"
rundeck_database_password: "{{ vault_rundeck_database_password }}"

rundeck_database_manage_db: false
```

The role automatically downloads the configured PostgreSQL JDBC driver.

### Database management

By default, the role does not create external databases.

Set:

```yaml
rundeck_database_manage_db: true
```

to enable optional database creation tasks.

For this functionality, provide the appropriate administrative credentials:

```yaml
rundeck_database_admin_user: postgres
rundeck_database_admin_password: "{{ vault_database_admin_password }}"
```

For MySQL, use an appropriate administrative account such as `root`.

The database server must be reachable through Ansible delegation.

**Security recommendation:** Store database passwords and other sensitive values using Ansible Vault rather than committing plaintext credentials to your repository.

## Role variables

The following table lists the main configurable variables.

| Variable | Default | Description |
|---|---|---|
| `rundeck_version` | `5.1.0.20240214-1` | Rundeck package version |
| `rundeck_package_state` | `present` | Desired package state |
| `rundeck_install_java` | `true` | Install Java automatically |
| `rundeck_java_package_debian` | `openjdk-17-jre-headless` | Java package for Debian |
| `rundeck_java_package_redhat` | `java-17-openjdk-headless` | Java package for RedHat |
| `rundeck_database_type` | `h2` | Database type |
| `rundeck_database_host` | `localhost` | Database hostname |
| `rundeck_database_name` | `rundeck` | Database name |
| `rundeck_database_username` | `rundeck` | Database username |
| `rundeck_database_password` | `rundeck` | Database password |
| `rundeck_database_manage_db` | `false` | Create the external database |
| `rundeck_jdbc_mysql_version` | `8.0.33` | MySQL JDBC driver version |
| `rundeck_jdbc_postgresql_version` | `42.7.3` | PostgreSQL JDBC driver version |
| `rundeck_jdbc_lib_dir` | `/var/lib/rundeck/lib` | JDBC driver directory |

Additional variables, including service and configuration options, are available in `defaults/main.yml`.

## Testing

The role uses [Molecule](https://ansible.readthedocs.io/projects/molecule/) for automated testing.

GitHub Actions runs the following checks:

- YAML linting with `yamllint`.
- Ansible linting with `ansible-lint`.
- Molecule syntax validation.
- Container creation.
- Role convergence.
- Idempotence testing.
- Configuration verification.
- Container cleanup.

The CI pipeline tests the role against Ubuntu 22.04 and Rocky Linux 9.

## Project structure

```text
ansible-role-rundeck/
├── .github/
│   └── workflows/
│       └── ci.yml
├── defaults/
│   └── main.yml
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── molecule/
│   ├── default/
│   └── rocky9/
├── tasks/
│   ├── main.yml
│   ├── install-debian.yml
│   ├── install-redhat.yml
│   ├── database.yml
│   └── configure.yml
├── templates/
├── vars/
│   ├── debian.yml
│   └── redhat.yml
├── requirements-collections.yml
├── .ansible-lint
├── .yamllint
├── LICENSE
└── README.md
```

## Contributing

Contributions, bug reports, and feature requests are welcome.

Please open an issue or submit a pull request through the GitHub repository.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Author

**halif**

GitHub: [https://github.com/halif](https://github.com/halif)

Repository: [https://github.com/halif/ansible-role-rundeck](https://github.com/halif/ansible-role-rundeck)