<!--
SPDX-FileCopyrightText: 2018-2025 Slavi Pantaleev
SPDX-FileCopyrightText: 2019-2022 Aaron Raimist
SPDX-FileCopyrightText: 2019-2023 MDAD project contributors
SPDX-FileCopyrightText: 2023 QEDeD
SPDX-FileCopyrightText: 2024 Fabio Bonelli
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Molecule Testing

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

## Prerequisites

To utilize Molecule you need to prepare several requirements:

- **x86** computer running one of these operating systems that make use of [systemd](https://systemd.io/):
  - **Archlinux**
  - **CentOS**, **Rocky Linux**, **AlmaLinux**, or possibly other RHEL alternatives (although your mileage may vary)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/ansible.md#supported-ansible-versions) if you run the Ansible playbook on it)
- `root` access on the computer which Molecule runs against
- [Ansible](http://ansible.com/) program
- [Python](https://www.python.org/)
  - Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`)
- [Docker](https://www.docker.com)
  - Access to Docker UNIX socket (`/var/run/docker.sock`) is required by default

## Installation

To set up the environment for using Molecule, run the command below on the terminal:

```bash
python3 -m venv ./molecule/venv
source ./molecule/venv/bin/activate
pip3 install -r ./molecule/requirements.txt
```

## Scenarios

Currently these testing scenarios are available:

### `default`

Tests a standard Vikunja installation, backed by SQLite.

### `mariadb`

Tests a standard Vikunja installation with the MariaDB database.

### `postgres`

Tests a standard Vikunja installation with the Postgres database.

## What is verified

Every scenario runs the same set of checks (in [`resources/tasks/verify_vikunja.yml`](./resources/tasks/verify_vikunja.yml)) and then adds whatever proves its own database backend is the one carrying the data.

The shared checks are built around one observation: a Vikunja that cannot reach its database parks itself in `Running migrations…` and stays there. The container keeps running, so systemd keeps reporting the service `active`, and yet nothing is ever served. A service being `active` therefore proves very little here, and the checks are chosen so that they cannot pass against such an instance:

- the API answers on `/api/v1/info`, which Vikunja only serves once it has connected to its database and migrated it
- the running build reports the version [`defaults/main.yml`](../defaults/main.yml) pins, asked both over HTTP and of the binary itself
- values the role writes into the env file are read back out of the running process — the public URL, the registration setting, and a MOTD that can only arrive through `vikunja_environment_variables_additional_variables`
- self-registration is refused, as the role configures
- the API refuses unauthenticated callers
- the role's own `user create` task file creates a user, who then logs in and receives a JWT
- a project and a task are created over the API and read back

Each scenario then queries its own backend directly for the task and the user, and the `mariadb` and `postgres` scenarios additionally assert that no SQLite database was written alongside — a fallback would otherwise leave every check above green.

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
```

You can utilize other distributions by setting one to the `MOLECULE_DISTRO` environment variable:

```bash
# Ubuntu 24.04
MOLECULE_DISTRO=ubuntu2404 molecule test --scenario-name default

# Debian 13
MOLECULE_DISTRO=debian13 molecule test --scenario-name default

# Debian 12
MOLECULE_DISTRO=debian12 molecule test --scenario-name default
```
