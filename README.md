# wordpress-env

A development environment for multiple WordPress sites.

## Usage

Put `docker-compose.override.yml`:

```sh
$ cp docker-compose.override.yml.example docker-compose.override.yml
```

Run services:

```sh
$ docker-compose up
```

### Add sites

If you create a directory for the site under `sites`, the directory name is used as database name and hostname (`http://<site_name>.localhost`).

```sh
$ mkdir sites/<site_name> # put WordPress files into this directory
$ cp site-example/wp-config.php sites/<site_name>/wp-config.php
```

```sql
CREATE DATABASE <site_name> DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```
