---
title: Run calibre-web in Docker
description: Manage your ebook collection. Like a BOSS.
recipe: Calibre-Web
---

# Calibre-Web in Docker

The [AutoPirate](/recipes/autopirate/) recipe includes [Lazy Librarian](https://github.com/itsmegb/LazyLibrarian), a tool for tracking, finding, and downloading eBooks. However, after the eBooks are downloaded, Lazy Librarian is not much use for organising, tracking, and actually **reading** them.

[Calibre-Web](https://github.com/janeczku/calibre-web) could be described as "_[Plex](/recipes/plex/) (or [Emby](/recipes/emby/)) for eBooks_" - it's a web-based interface to manage your eBook library, screenshot below:

![Calibre-Web Screenshot](../images/calibre-web.png){ loading=lazy }

Of course, you probably already manage your eBooks using the excellent [Calibre](https://calibre-ebook.com/), but this is primarily a (_powerful_) desktop application. Calibre-Web is an alternative way to manage / view your existing Calibre database, meaning you can continue to use Calibre on your desktop if you wish.

As a long-time Kindle user, Calibre-Web brings (among [others](https://github.com/janeczku/calibre-web)) the following features which appeal to me:

* Filter and search by titles, authors, tags, **series** and language
* Create custom book collection (shelves)
Support for editing eBook metadata and deleting eBooks from Calibre library
* Support for converting eBooks from EPUB to Kindle format (mobi/azw)
* Send eBooks to Kindle devices with the click of a button
* Support for reading eBooks directly in the browser (.txt, .epub, .pdf, .cbr, .cbt, .cbz)
* Upload new books in PDF, epub, fb2 format

## {{ page.meta.recipe }} Requirements

--8<-- "recipe-standard-ingredients.md"

## Preparation

### Setup data locations

We'll need a directory to store some config data for Calibre-Web, container, so create /var/data/calibre-web, and ensure the directory is owned by the same use which owns your Calibre data (below)

```bash
mkdir /var/data/calibre-web
chown calibre:calibre /var/data/calibre-web # for example
```

Ensure that your Calibre library is accessible to the swarm (_i.e., exists on shared storage_), and that the same user who owns the config directory above, also owns the actual calibre library data (_including the ebooks managed by Calibre_).

### Prepare {{ page.meta.recipe }} environment

Create `/var/data/config/calibre-web/calibre-web.env`, and populate with the following variables

```bash

PUID=
PGID=
```

### {{ page.meta.recipe }} Docker Swarm config

Create a docker swarm config file in docker-compose syntax (v3), something like the example below:

--8<-- "premix-cta.md"

```yaml
version: '3'

services:
  app:
    image: technosoft2000/calibre-web
    env_file : /var/data/config/calibre-web/calibre-web.env
    volumes:
     - /var/data/calibre-web:/config
     - /srv/data/Archive/Ebooks/calibre:/books
    networks:
      - internal
      - traefik_public
    deploy:
      labels:
        # traefik common
        - traefik.enable=true
        - traefik.docker.network=traefik_public

        # traefikv1
        - traefik.frontend.rule=Host:calibre.example.com
        - traefik.port=8083     

        # traefikv2
        - "traefik.http.routers.calibre.rule=Host(`calibre.example.com`)"
        - "traefik.http.services.calibre.loadbalancer.server.port=8083"
        - "traefik.enable=true"

        # Remove if you wish to access the URL directly
        - "traefik.http.routers.calibre.middlewares=forward-auth@file"

networks:
  traefik_public:
    external: true
  internal:
    driver: overlay
    ipam:
      config:
        - subnet: 172.16.18.0/24
```

--8<-- "reference-networks.md"

## Serving

### Launch Calibre-Web

Launch the Calibre-Web stack by running ```docker stack deploy calibre-web -c <path -to-docker-compose.yml>```

Log into your new instance at `https://**YOUR-FQDN**`. You'll be directed to the initial GUI configuraition. Set the first field (_Location of Calibre database_) to "_/books/_", and when complete, login using defaults username of "**admin**" with password "**admin123**".

[^1]: Yes, Calibre does provide a server component. But it's not as fully-featured as Calibre-Web (_i.e., you can't use it to send ebooks directly to your Kindle_)
[^2]: A future enhancement might be integrating this recipe with the filestore for [NextCloud](/recipes/nextcloud/), so that the desktop database (Calibre) can be kept synced with Calibre-Web.
[^3]: If you plan to use calibre-web to send `.mobi` files to your Kindle via `@kindle.com` email addresses, be sure to add the sending address to the "[Approved Personal Documents Email List](https://www.amazon.com/hz/mycd/myx#/home/settings/payment)"
--8<-- "recipe-footer.md"
