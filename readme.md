# Introduction

This project has been made to easily set up a GLPI development environment using `docker`.

Feel free to open issues and propose enhancements.

# Create and configure containers

## Using docker-compose
`docker-compose` is used to simplifies Docker containers definition and configuration using definitions declared in `docker-compose.*-.yml` files.
Services declarations has been splitted into multiple files to give ability to run only required services.
Configuration files are:
- `docker-compose.yml` for the webserver running GLPI application,
- `docker-compose.database.yml` for the database server,
- `docker-compose.utilities.yml` for development utilities, like `Adminer` and `MailHog`.

## Configuring containers you want to use
By default, `docker-compose` command will use only the `docker-compose.yml` configuration file that defines only the application container. If you also want to use database and utilities containers, you have to specify it. There is two ways to do so.

The first solution is to specify list of configurations files into the `COMPOSE_FILE` variable in the `.env` configuration file, for example: `COMPOSE_FILE=docker-compose.yml:docker-compose.database.yml`. It is the recommended solution.

The second solution is to use the `--file /path/to/docker-compose.file.yml` option on `docker-compose` command call for each file to use.

## Defining configuration variables
You can define some configuration variables using a `.env` file located in the same directory as `docker-compose.*-.yml` configuration files.
All variables are described in the following tables and have default values that makes their definition optionnal.

### Application container variables

| Variable name | Description | Default value |
| --- | --- | --- |
| `APPLICATION_CONTAINER_NAME` | Container name. Can be used to identify container when using `docker` and `docker-compose` commands. | `glpi` |
| `APPLICATION_CONTAINER_BASE_IMAGE` | Container base image. | `php:apache-buster` |
| `APPLICATION_CONTAINER_RESTART_POLICY` | [Restart policy.](https://docs.docker.com/config/containers/start-containers-automatically/) | `unless-stopped` |
| `APPLICATION_NETWORK_ALIAS` | Additionnal network alias. | `glpi.local` |
| `APPLICATION_HTTP_PORT` | Http port listened by container. Can be changed to prevent conflicts between multiple containers. | `80` |
| `APPLICATION_PATH` | Root path of application source. Will be mounted in `/var/www/glpi`. | `./mounts/application/var/www/glpi` |
| `HOST_GROUP_ID` | Group id used by `Apache` group. Should have the same value as group used in host system to access files in order to prevent permission conflicts. | `1000` |
| `HOST_USER_ID` | User id used by `Apache` user. Should have the same value as user used in host system to access files in order to prevent permission conflicts. | `1000` |
| `NETWORK_NAME` | Network name. Can be used in `docker network` commands, for example if you need to connect a tier container to GLPI API. | `glpi_bridge` |

### Database container variables

| Variable name | Description | Default value |
| --- | --- | --- |
| `DATABASE_CONTAINER_NAME` | Container name. Can be used to identify container when using `docker` and `docker-compose` commands. | `glpi_db` |
| `DATABASE_CONTAINER_IMAGE` | Container image to use. | `mariadb:10.4` |
| `DATABASE_CONTAINER_RESTART_POLICY` | [Restart policy.](https://docs.docker.com/config/containers/start-containers-automatically/) | `unless-stopped` |
| `DATABASE_CONFIG_PATH` | Path of database custom configuration file. Will be mounted in `/etc/mysql/conf.d/custom.cnf`. | `./mounts/db/etc/mysql/conf.d/custom.cnf` |
| `DATABASE_STORAGE_PATH` | Root path of database storage files. Will be mounted in `/var/lib/mysql`. If path is empty, container will initialize a new database at startup. | `./mounts/db/var/lib/mysql` |
| `MYSQL_DATABASE` | Name of database that will be created on database initialization. | `glpi` |
| `MYSQL_PASSWORD` | Password of user that will be created on database initialization. | `glpi` |
| `MYSQL_USER` | Name of user that will be created on database initialization. | `glpi` |
| `NETWORK_NAME` | Network name. Can be used in `docker network` commands, for example if you need to connect a tier container to GLPI database. | `glpi_bridge` |

### Utilities containers variables

| Variable name | Description | Default value |
| --- | --- | --- |
| `UTILITIES_CONTAINERS_RESTART_POLICY` | [Restart policy.](https://docs.docker.com/config/containers/start-containers-automatically/) | `unless-stopped` |

## Build application container
Application container has to be built. In order to do so, run the command `docker-compose -f docker-compose.yml build`.

# Install GLPI

## Run containers
To be able to proceed to GLPI installation, containers must set up using the `docker-compose up` command.

## Fetch PHP dependencies
If it is not done yet, fetch PHP dependencies using the `docker exec --tty --user www-data glpi composer install --working-dir=/var/www/glpi` command.

## Launch installation wizard
You should be able to launch installation wizard by accessing <http://127.0.0.1/> inside your web browser.

Installation wizard will ask you for database hostname. If you use the database container, you can use `db` as hostname, which corresponds to the entry in `docker-compose.database.yml` configuration file, or use the container name defined in the `DATABASE_CONTAINER_NAME` environment variable.

# Work with containers

## Run containers
To run containers, use the `docker-compose up` command. You can use the `--detach` to run process in background.

## Stop containers
If you run containers in foreground, you can stop them using `CTRL+C`, otherwise, use the `docker-compose stop` command to stop them.

## Run command inside containers
To run a command inside a container, use the `docker exec` command.

For example, to open a bash console inside the application container using the `www-data` user, use `docker exec --interactive --tty --user www-data glpi bash`.

# Utilities containers

## Adminer
An instance of `Adminer`, a database management web tool, can be accessed at <http://127.0.0.1:8080/>.

## MailHog
An instance of `MailHog`, a SMTP server which catches any message sent to it to display in a web interface, can be accessed at <http://127.0.0.1:8025/>.

SMTP server will be reachable by your GLPI applicating using the `mail` host on port `1025`.

The PHP `mail` function is configured to send messages to a `ssmtp` instance which will forward them to `MailHog`.

## SMTP (namshi/smtp)
A SMTP server that can be used for the `release mail` function of `MailHog`.
