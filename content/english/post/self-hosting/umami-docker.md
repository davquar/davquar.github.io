---
author: Davide Quaranta
title: Self-hosting Umami with Docker Compose
date: 2022-09-01T00:00:00Z
categories: [Self-Hosting]
tags: [devops, sysadmin, umami, analytics, docker]
description: Umami is an open-source, privacy oriented and lightweight web analytics service written in Node. Umami is super easy and quick to self-host, and in this post we'll see a setup with Docker Compose.
---

The setup described in this post assumes that we already:

* Have a website to monitor.
* Have a domain to expose our Umami instance on.
* Have Docker and Compose installed and operational.

The domain configuration is not covered here since you can do it as you prefer, but in the setup presented here, it is handled by these images:

* [nginx-proxy/nginx-proxy](https://github.com/nginx-proxy/nginx-proxy): automatic nginx proxy configuration.
* [nginx-proxy/acme-companion](https://github.com/nginx-proxy/acme-companion): automatic TLS certificate renewal.

The images that make up the Umami stack instead are (adapt versions as needed):

* Umami: [ghcr.io/umami-software/umami:postgresql-v1.37.0](https://github.com/umami-software/umami/pkgs/container/umami).
* PostgreSQL: [postgres:12-alpine](https://hub.docker.com/_/postgres/).

Umami also supports MySQL: the configuration is almost identical: just change images and configuration parameters appropriately.

## Umami setup with Docker Compose

### Preparation

We want to create this structure:

```
umami/
├── .env
├── docker-compose.yml
└── sql/
    └── schema.postgresql.sql
```

So in the `umami` directory we can just execute:

```shell
mkdir sql
cd sql
wget https://raw.githubusercontent.com/umami-software/umami/master/sql/schema.postgresql.sql
```


The file `schema.postgresql.sql` is needed to execute the initial database migrations to create the schema, and it is available in [Umami's GitHub repository](https://github.com/umami-software/umami).

The `docker-compose.yml` file is also in the repository as an example, and it is quite perfect to start our configuration; in facts the setup that you see below is just a minor modification of it.

### `docker-compose.yml`

The Docker Compose that we want is:

```yaml
version: '3'

services:
  umami:
    container_name: umami
    image: ghcr.io/umami-software/umami:postgresql-v1.37.0
    ports:
      - 127.0.0.1:3000:3000
    env_file: .env
    environment:
      TRACKER_SCRIPT_NAME: saporito
      VIRTUAL_HOST: stats.ourdomain.ext
      LETSENCRYPT_HOST: stats.ourdomain.ext
      LETSENCRYPT_EMAIL: ouremail@ourdomain.ext
    networks:
      - default
      - proxy
    depends_on:
      - db
    restart: always

  db:
    container_name: umami_db
    image: postgres:12-alpine
    env_file: .env
    volumes:
      - ./sql/schema.postgresql.sql:/docker-entrypoint-initdb.d/schema.postgresql.sql:ro
      - umami-db-data:/var/lib/postgresql/data
    networks:
      - default
    restart: always

volumes:
  umami-db-data:

networks:
  proxy:
    external:
      name: nginx-proxy
```

Some comments:

* Umami is only exposed to `localhost`, because from the outside we want to reach it only through the proxy, not directly via port number.
* The environment variables `VIRTUAL_HOST`, `LETSENCRYPT_*` are needed for the proxy, hence not strictly related to Umami.
* `TRACKER_SCRIPT_NAME` allows to change the filename of the tracking script to avoid being blocked by some ad blockers; I have chosen "saporito" because it is the Italian translation of "umami", which means "tasty". With this configuration the tracking script will be like `https://stats.ourdomain.ext/saporito.js` instead of `https://stats.ourdomain.ext/umami.js`. This setting is optional.
* To communicate with each other, `umami` and `db` are part of the same `default` network created by compose.
* `umami` is also part of the `proxy` network created externally.
* The database schema file is bind-mounted in `/docker-entrypoint-initdb.d/`, meaning that it will be executed as SQL query when Postgres starts.
* The database data is persistently stored in the volume `umami-db-data`.

### `.env`

In the `.env` file we can place some environment variables related to the database credentials.

```
DATABASE_URL=postgresql://umami_db_user:umami_db_password@db:5432/umami_db_name
DATABASE_TYPE=postgresql
HASH_SALT=generate_a_random_salt

POSTGRES_DB=umami_db_name
POSTGRES_USER=umami_db_user
POSTGRES_PASSWORD=generate_a_strong_password
```

Some notes:

* The obvious: replace the values with the real ones.
* If you update the credentials, don't forget to also update the DSN specified in `DATABASE_URL`.

## Running the stack

At this point we can just spin up our containers and contextually see the logs:

```shell
docker-compose up -d && docker-compose logs -f
```

When the logs suggest us that the database has been migrated and Umami is ready, we can head up to `https://stats.oursite.ext` and login with the default credentials `admin:umami`.

To exit from the log tail, `CTRL+C`.

## Post-installation

The next steps are really trivial and simple:

1. Change the Umami credentials.
2. Add the site in Umami.
3. Get a tracking code.
4. Place the tracking code in the site.

Now as soon as we navigate in any page that contains the tracking code, we should instantly see our visit in the real-time view in Umami.

## Final thoughts

I'm positively surprised at how simple has been Umami to set up and get fully running. Previously I have been [self-hosting Matomo](../matomo-docker/), which in contrast feels bloated and slow to get up. Umami is like a breath of clean and fresh air.

Thanks for the read, bye :wave: