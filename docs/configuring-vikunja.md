<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Vikunja

This is an [Ansible](https://www.ansible.com/) role which installs [Vikunja](https://vikunja.io/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Vikunja is a self-hosted pastebin powered by Git. All snippets are stored in a Git repository and can be read and/or modified using standard Git commands, or with the web interface.

See the project's [documentation](https://vikunja.io/docs/) to learn what Vikunja does and why it might be useful to you.

## Prerequisites

To run a Vikunja instance it is necessary to prepare a database. You can use a [MySQL](https://www.mysql.com/) compatible database server, [Postgres](https://www.postgresql.org/), or [SQLite](https://www.sqlite.org/). By default it is configured to use SQLite.

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Vikunja with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# vikunja                                                              #
#                                                                      #
########################################################################

vikunja_enabled: true

########################################################################
#                                                                      #
# /vikunja                                                             #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Vikunja you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
vikunja_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Vikunja under a subpath (by configuring the `vikunja_path_prefix` variable) does not seem to be possible due to Vikunja's technical limitations.

### Set a random string for JWT tokens verification

You also need to set a random string used for verifying issued JWT tokens. To do so, add the following configuration to your `vars.yml` file. The value can be generated with `pwgen -s 64 1` or in another way.

```yaml
vikunja_environment_variables_service_jwtsecret: YOUR_SECRET_KEY_HERE
```

### Specify database (optional)

You can specify a database used by Vikunja. By default it is configured to use SQLite, and the SQLite database is stored in the directory specified with `vikunja_database_path`.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
vikunja_database_type: postgres
```

Set `mysql` to use a MySQL compatible database.

For other settings, check variables such as `vikunja_database_postgres_*` and `vikunja_database_mysql_*` on [`defaults/main.yml`](../defaults/main.yml).

### Configure a Redis server for caching (optional)

Also, you can optionally enable a [Redis](https://redis.io/) server for managing cache.

If you are looking for an Ansible role for Redis, you can check out [ansible-role-redis](https://github.com/mother-of-all-self-hosting/ansible-role-redis) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team. The roles for [KeyDB](https://keydb.dev/) ([ansible-role-keydb](https://github.com/mother-of-all-self-hosting/ansible-role-keydb)) and [Valkey](https://valkey.io/) ([ansible-role-valkey](https://github.com/mother-of-all-self-hosting/ansible-role-valkey)) are available as well.

To enable Redis for Vikunja, add the following configuration to your `vars.yml` file, so that the Vikunja instance will connect to the server:

```yaml
vikunja_redis_hostname: YOUR_REDIS_SERVER_HOSTNAME_HERE
vikunja_redis_port: 6379
vikunja_redis_password: YOUR_REDIS_SERVER_PASSWORD_HERE
vikunja_redis_database: 0
```

Make sure to replace `YOUR_REDIS_SERVER_HOSTNAME_HERE` and `YOUR_REDIS_SERVER_PASSWORD_HERE` with your own values.

### Configure the mailer (optional)

You can configure a SMTP mailer to enable it for signing up and resetting password. If it is disabled, all users are enabled right away and password reset will not be possible.

To configure it, add the following configuration to your `vars.yml` file as below (adapt to your needs):

```yaml
# Set to `true` to enable mailer
vikunja_environment_variables_smtp_enabled: true

# Set the hostname of the SMTP server
vikunja_environment_variables_smtp_host: ""

# Set the port number of the SMTP server
vikunja_environment_variables_smtp_port: 587

# Set the username for the SMTP server
vikunja_environment_variables_smtp_user: ""

# Set the password for the SMTP server
vikunja_environment_variables_smtp_password: ""

# Set the email address that emails will be sent from
vikunja_environment_variables_smtp_from: ""

# Specify the SMTP Auth Type
# Valid values: plain, login, cram-md5
vikunja_environment_variables_smtp_authtype: plain

# Set to `true` to skip verification of the TLS certificate on the server
vikunja_environment_variables_skiptlsverify: false

# Set to `true` to make Vikunja use SSL instead of STARTTLS.
vikunja_environment_variables_smtp_forcessl: false
```

⚠️ **Note**: without setting an authentication method such as DKIM, SPF, and DMARC for your hostname, emails are most likely to be quarantined as spam at recipient's mail servers. If you have set up a mail server with the [MASH project's exim-relay Ansible role](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay), you can enable DKIM signing with it. Refer [its documentation](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay/blob/main/docs/configuring-exim-relay.md#enable-dkim-support-optional) for details.

### Enabling user registration

By default the role is configured to disable user registration. You can enable it by adding the following configuration to your `vars.yml` file.

```yaml
vikunja_environment_variables_service_enableregistration: true
```

Alternatively, you can also create users by running the command to run [`user create`](https://vikunja.io/docs/cli/#user-create) inside the container. See below in [this section](#creating-users) for the usage.

### Configuring rate limit

You can enable the rate limit by adding the following configuration to your `vars.yml` file:

```yaml
vikunja_environment_variables_ratelimit_enabled: true
```

### Configuring SSH port for Vikunja (optional)

Vikunja uses port 2222 for its optional SSH feature.

If you wish to expose the port, add the following configuration to your `vars.yml` file and adjust the port as you see fit.

```yaml
vikunja_container_ssh_host_bind_port: 2222
```

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `vikunja_environment_variables_additional_variables` variable

See [the official documentation](https://vikunja.io/docs/config-options/) for a complete list of Vikunja's config options that you could put in `vikunja_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Vikunja becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser, and register the account. **Note that the first registered user becomes an administrator automatically.**

>[!WARNING]
> On the current version (as of `1.11.0`) are there several inconveniences related to account management, which could lead you from being locked out of the administrator account.
>
> - If you are a solo administrator on the instance, deleting yourself on `https://example.com/admin-panel/users` leads that there will be no administrator; you will basically be locked out of the admin panel and can no longer configure the instance with it.
> - Be careful when setting both "Disable signup" and "Disable login form" to be effective on `https://example.com/admin-panel/configuration`; you will be locked out of the administrator account with incorrect OAuth settings.

### Creating users

You can create users by running the command below to run [`user create`](https://vikunja.io/docs/cli/#user-create) inside the container.

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=user-create-vikunja -e username=USERNAME_HERE -e password=PASSWORD_HERE -e email=EMAIL_ADDRESS_HERE
```

### Running the CLI command

It is possible to run commands on the command line inside the container by running the `cli-vikunja` tag, setting the `command` extra variable.

For example, you can run the command `version` by running the playbook with the tag as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=cli-vikunja -e command='version'
```

See [this page](https://vikunja.io/docs/cli/) for the list of available commands.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu vikunja` (or how you/your playbook named the service, e.g. `mash-vikunja`).

#### Increase logging verbosity

If you want to increase the verbosity, add the following configuration to your `vars.yml` file:

```yaml
vikunja_environment_variables_log_level: DEBUG
```
