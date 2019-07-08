# Ansible Role: Flight Deck Cluster

Creates a [Flight Deck](https://github.com/ten7/flight-deck) flavored cluster on Kubernetes.

## Requirements

* All requirements of the k8s module.
* You either need a kubeconfig file, or set appropriate environment variables when you import the role.

## Specifying credentials

`flightdeck_cluster_kubeconfig`

Specifies the path to a kubeconfig file used for authentication.

## Defining the cluster

`flightdeck_cluster`

This dictionary describes the k8s cluster to create, including secrets, services, and other options.

The `flightdeck_cluster` dictionary as the following top-level items:

### Specifying the namespace

```yaml
flightdeck_cluster:
  namespace: "example-com"
```

The `namespace` items specifies the namespace in which to create cluster defintions. If the namespace doesn't exist, it will be created. If not specified, definitions will be created in the `default` namespace.

### Secrets and configmaps

This role can define new k8s secrets and config maps via the `flightdeck_cluster.secrets` and `flightdeck_cluster.configMaps` keys.

The `flightdeck_cluster.secrets` list has two forms, the first is useful for key/value data:

```yaml
flightdeck_cluster:
  secrets:
    - name: "mysecret"
      type: "Opaque"
      data:
        - name: "alpha"
          value: "1234567890abscdefg"
```

Where:

* **name** is the name of the secret.
* **type** is the type of the secret. Optional, defaults to `Opaque`.
* **data** is a list of key/value data to store in the secret.

Sometimes you need to store entire configuration files in a k8s secret, for that, you can use the *files* form:

```yaml
flightdeck_cluster:
  secrets:
    - name: "filesecret"
      files:
        - name: "my-secret.yml"
          content: |
            my_key_name: "my value"
            my_other_key_name: "my other value"
```

For larger config files that have non-sensitive data, you can use `flightdeck_cluster.configMaps`. It has the same format as the *files* form:

```yaml
flightdeck_cluster:
  configMaps:
    - name: "mysql_config"
      files:
        - name: "my.cnf"
          content: |
            bind-address: 0.0.0.0
```

### The web service

The web service combines the Flight Deck web container and the varnish container into a single pod.

```yaml
flightdeck_cluster:
  web:
    state: present
    replicas: 3
```

* **state**: Optional. If the service is `present` or `absent`, defaults to `present` when `flightdeck_cluster.web` is defined.
* **replicas**: Optional, but recommended. The number of web replicas to create. Defaults to 3.

#### Using custom images

By default, the `ten7/flight-deck-web` container is used as the web server. This is the default, but it's not the most useful. To deploy your site, you should use a [private container registry](https://docs.docker.com/registry/deploying/) that includes your site files as part of the image.

To do this, define the following:

```yaml
flightdeck_cluster:
  web:
    image: "myRegistry.tld:5000/my_custom_image:tag"
    imagePullSecrets:
      - "registry"
```

Where:

* **image**: Optional. The image to use for the web server container. Defaults to `ten7/flight-deck-web`.
* **imagePullSecrets** Optional. A list of secret names containing registry credentials. Required when `image` is on a private container registry.

#### Mounting secrets and config

Often you will need to mount credentials as a k8s secret, or configuration as a config map. Once these are defined under the `secrets` and `configMaps` keys respectively, you can mount them as volumes by defining the `secrets` or `configMaps` keys under the `web key.

The `flightdeck_cluster.web.secrets` and `flightdeck_cluster.web.configMaps` accept zero or more items with the following structure:

```yaml
flightdeck_cluster:
  web:
    secrets:
      - name: "dblogin"
        volumeName: "vol-db"
        path: "/secrets"
    configMaps:
      - name: "myconfig"
        volumeName: "vol-my-config"
        path: "/config"
```

Where:

* **name** is the name of the secret or config map.
* **volumeName** is the name of the volume. Optional. Defaults to the secret or config name prefixed by `vol-secet-` or `vol-config-` respectively.
* **path** is the parent directory to which to mount the secret or configMap.

#### Configuring Varnish

The varnish container may also be configured under `flightdeck_cluster.web.varnish`:

```yaml
flightdeck_cluster:
  web:
    varnish:
      memory: "256M"
      storage: "1024M"
```

Where:

* **memory** is the amount of memory to allocate to the container. Optional, default is 256MB.
* **storage** is the amount of disk to allocate for caching. Optional, default is 1GB.

#### Specifying placement

This role can use a node selector to place containers on particular nodes in the cluster using the `nodeSelector` item:

```yaml
flightdeck_cluster:
  web:
    nodeSelector:
      key: "mylabel"
      value: "myvalue"
```

Where:

* **nodeselector.key** is the name of the label to match against nodes.
* **nodeselector.value** is the label value to match against nodes.

### The memcache service

The `flightdeck_cluster.memcache` item describes the memcache service to create in the cluster.

```yaml
flightdeck_cluster:
  memcache:
    state: present
    replicas: 3
    image: 'memcached:1.5-alpine'

```

Where:

* **state**: Optional. If the service is `present` or `absent`, defaults to `present` when `flightdeck_cluster.memcache` is defined.
* **replicas**: Optional, but recommended. The number of memcache replicas to create. Defaults to 3.
* **image**: Optional. The image to use for the memcache service. Defaults to the official memecache image.

### MySQL database

If you do not have an existing MySQL database service, you may choose to use the one provided by this role by defining the `mysql` key:

```yaml
flightdeck_cluster:
  mysql:
    image: "ten7/flight-deck-db:10"
    size: "10Gi"
```

Where:

* **state** specifies if the backup service is `present` or `absent`. Optional, defaults to `present` when `flightdeck_cluster.mysql` is defined.
* **image** is the image to use. Optional, defaults to `ten7/flight-deck-db:10`.
* **nodeSelector** is the key/value pair to use to place the pod in the cluster. Optional, works like the `web` service.
* **size** is the size of the Persistent Volume to attach to the service.
* **secrets** are the secrets to mount in the container. Optional. Works like the `web` service.
* **configMaps** are the configMaps to mount in the container. Optional. Works like the `web` service.

### Tractorbeam backups

This role has support for adding [tractorbeam](https://github.com/ten7/tractorbeam) backups using the `ten7/tractorbeam` container. Define the `tractorbeam` key:

```yaml
flightdeck_cluster:
  tractorbeam:
    state: present
    image: "ten7/tractorbeam:latest"
    dailySchedule: "0 0 * * *"
    weeklySchedule: "0 2 * * 0"
    monthlySchedule: "0 4 1 * *"
    secrets:
      - name: "my-tractorbeam-yml-secret"
```

Where:

* **state** specifies if the backup service is `present` or `absent`. Optional, defaults to `present` when `flightdeck_cluster.tractorbeam` is defined.
* **image** is the image to use for the backup cronjobs. Optional, defaults to `ten7/tractorbeam:latest`.
* **nodeSelector** is the key/value pair to use to place the pod in the cluster. Optional, works like the `web` service.
* **dailySchedule** is the crontab formatted schedule on which to run the daily backup. Optional. Defaults to 12am UTC every day.
* **weeklySchedule** is the crontab formatted schedule on which to run the weekly backup. Optional. Defaults to 2am URT every Sunday.
* **monthlySchedule** is the crontab formatted schedule on which to run the monthly backup. Optional. Defaults to 4am UTC on the first of each month.
* **secrets** are the secrets to mount in the container. Optional. Works like the `web` service.
* **configMaps** are the configMaps to mount in the container. Optional. Works like the `web` service.

See the [tractorbeam docs](https://github.com/ten7/tractorbeam) for how to use and configure this service.

### Solr

You can provision a Solr image using this role. To enable, create the `flightdeck_cluster.solr` item:

```yaml
flightdeck_cluster:
  solr:
    state: present
    image: "ten7/flight-deck-solr:6"
    nodeSelector:
      key: "app"
      value: "search"
    size: "10Gi"
```

Where:

* **state** specifies if the backup service is `present` or `absent`. Optional, defaults to `present` when `flightdeck_cluster.solr` is defined.
* **image** is the image to use for the Solr container. Optional, defaults to `ten7/flight-deck-solr:6`.
* **nodeSelector** is the key/value pair to use to place the pod in the cluster. Optional, works like the `web` service.
* **size** is the size of Persistent Volume to attach to the Solr container.
* **secrets** are the secrets to mount in the container. Optional. Works like the `web` service.
* **configMaps** are the configMaps to mount in the container. Optional. Works like the `web` service.

### Cert Manager for Let's Encrypt

This role also can provision [Cert Manager](https://docs.cert-manager.io/en/latest/index.html) with [Let's Encrypt](https://letsencrypt.org/) support. The Cert Manager will auto-renew provisioned certificates.

```yaml
flightdeck_cluster:
  certManager:
    state: present
```

Where:

* **state** specifies if the ingress controller is `present` or `absent`. Optional, defaults to `absent` when `flightdeck_cluster.certManager` is defined.

#### Issuers

This alone does not enable Let's Encrypt. To do that, you need to define one or more "issuers" under the `letsEncrypt` item:

```yaml
flightdeck_cluster:
  certManager:
    state: present
    letsEncrypt:
      - name: "my-lets-encrypt-prod-issuer"
        email: "ops@Example.com"
        secret: "lets-encrypt-private-key"
        server: "https://acme-v02.api.letsencrypt.org/directory"
```

Where:

* **name** is the name of the issuer to use internally. Required.
* **email** is the email address to use for the issuer. Required.
* **secret** is the secret name to use to store the private key. The secret will be automatically created if it doesn't exist. Optional. Defaults to the issuer name.
* **server** The Let's Encrypt API server to use to provision certificates. Optional, defaults to the prod server. Use `https://acme-staging-v02.api.letsencrypt.org/directory` for creating staging and testing domains.

Having multiple issuers allows you to segment notification emails, as well as use different API servers depending on use.

#### Namespaces and Cert Manager

Ideally, you shouldn't use the same namespace for Cert Manager as you do your other cluster services. Instead, create a play to define the cert manager in its own namespace using `include_role`, using `vars` to override key values at the play level:

```yaml
- name: Create the cert-manager
  include_role:
    name: "ten7.flightdeck_cluster"
  vars:
    flightdeck_cluster_kubeconfig: "/path/to/my/kubectl.yaml"
    flightdeck_cluster:
      namespace: "cert-manager"
      certManager:
        state: present
        letsEncrypt:
          - name: "my-lets-encrypt-prod-issuer"
            email: "ops@Example.com"
            secret: "lets-encrypt-private-key"
            server: "https://acme-v02.api.letsencrypt.org/directory"
```

Alternatively, create a separate playbook entirely to deploy Cert Manager.

### Configuring Ingress rules

This role provides the ability to manage ingress definitions. It does *not* provide an ingress controller.

To manage the ingress definition, create the `flightdeck_cluster.ingress` item:

```yaml
flightdeck_cluster:
  ingress:
    rules:
      - host: "example.com"
        paths:
          - path: "/"
            backend: "web"
            port: "6081"
```

The `flightdeck_cluster.ingress.rules` key is a list routing rules for your cluster.

Each item in the list has the following items:

* **host** is hostname (including the subdomain) to which the rule applies.
* **paths** is a list of HTTP request paths to which the rule applies. Optional.

Each item in the `paths` list has the following:

* **path** is the URL request path for which the rule applies. Optional, default is `/`.
* **backend** is the name of the k8s service to which to route traffic. Optional, default is `web`.
* **port** is the port to which to route traffic. Optional, default is `6081`.

#### HTTPS and certificates

Many ingress controllers will terminate HTTPS, allowing traffic to be unencrypted internally to the cluster.

Certificate chains can be created as secrets then used by defining `flightdeck_cluster.ingress.tls`.

```yaml
flightdeck_cluster:
  ingress:
    tls:
      - secret: "mycertchain"
        hosts:
          - "example.com"
```

Each item under `flightdeck_cluster.ingress.tls` has the following items:

* **secret** is the secret name containing the certificate chain.
* **hosts** is a list of hosts, including subdomains, for which to use the cert.

#### Using Cert Manager for Let's Encrypt

If you've defined `flightdeck_cluster.certManager` and configured one or more Let's Encrypt issuers, you can use them to provision non-wildcard certificates:

```yaml
flightdeck_cluster:
  ingress:
    tlsIssuer: "my-lets-encrypt-prod-issuer"
    tls:
      - secret: "lets-encrypt-private-key"
        hosts:
          - "example.com"
```

The `tlsIssuer` item instructs the ingress defition to use an issuer you created in `flightdeck_cluster.certManager.letsEncrypt`. The `secret` for each `tls` item must match the secret name for the given issuer. Note, that if you didn't define a `secret` for the issuer, the issuer name is used by default.

#### Tuning ingress

```yaml
flightdeck_cluster:
  ingress:
    maxBodySize: "50m"
```

* **maxBodySize** is the max HTTP POST size accepted by the ingress. Optional, defaults to `128m`.

### Ingress Controller

When self-hosting and on some managed Kubernetes providers, you may require an *ingress controller*. This isn't the same as an ingress *definition* such as above. Instead, this is a load balancer which listens for new ingress definitions. This role relies on the community supported [nginx-ingress controller](https://github.com/kubernetes/ingress-nginx).

Be sure to consult your provider's documentation before creating an ingress controller, as each may have their own preferences on implementation.

To create an ingress controller, define the `ingressController` key:

```yaml
flightdeck_cluster:
  ingressController:
    state: present
    name: "nginx-ingress"
    ports:
      - name: "http"
        port: "80"
        targetPort: "http"
```

Where:

* **state** specifies if the ingress controller is `present` or `absent`. Optional, defaults to `present` when `flightdeck_cluster.ingressController` is defined.
* **name** is the name of the ingress controller to create. Optional, defaults to `nginx-ingress`.
* **ports** is a list of external port mappings, including a `name`, the `port` number, and the `targetPort` name. Optional, defaults to exposing the default HTTP and HTTPS ports.

### Cron

This role also can leverage Kubernetes cronjobs by using the `cron` key. Unlike many other keys, this one takes one or more items, each with the following format:

```yaml
cron:
  - name: "my-cron"
    schedule: "0 */3 * * *"
    image: "myRegistry.tld:5000/my_custom_image:tag"
    command:
      - "mycommand"
    args:
      - "--do=stuff"
    imagePullSecrets:
      - "registry"
    secrets:
      - name: "dblogin"
```

Where:
* **state** specifies if the backup service is `present` or `absent`. Optional, defaults to `present` when `flightdeck_cluster.tractorbeam` is defined.
* **image** is the image to use for the backup cronjobs. Optional, defaults to `ten7/tractorbeam:latest`.
* **nodeSelector** is the key/value pair to use to place the pod in the cluster. Optional, works like the `web` service.
* **schedule** is the crontab formatted schedule on which to run the job. Required.
* **command** is the command (entrypoint) to run in the pod. Optional.
* **args** are the arguments to pass to the command. Optional.
* **secrets** are the secrets to mount in the container. Optional. Works like the `web` service.
* **configMaps** are the configMaps to mount in the container. Optional. Works like the `web` service.

## Dependencies

None, but the following roles are recommended:

* All requirements for the k8s module.
* [socketwench/digitalocean](https://galaxy.ansible.com/socketwench/digitalocean)
* [socketwench/digitalocean_kubeconfig](https://galaxy.ansible.com/socketwench/digitalocean_kubeconfig)

## Example Playbook

    ---
    - hosts: k8s
      vars:
        digitalocean_api_token: '1234567890abscdefg'
        digitalocean_projects:
          - name: 'MYPROJECT'
        digitalocean_clusters:
          - name: "my-k8s"
            region: "sfo2"
            version: "1.13.2-do.1"
            tags:
              - "live"
            node_pools:
              - name: "default-pool"
                state: present
                size: "s-1vcpu-2gb"
                count: 3
        digitalocean_kubeconfig:
          cluster: "my-k8s"
          kubeconfig: "/path/to/my/kubectl.yaml"
        flightdeck_cluster_kubeconfig: "/path/to/my/kubectl.yaml"
        flightdeck_cluster:
          namespace: "example-com"
          web:
            replicas: 3
          memcache:
            replicas: 3
          ingress:
            tls:
              - secret: "mycertchain"
                hosts:
                  - "example.com"
          rules:
            - host: "example.com"
              paths:
                - path: "/"
                  backend: "web"
                  port: "6081"
      roles:
        - socketwench.digitalocean
        - socketwench.digitalocean_kubeconfig

## License

GPL v3

## Author Information

This role was created by [socketwench](https://deninet.com/) for [TEN7](https://ten7.com).
