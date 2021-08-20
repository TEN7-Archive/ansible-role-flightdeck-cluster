# Flightdeck

![Flightdeck logo](flightdeck.svg)

Flightdeck is a small, minimal set of Docker containers to run PHP applications such as Drupal and Wordpress. You can run Flightdeck locally as a development environment, or in production on Docker Swarm or Kubernetes.

## Getting started

* [For local development, click here.](getting-started-locally.md)
* [For Kubernetes deployment, click here.](getting-started-k8s.md)

## Container Library

The core of Flightdeck is a library of Docker containers. Designed to run PHP applications such as Drupal and Wordpress, these containers are are small, flexible, and provide the tools needed to run your site either locally, online in a staging environment, or in production.

| Service name | Versions | Provides |
| ------------ | -------- | -------- |
| web | [7.4](https://hub.docker.com/r/ten7/flightdeck-web-7.4/), [7.3](https://hub.docker.com/r/ten7/flightdeck-web-7.3/) | PHP, Apache, NPM, Drush, and other CLI tools |
| db | [10.4](https://hub.docker.com/r/ten7/flightdeck-db-10.4/), [10.3](https://hub.docker.com/r/ten7/flightdeck-db-10.3/) | MariaDB (MySQL compatible)
| solr | [8.6](https://hub.docker.com/r/ten7/flightdeck-solr-8.6/), [6.6](https://hub.docker.com/r/ten7/flightdeck-solr-6.6/) | Apache Solr search engine |
| varnish | [6.4](https://hub.docker.com/r/ten7/flightdeck-varnish-6.4/) | Varnish caching reverse proxy |

See the links under **Versions** above for container specific documentation and source.

### Recommended additional containers

This containers aren't part of Flightdeck, but work well with it:

| Service name | Pull URL | Provides |
| ------------ | -------- | -------- |
| pma | [phpmyadmin/phpmyadmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/) | phpMyAdmin database GUI |
| mailhog | [mailhog/mailhog](https://hub.docker.com/r/mailhog/mailhog/) | Mail catcher for testing |
| memcached | [memcached:1.6-alpine](https://hub.docker.com/_/memcached/) | Caching object store |

## Customizing the web container

You can choose to customize the web container with your own custom application. A sample of that is available as [github.com/ten7/flightdeck-drupal](https://github.com/ten7/flightdeck-drupal)

## Deployment on Kubernetes

Flightdeck has been in use as a production environment on Kubernetes since 2018. To deploy Flightdeck, it is recommended to use [Flightdeck Cluster](https://github.com/ten7/ansible-role-flightdeck-cluster), an Ansible role used to set up cluster-wide, and site specific services.

## Support

You can always [post an issue](https://github.com/ten7/flightdeck.ten7.com/issues/new) on our Github page for general issues. If you have an issue or a feature request for a specific container, please see that container's git repository.

## License

See [LICENSE](https://raw.githubusercontent.com/ten7/flightdeck.ten7.com/master/LICENSE).
