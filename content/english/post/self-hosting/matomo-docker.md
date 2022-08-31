---
author: Davide Quaranta
title: Self-hosting Matomo with Docker
date: 2020-04-07T00:00:00Z
categories: [Self-Hosting]
tags: [devops, sysadmin, matomo, analytics, docker]
description: Matomo is a viable alternative to Google Analytics as an ethical and privacy-oriented web analytics tool. In this post we see how to install it with Docker, alongside MariaDB and a couple of nginx-related containers.
---

**Matomo** is a web analytics platform, which allows us to obtain statistics about the visitors and content of our websites.

What distinguishes Matomo (but also other similar platforms) from the big and _easy_ Google Analytics, is the fact that with the former we have **total control and ownership of the data**.

In this article we see how to **self-host** Matomo, with a setup consisting of:

- Docker (and Compose);
- [Matomo On-Premise](https://hub.docker.com/_/matomo);
- [MariaDB](https://hub.docker.com/_/mariadb/);

We also assume that we already have:

- A container with an active web project (e.g., Apache, Ghost, WordPress);
- [jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/);
- An `nginx-proxy` network;
- [jrcs/letsencrypt-nginx-proxy-companion](https://hub.docker.com/r/jrcs/letsencrypt-nginx-proxy-companion), to magically manage SSL certificates.

## What we want

Suppose we have a web project on `mysite.ext`. We want to shove Matomo into a subdomain `stats.mysite.ext`.

To do this we trivially need to:

- Configure a DNS record of type CNAME that makes the subdomain an alias of the main domain.
- Install Matomo and expose it to `https://stats.mysite.ext`.

After creating that record, we move on to the second step.

## Preparation

We create a `~/matomo` folder with in it:

### docker-compose.yml

```yaml
version: "2"

services:
  matomo:
    container_name: matomo
    image: matomo
    ports:
      - 8080:80
    environment:
      - MATOMO_DATABASE_HOST=matomo_db
      - VIRTUAL_HOST=stats.mysite.ext
      - LETSENCRYPT_HOST=stats.mysite.ext
      - LETSENCRYPT_EMAIL=email@someting.ext
    env_file:
      - ./db.env
    networks:
      - proxy
      - net
    depends_on:
      - matomo_db
    restart: unless-stopped

  matomo_db:
    container_name: matomo_db
    image: mariadb
    command: --max-allowed-packet=64MB
    environment:
      - MYSQL_ROOT_PASSWORD=inventa
    env_file:
      - ./db.env
    networks:
      - net
    restart: unless-stopped

networks:
  proxy:
    external:
      name: nginx-proxy
  net:
    driver: bridge
```

Notable stuff:

- We attached `matomo` to the existing `nginx-proxy` network, on which our web project also runs.
- We have defined a new `net`, in which both `matomo` and `matomo_db` are present, but we do not need the latter to be part of `nginx-proxy`.
- The environment variables `LETSENCRYPT_*` are needed for `letsencrypt-nginx-proxy-companion`.
- We have defined a dependency (and thus a starting order) relationship between the two containers.

### db.env

```
MYSQL_PASSWORD=inventa2
MYSQL_DATABASE=matomo
MYSQL_USER=matomo
MATOMO_DATABASE_ADAPTER=mysql
MATOMO_DATABASE_TABLES_PREFIX=matomo_
MATOMO_DATABASE_USERNAME=matomo
MATOMO_DATABASE_PASSWORD=
MATOMO_DATABASE_DBNAME=matomo
```

As soon as we are ready `docker-compose up -d`.

We verify that our nice containers are running with `docker ps`. If all is well, let's move on.

We note that at this point we should already have an active, working SSL certificate for our subdomain.

## Installing Matomo

After running our stack, if we go to `stats.mysite.ext` we will be greeted by the Matomo installation screen.

We just go ahead to the DB configuration, which will already be compiled (thanks to the `MATOMO_*` environment variables), except for the DB password, which must be the value of `MYSQL_PASSWORD`.

After confirming, the DB connection should work and Matomo will install without any problems.

## Initial configuration of Matomo

Before we paste the tracking code into our site, we'd better configure report archiving; if we don't do this small thing we'll experience the thrill of a very slow site and _high high downtime_.

**What archiving is**: It is simply Matomo processing the collected data and making it visible to us.

By default, archiving is done _every so often_ and triggered by visits from our users. This means that among the requests to `stats.mysite.ext/matomo.php` there will be some that say "Hey Matomo, start archiving."

What we want is to disable this behavior and put control in our hands by configuring an automatic process that every _tot_ performs archiving.

### Archiving of reports

Simple:

1. We access `stats.mysite.ext` with the _SuperUser_ defined during installation;
2. We go to the System->**General Settings** section;
3. We set the browser-activated storage to **No**.

Now we need to configure an automatic _something_. We have a couple of ways:

- Define a [sidecar](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar) container called `matomo_cron`, with the same image as `matomo` and possibly the same volumes, with an `entrypoint` script that essentially does two things:
  - Sleeps for `n` seconds;
  - Runs the archiving script.
- Trivially define a [cron job](https://matomo.org/docs/setup-auto-archiving/) on the host system.

In this article we follow the second path, although a bit less elegant.

#### Cron Job

On the host system and with the user we normally run docker with, we run `crontab -e`, and paste this stuff:

```sh
5 * * * * if [ $(docker inspect -f '{{.State.Running}}' matomo) ]; then docker exec -t matomo on -s "/bin/bash" -c "/usr/local/bin/php /var/www/html/console core:archive --url=https://stats.mysite.ext" www-data; fi >> /home/user/logs/matomo-archive.log

```

Since we defined it with `crontab -e`, it will be executed by the current user, and the fields have (in order) this meaning: minutes, hours, days, months, days of the week, command.

If we break up our code:

- It is executed every hour at minute `05`;
- If `matomo` is running:
  - Runs the storage for the site, with the user (in the container) `www-data`;
- Spits out the output in `~/logs/matomo-archive.log`.

The final blank line is in the `cron` specification and its absence _may_ cause the script to fail.

However, I recommend testing the script right away and verify that everything is ok.

## Conclusion

At this point we can paste the tracking code into the site, and enjoy statistics:

- Homemade;
- Easily manageable in terms of privacy;
- Self-updating every hour.

If we want to be cool, we also download Matomo's mobile app and look at statistics from there as well.
