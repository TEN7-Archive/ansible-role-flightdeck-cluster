# Flight Deck

Flight deck is a small, minimal set of Docker containers to run PHP applications such as Drupal and Wordpress. You can run Flight Deck locally as a development environment, or in production on Docker Swarm or Kubernetes.

## Getting started locally

After installing Docker Desktop, you need to add two files you your project: `docker-compose.yml` and `flight-deck.yml`.

### Directory layout

Below is the recommended project layout for a site to run using Flight Deck.

```
├── config
│   ├── development
│   └── sync
├── db-backups
│   └── backup-1984-10-26T01-12-00.mysql.gz
├── docker-compose.override.yml.example
├── docker-compose.yml
├── flight-deck.yml
├── solr-conf
│   └── 8.x
└── src
    ├── core
    └── index.php
```

### The Compose file

The `docker-compose.yml` describes what containers to start up, their mountpoints, and exposed ports.

```yaml
version: '3'
services:
  web:
    image: ten7/flightdeck-web-7.4
    ports:
      - "80:80"
    volumes:
      - ./src:/var/www/html:cached
      - ./config:/var/www/config:cached
      - ./db-backups:/var/www/db-backups:cached
      - ./flight-deck.yml:/config/web/flight-deck-web.yml
  db:
    image: ten7/flightdeck-db-10.3
    ports:
      - 3306:3306
    volumes:
      - /var/lib/mysql
      - ./db-backups:/tmp/db-backups:cached
      - ./flight-deck.yml:/secrets/flight-deck-db.yml
  memcached:
    image: memcached:1.5-alpine
  pma:
    image: phpmyadmin/phpmyadmin
    environment:
      PMA_HOST: db
      PMA_USER: root
      PMA_PASSWORD: root
      PHP_UPLOAD_MAX_FILESIZE: 1G
      PHP_MAX_INPUT_VARS: 1G
    ports:
     - "8001:80"
  mailhog:
    image: mailhog/mailhog
    ports:
      - "8002:8025"
  solr:
    image: ten7/flightdeck-solr-8
    volumes:
      - ./solr-conf/8.x:/solr-conf
      - ./flight-deck.yml:/config/solr/flight-deck-solr.yml
    ports:
      - "8003:8983"
  varnish:
    image: "ten7/flightdeck-varnish-6.4"
    depends_on:
      - web
    volumes:
      - ./flight-deck.yml:/config/varnish/flight-deck-varnish.yml
    ports:
      - "8004:6081"
      - "8005:6082"

```

### Configuration

Flight Deck relies on YAML to configure each container on startup. These can be separate files, or combined into one large file:

```yaml
---
flightdeck_debug: no
flightdeck_solr:
  port: 8983
  cores:
    - name: "mycore"
      type: "custom"
      confDir: "/solr-conf"
mysql_root_password: "root"
mysql_allow_remote_root: yes
mysql_databases:
  - name: "drupal"
    encoding: "latin1"
    collation: "latin1_general_ci"
mysql_users:
  - name: "drupal"
    host: "%"
    password: "drupal"
    priv: "drupal.*:ALL"
mysql_key_buffer_size: "256M"
mysql_max_allowed_packet: "64M"
mysql_table_open_cache: "256"
mysql_query_cache_size: "0"
flightdeck_varnish:
  secret: "abcdef12345"
  memSize: "256m"
  backends:
    - name: "default"
      host: "web"
      probe: no
      probeHost: "web"
      probeHeaders: []
flightdeck_web:
  vhosts:
    - name: "docker.test"
      aliases:
        - "docker.test"
      env:
        - name: "T7_SITE_ENVIRONMENT"
          value: "docker"
  php:
    upload_max_filesize: "128M"
    post_max_size: "128M"
    opcache_revalidate_freq: "1"

```

### Local URL

While you can access Flight Deck locally using `localhost`, it is highly recommended to create a URL alias instead.

For Linux and Mac users, you can edit `/etc/hosts`.

For Windows users, edit `C:\Windows\System32\drivers\etc\hosts`

No matter if you're using Linux, Mac, or Windows, the configuration to add is the same:

```
127.0.0.1  docker.test
```

You may choose to add unique aliases for each Flight Deck powered project you work with:

```
127.0.0.1  docker.test
127.0.0.1  marty.test
127.0.0.1  docbrown.test
127.0.0.1  biff.test
```

It is **highly** recommended to use the `.test` TLD, as it is by definition used for local alias and network testing.

## The Containers

### Web

* Image: [`ten7/flightdeck-web-7.4`](https://hub.docker.com/repository/docker/ten7/flightdeck-web-7.4)
* Source and docs: [github.com/ten7/flightdeck-web-7.4](https://github.com/ten7/flightdeck-web-7.4)
* Other versions available: [`flightdeck-web-7.3`](https://hub.docker.com/repository/docker/ten7/flightdeck-web-7.3)

The primary container in Flight Deck is the `web` container. It has Apache and PHP installed. Furthermore, to support local development is comes with Drush, NPM, Gulp, and Composer.  

To access these utilities, open a command prompt. Change to your project directory, and enter into the container using:

```shell
docker-compose exec web bash
```

### DB

* Image: [`ten7/flightdeck-db-10.4`](https://hub.docker.com/repository/docker/ten7/flightdeck-db-10.4)
* Source and docs: [github.com/ten7/flightdeck-db-10.4](https://github.com/ten7/flightdeck-db-10.4)
* Other versions available: [`flightdeck-db-10.3`](https://hub.docker.com/repository/docker/ten7/flightdeck-db-10.3)

The `db` container provides a MySQL compatible database to power your application. You can access it in several ways:

* A MySQL client such as SequelPro on your host machine.
* The included phpMyAdmin instance on `docker.test:8001`
* Inside the either the `web` or `db` containers, using the `mysql` command line utility.

By default, the `db` container creates one database and one database user, but you may configure multiple databases and accounts in `flight-deck.yml`.

### Solr

* Image: [`ten7/flightdeck-solr-8`](https://hub.docker.com/repository/docker/ten7/flightdeck-solr-8)
* Source and docs: [github.com/ten7/flightdeck-solr-8](https://github.com/ten7/flightdeck-solr-8)
* Other versions available: [`flightdeck-solr-6`](https://hub.docker.com/repository/docker/ten7/flightdeck-solr-6)

The `solr` container provides a Solr search server. Like `db`, you can create multiple cores by configuring `flight-deck.yml`.

### Varnish

* Image: [`ten7/flightdeck-varnish-6.4`](https://hub.docker.com/repository/docker/ten7/flightdeck-varnish-6.4)
* Source and docs: [github.com/ten7/flightdeck-varnish-6.4](https://github.com/ten7/flightdeck-varnish-6.4)

The `varnish` container provides a caching reverse proxy. This isn't necessary for local development. It is, however, highly recommended for production environments.

## Customizing the web container

You can choose to customize the web container with your own custom application. A sample of that is available as [github.com/ten7/flightdeck-drupal](https://github.com/ten7/flightdeck-drupal)

## Deployment on Kubernetes

Flight Deck has been in use as a production environment on Kubernetes since 2018. To deploy Flight Deck, it is recommended to use [Flight Deck Cluster](https://github.com/ten7/ansible-role-flightdeck-cluster), an Ansible role used to set up cluster-wide, and site specific services.
