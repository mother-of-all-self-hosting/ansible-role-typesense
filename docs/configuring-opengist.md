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

# Setting up Opengist

This is an [Ansible](https://www.ansible.com/) role which installs [Opengist](https://opengist.io/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Opengist is a self-hosted pastebin powered by Git. All snippets are stored in a Git repository and can be read and/or modified using standard Git commands, or with the web interface.

See the project's [documentation](https://opengist.io/docs/) to learn what Opengist does and why it might be useful to you.

## Prerequisites

To run a Opengist instance it is necessary to prepare a database. You can use a [MySQL](https://www.mysql.com/) compatible database server, [Postgres](https://www.postgresql.org/), or [SQLite](https://www.sqlite.org/). By default it is configured to use SQLite.

If you are looking for Ansible roles for a MySQL compatible server or Postgres, you can check out [ansible-role-mariadb](https://github.com/mother-of-all-self-hosting/ansible-role-mariadb) and [ansible-role-postgres](https://github.com/mother-of-all-self-hosting/ansible-role-postgres), both of which are maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Opengist with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# opengist                                                             #
#                                                                      #
########################################################################

opengist_enabled: true

########################################################################
#                                                                      #
# /opengist                                                            #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Opengist you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
opengist_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Opengist under a subpath (by configuring the `opengist_path_prefix` variable) does not seem to be possible due to Opengist's technical limitations.

### Set 32-byte hex digits for secret key

You also need to specify **32-byte hex digits** for session store and encrypting MFA data on the database. To do so, add the following configuration to your `vars.yml` file. The value can be generated with `openssl rand -hex 32` or in another way.

```yaml
opengist_environment_variables_secret_key: YOUR_SECRET_KEY_HERE
```

>[!NOTE]
> Other type of values such as one generated with `pwgen -s 64 1` does not work.

### Specify database (optional)

You can specify a database used by Opengist. By default it is configured to use SQLite, and the SQLite database is stored in the directory specified with `opengist_data_path`.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
opengist_database_type: postgres
```

Set `mysql` to use a MySQL compatible database.

For other settings, check variables such as `opengist_database_postgres_*` and `opengist_database_mysql_*` on [`defaults/main.yml`](../defaults/main.yml).

### Configuring SSH port for Opengist (optional)

Opengist uses port 2222 for its optional SSH feature.

If you wish to expose the port, add the following configuration to your `vars.yml` file and adjust the port as you see fit.

```yaml
opengist_container_ssh_host_bind_port: 2222
```

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `opengist_environment_variables_additional_variables` variable

See [the official documentation](https://opengist.io/docs/configuration/cheat-sheet.html) for a complete list of Opengist's config options that you could put in `opengist_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Opengist becomes available at the specified hostname like `https://example.com`.

To get started, open the URL with a web browser, and register the account. **Note that the first registered user becomes an administrator automatically.**

>[!WARNING]
> On the current version (as of `1.11.0`) are there several inconveniences related to account management, which could lead you from being locked out of the administrator account.
>
> - If you are a solo administrator on the instance, deleting yourself on `https://example.com/admin-panel/users` leads that there will be no administrator; you will basically be locked out of the admin panel and can no longer configure the instance with it.
> - Be careful when setting both "Disable signup" and "Disable login form" to be effective on `https://example.com/admin-panel/configuration`; you will be locked out of the administrator account with incorrect OAuth settings.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu opengist` (or how you/your playbook named the service, e.g. `mash-opengist`).

#### Increase logging verbosity

If you want to increase the verbosity, add the following configuration to your `vars.yml` file:

```yaml
opengist_environment_variables_log_level: debug
```
