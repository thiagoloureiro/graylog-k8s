# Graylog

This chart provide the [Graylog][1] deployments.
Note: It is strongly recommend to use on Official Graylog image to run this chart.

## Quick Installation

This chart requires the following charts before install Graylog

1. MongoDB
2. Opensearch

To install the Graylog Chart with all dependencies

```bash
kubectl create namespace graylog

helm install --namespace "graylog" graylog graylog-k8s/graylog
```

## Manually Install Dependencies

This method is _recommended_ when you want to expand the availability, scalability, and security of the services. You need to install MongoDB replicaset and Opensearch with proper settings before install Graylog.

To install MongoDB, run

```bash
helm install --namespace "graylog" mongodb bitnami/mongodb
```

Note: There are many alternative MongoDB available on [artifacthub.io](https://artifacthub.io/packages/search?page=1&ts_query_web=mongodb). If you found the `bitnami/mongodb` is not suitable, you can use another MongoDB chart. Modify `graylog.mongodb.uri` to match your MongoDB endpoint.

To install Opensearch, run

```bash
helm install --namespace "graylog" opensearch elastic/opensearch
```

The Opensearch installation command above will install all Opensearch
nodes types in single node. It is strongly recommend to follow the Opensearch [guide](https://github.com/elastic/helm-charts/tree/main/epensearch#how-to-deploy-dedicated-nodes-types) to install dedicated node on production.

Note: There are many alternative Opensearch available on [artifacthub.io](https://artifacthub.io/packages/search?page=1&ts_query_web=Opensearch). If you found the `stable/opensearch` is not suitable, you can search other charts from GitHub repositories.

## Install Chart

To install the Graylog Chart into your Kubernetes cluster (This Chart requires persistent volume by default, you may need to create a storage class before install chart.

```bash
helm install --namespace "graylog" graylog graylog-k8s/graylog \
  --set tags.install-mongodb=false\
  --set tags.install-opensearch=false\
  --set graylog.mongodb.uri=mongodb://mongodb-mongodb-replicaset-0.mongodb-mongodb-replicaset.graylog.svc.cluster.local:27017/graylog?replicaSet=rs0 \
  --set graylog.opensearch.hosts=http://opensearch-cluster-master-headless.graylog.svc.cluster.local:9200
  --set graylog.opensearch.version=7
```

After installation succeeds, you can get a status of Chart

```bash
helm status graylog
```

If you want to delete your Chart, use this command

```bash
helm delete graylog
```

## Install Chart with specific Graylog cluster size

By default, this Chart will create a graylog with 2 nodes (1 master, 1 coordinating). If you want to change the cluster size during installation, you can use `--set graylog.replicas={value}` argument. Or edit `values.yaml`

For example:
Set cluster size to 5

```bash
helm install --namespace "graylog" graylog --set graylog.replicas=5 kongz/graylog
```

The command above will install 1 master and 4 coordinating.

## Install Chart with specific node pool

Sometime you may need to deploy your graylog to specific node pool to allocate resources.

### Using node selector

For example, you have 6 vms in node pools and you want to deploy graylog to node which labeled as `cloud.google.com/gke-nodepool: graylog-pool`
Set the following values in `values.yaml`

```yaml
graylog:
  nodeSelector: { cloud.google.com/gke-nodepool: graylog-pool }
```

### Using tolerations

For example, you have 6 vms in node pools and 3 nodes are tainted with `NO_SCHEDULE graylog=true`
Set the following values in `values.yaml`

```yaml
graylog:
  tolerations:
    - key: graylog
      value: "true"
      operator: "Equal"
```

## Configuration

The following table lists the configurable parameters of the Graylog chart and their default values.

| Parameter                                         | Description                                                                                                                                     | Default                           |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `graylog.image.repository`                        | `graylog` image repository                                                                                                                      | `graylog/graylog`                 |
| `graylog.image.tag`                               | `graylog` image tag                                                                                                                             | `4.2.7-1`                         |
| `graylog.image.pullPolicy`                        | Image pull policy                                                                                                                               | `IfNotPresent`                    |
| `graylog.replicas`                                | The number of Graylog instances in the cluster. The chart will automatic create assign master to one of replicas                                | `2`                               |
| `graylog.resources`                               | CPU/Memory resource requests/limits                                                                                                             | Memory: `1024Mi`, CPU: `500m`     |
| `graylog.heapSize`                                | Override Java heap size. If this value empty, chart will allocate heap size using `-XX:+UseCGroupMemoryLimitForHeap`                            |                                   |
| `graylog.externalUri`                             | External URI that Graylog is available at                                                                                                       |                                   |
| `graylog.nodeSelector`                            | Graylog server pod assignment                                                                                                                   | `{}`                              |
| `graylog.affinity`                                | Graylog server affinity                                                                                                                         | `{}`                              |
| `graylog.tolerations`                             | Graylog server tolerations                                                                                                                      | `[]`                              |
| `graylog.nodeSelector`                            | Graylog server node selector                                                                                                                    | `{}`                              |
| `graylog.env`                                     | Graylog server env variables                                                                                                                    | `{}`                              |
| `graylog.customLabels`                            | Graylog additional labels for statefulset/pods                                                                                                  | `{}`                              |
| `graylog.envRaw`                                  | Graylog server env variables in raw yaml format                                                                                                 | `{}`                              |
| `graylog.podSecurityContext`                      | Set security context for defining privilege and accessing control settings entire Pod                                                           | `{}`                              |
| `graylog.securityContext`                         | Set security context for defining privilege and accessing control settings for Graylog container                                                | `privileged: false`               |
| `graylog.javaOpts`                                | Graylog service `JAVA_OPTS`                                                                                                                     |                                   |
| ~~graylog.additionalJavaOpts~~                    | Graylog service additional `JAVA_OPTS` (Replaced with `graylog.javaOpts`)                                                                       |                                   |
| `graylog.service.type`                            | Kubernetes Service type                                                                                                                         | `ClusterIP`                       |
| `graylog.service.port`                            | Graylog Service port                                                                                                                            | `9000`                            |
| `graylog.service.ports`                           | Graylog Service extra ports                                                                                                                     | `[]`                              |
| `graylog.service.master.enabled`                  | If true, Graylog Master Service will be created                                                                                                 | `true`                            |
| `graylog.service.master.port`                     | Graylog Master Service port                                                                                                                     | `9000`                            |
| `graylog.service.master.type`                     | Graylog Master Service type                                                                                                                     | `ClusterIP`                       |
| `graylog.service.master.annotations`              | Graylog Master Service annotations                                                                                                              | `{}`                              |
| `graylog.service.headless.suffix`                 | If present, suffix appended to the name of the chart to form the headless service name, ie: `-headless` would result in `graylog-headless`      |                                   |
| `graylog.podAnnotations`                          | Kubernetes Pod annotations                                                                                                                      | `{}`                              |
| `graylog.priorityClassName`                       | If specified, indicates the pod's priority.                                                                                                     | ``                                |
| `graylog.terminationGracePeriodSeconds`           | Pod termination grace period                                                                                                                    | `120`                             |
| `graylog.updateStrategy`                          | Update Strategy of the StatefulSet                                                                                                              | `RollingUpdate`                   |
| `graylog.persistence.enabled`                     | Use a PVC to persist data                                                                                                                       | `true`                            |
| `graylog.persistence.storageClass`                | Storage class of backing PVC (uses storage class annotation)                                                                                    | `nil`                             |
| `graylog.persistence.accessMode`                  | Use volume as ReadOnly or ReadWrite                                                                                                             | `ReadWriteOnce`                   |
| `graylog.persistence.size`                        | Size of data volume                                                                                                                             | `10Gi`                            |
| `graylog.tls.enabled`                             | If true, Graylog will listen on HTTPS                                                                                                           | `false`                           |
| `graylog.tls.keyFile`                             | Path to key file for HTTPS                                                                                                                      | `/etc/graylog/server/server.key`  |
| `graylog.tls.certFile`                            | Path to crt file for HTTPS                                                                                                                      | `/etc/graylog/server/server.cert` |
| `graylog.ingress.enabled`                         | If true, Graylog Ingress will be created                                                                                                        | `false`                           |
| `graylog.ingress.ingressClassName`                | Graylog Ingress class name                                                                                                                      |                                   |
| `graylog.ingress.port`                            | Graylog Ingress port                                                                                                                            | `false`                           |
| `graylog.ingress.annotations`                     | Graylog Ingress annotations                                                                                                                     | `{}`                              |
| `graylog.ingress.hosts`                           | Graylog Ingress host names                                                                                                                      | `[]`                              |
| `graylog.ingress.tls`                             | Graylog Ingress TLS configuration (YAML)                                                                                                        | `[]`                              |
| `graylog.ingress.pathType`                        | Graylog Ingress path type                                                                                                                       | `Prefix`                          |
| `graylog.ingress.extraPaths`                      | Ingress extra paths to prepend to every host configuration. Useful when configuring [custom actions with AWS ALB Ingress Controller][2].        | `[]`                              |
| `graylog.input`                                   | Graylog Input configuration (YAML) Sees #Input section for detail                                                                               | `{}`                              |
| `graylog.input.tcp.service.name`                  | Graylog TCP Input service name                                                                                                                  | `graylog-tcp`                     |
| `graylog.input.udp.service.name`                  | Graylog UDP Input service name                                                                                                                  | `graylog-udp`                     |
| `graylog.metrics.enabled`                         | If true, add Prometheus annotations to pods                                                                                                     | `false`                           |
| `graylog.metrics.serviceMonitor.enabled`          | If true, a ServiceMonitor resource for the prometheus-operator is created                                                                       | `false`                           |
| `graylog.metrics.serviceMonitor.additionalLabels` | ServiceMonitor additional Labels                                                                                                                | `false`                           |
| `graylog.metrics.serviceMonitor.scrapeTimeout`    | ServiceMonitor Timeout for scraping                                                                                                             | `false`                           |
| `graylog.metrics.serviceMonitor.interval`         | ServiceMonitor Interval at which Prometheus scrapes metrics                                                                                     | `false`                           |
| `graylog.geoip.enabled`                           | If true, Maxmind Geoip Lite will be installed to ${GRAYLOG_HOME}/etc/GeoLite2-City.mmdb                                                         | `false`                           |
| `graylog.geoip.mmdbUri`                           | If set and geoip enabled, Maxmind Geoip Lite will be installed from the URL you have defined to ${GRAYLOG_HOME}/etc/GeoLite2-City.mmdb          |                                   |
| `graylog.plugins.locations`                       | A list of Graylog installation plugins                                                                                                          | `[]`                              |
| `graylog.plugins.proxy.enabled`                   | If true, configure a proxy server to download the plugins                                                                                       | `false`                           |
| `graylog.plugins.proxy.host`                      | The proxy server that should be used to download the plugins                                                                                    | `http://your.proxy.host:8080`     |
| `graylog.rootUsername`                            | Graylog root user name                                                                                                                          | `admin`                           |
| `graylog.rootPassword`                            | Graylog root password. If not set, random 16-character alphanumeric string                                                                      |                                   |
| `graylog.rootEmail`                               | Graylog root email.                                                                                                                             |                                   |
| `graylog.existingRootSecret`                      | Graylog existing root secret                                                                                                                    |                                   |
| `graylog.rootTimezone`                            | Graylog root timezone.                                                                                                                          | `UTC`                             |
| `graylog.opensearch.version`                      | Graylog opensearch version. You need to specify a value 6 or 7. It is required for Graylog >4.0.2                                               | `6`                               |
| `graylog.opensearch.hosts`                        | Graylog opensearch host name. You need to specific where data will be stored.                                                                   |                                   |
| `graylog.opensearch.uriSecretName`                | K8s secret name where opensearch hosts will be set from.                                                                                        | `{{ graylog.fullname }}-es`       |
| `graylog.opensearch.uriSecretKey`                 | K8s secret key name where opensearch hosts will be set from.                                                                                    |                                   |
| `graylog.opensearch.uriSSL`                       | Prepends 'https://' to the URL fetched from 'uriSecretKey' if true. Prepends `http://` otherwise.                                               | `false`                           |
| `graylog.mongodb.uri`                             | Graylog MongoDB connection string. You need to specific where data will be stored.                                                              |                                   |
| `graylog.mongodb.uriSecretName`                   | K8s secret name where MongoDB URI will be set from.                                                                                             | `{{ graylog.fullname }}-mongodb`  |
| `graylog.mongodb.uriSecretKey`                    | K8s secret key name where MongoDB URI will be set from.                                                                                         |                                   |
| `graylog.transportEmail.enabled`                  | If true, enable transport email settings on Graylog                                                                                             | `false`                           |
| `graylog.transportEmail.hostname`                 | The hostname of the server used to send the email                                                                                               |                                   |
| `graylog.transportEmail.port`                     | The port of the server used to send the email                                                                                                   |                                   |
| `graylog.transportEmail.useTls`                   | If true, use TLS to connect to the mailserver                                                                                                   |                                   |
| `graylog.transportEmail.useSsl`                   | If true, use SSL to connect to the mailserver                                                                                                   |                                   |
| `graylog.transportEmail.useAuth`                  | If true, authenticate to the email server                                                                                                       |                                   |
| `graylog.transportEmail.authUsername`             | The username for server authentication                                                                                                          |                                   |
| `graylog.transportEmail.authPassword`             | The password for server authentication                                                                                                          |                                   |
| `graylog.transportEmail.subjectPrefix`            | Prepend this string to every mail subjects                                                                                                      |                                   |
| `graylog.transportEmail.fromEmail`                | Use this as a FROM address                                                                                                                      |                                   |
| `graylog.config`                                  | Add additional server configuration to `graylog.conf` file.                                                                                     |                                   |
| `graylog.serverFiles`                             | Add additional server files on /etc/graylog/server. This is useful for enable TLS on input                                                      | `{}`                              |
| `graylog.logInJson`                               | If true, Graylog pods will be configured to log in JSON (one event per line                                                                     | `false`                           |
| `graylog.journal.deleteBeforeStart`               | Delete all journal files before start Graylog                                                                                                   | `false`                           |
| `graylog.journal.maxSize`                         | Maximum size of message_journal_max_size in Graylog Config                                                                                      | `5gb`                             |
| `graylog.init.image.repository`                   | Configure init container image                                                                                                                  | `busybox`                         |
| `graylog.init.image.pullPolicy`                   | Configure init container image pull policy                                                                                                      | `{}`                              |
| `graylog.init.kubectlLocation`                    | Set kubectl location to download and use on init-container.                                                                                     |                                   |
| `graylog.init.kubectlVersion`                     | Set kubectl command version to download. If the value is not set, default value is .Capabilities.KubeVersion.Version                            |                                   |
| `graylog.init.env`                                | Additional environment variables to be added to Graylog initContainer                                                                           | `{}`                              |
| `graylog.init.resources`                          | Configure resource requests and limits for the Graylog StatefulSet initContainer                                                                | `{}`                              |
| `graylog.provisioner.enabled`                     | Enable optional Job to run an arbitrary Bash script                                                                                             | `false`                           |
| `graylog.provisioner.env`                         | Job environment variables                                                                                                                       | `false`                           |
| `graylog.provisioner.envRaw`                      | Job environment variables in raw yaml format                                                                                                    | `false`                           |
| `graylog.provisioner.annotations`                 | Graylog provisioner Job annotations                                                                                                             | `{}`                              |
| `graylog.provisioner.useGraylogServiceAccount`    | Use the same ServiceAccount used by Graylog pod                                                                                                 | `false`                           |
| `graylog.provisioner.script`                      | The contents of the provisioner Bash script                                                                                                     |                                   |
| `graylog.sidecarContainers`                       | Sidecar containers to run in the server statefulset                                                                                             | `[]`                              |
| `graylog.extraVolumeMounts`                       | Additional Volume mounts                                                                                                                        | `[]`                              |
| `graylog.extraVolumes`                            | Additional Volumes                                                                                                                              | `[]`                              |
| `graylog.extraVolumeClaimTemplates`               | Additional Volume claim templates                                                                                                               | `[]`                              |
| `graylog.extraInitContainers`                     | Additional Init containers                                                                                                                      | `[]`                              |
| `graylog.secret.annotations`                      | Graylog Secret annotations                                                                                                                      | `{}`                              |
| `rbac.create`                                     | If true, create & use RBAC resources                                                                                                            | `true`                            |
| `rbac.resources`                                  | List of resources                                                                                                                               | `[pods, secrets]`                 |
| `serviceAccount.create`                           | If true, create the Graylog service account                                                                                                     | `true`                            |
| `serviceAccount.name`                             | Name of the server service account to use or create                                                                                             | `{{ graylog.fullname }}`          |
| `serviceAccount.annotations`                      | Service Account annotations                                                                                                                     | `{}`                              |
| `tags.install-mongodb`                            | If true, this chart will install MongoDB from requirement dependencies. If you want to install MongoDB by yourself, please set to `false`       | `true`                            |
| `tags.install-opensearch`                         | If true, this chart will install opensearch from requirement dependencies. If you want to install opensearch by yourself, please set to `false` | `true`                            |
| `imagePullSecrets`                                | Configuration for [imagePullSecrets][3] so that you can use a private registry for your images                                                  | `[]`                              |
| `graylog.options.allowHighlighting`               | If true, enable [search result highlighting][6].                                                                                                | `false`                           |
| `graylog.options.allowLeadingWildcardSearches`    | if true, allow searches with leading wildcards. This can be extremely resource hungry and should only be enabled with care.                     | `false`                           |
| `graylog.options.gc_warning_threshold`            | The threshold of the garbage collection runs                                                                                                    | `1s`                              |
| `graylog.options.ringSize`                        | Size of internal ring buffers.                                                                                                                  | `65536`                           |
| `graylog.options.processbufferProcessors`         | Number of processors assigned to the process buffer                                                                                             | `5`                               |
| `graylog.options.inputbufferProcessors`           | Number of processors assigned to the input buffer                                                                                               | `2`                               |
| `graylog.options.outputbufferProcessors`          | Number of processors assigned to the output buffer                                                                                              | `3`                               |
| `graylog.options.inputBufferRingSize`             | Size of input internal ring buffers.                                                                                                            | `65536`                           |

## How it works

This chart will create a Graylog [statefulset][4] with one Master node. The chart will automatically create Master node Pod label `graylog-role=master`, if it does not exists. The others Pods will be label with `graylog-role=coordinating`

This chart will automatically calculate Java heap size from given `resources.requests.memory` value. If you want to specify number of heap size, you can set `graylog.heapSize` to your desired value. The `graylog.heapSize` value must be in JVM `-Xmx` format.

## Input

You can enable input ports by edit the `input` values. For example, you want to create a GELF input on port `12222`, and `12223` with Cloud LoadBalancer and syslog on UDP port `5410` without load balancer.

In services of `type: LoadBalancer`, the default externalTrafficPolicy is `Cluster`, but may be overridden in order to [preserve the client IP][5] with `Local`.

```yaml
input:
  tcp:
    service:
      type: LoadBalancer
      externalTrafficPolicy: Local
      loadBalancerIP:
    ports:
      - name: gelf1
        port: 12222
      - name: gelf2
        port: 12223
  udp:
    service:
      type: ClusterIP
    ports:
      - name: syslog
        port: 5410
```

OR, if you want to expose only a single service with all the input ports open, you can do so by specifying the `service.ports` value:

```yaml
service:
  ports:
    - name: gelf
      port: 12222
      protocol: TCP
    - name: syslog
      port: 5410
      protocol: UDP
```

Note: Name must be in **IANA_SVC_NAME** format - at most 15 characters, matching regex **[a-z0-9]**, containing at least one letter, and hyphens cannot be adjacent to other hyphens

Note: The port list should be sorted by port number.

## TLS

To enable TLS on input in Graylog, you need to specify the server private key and certificate. You can add them in `graylog.serverFiles` value. For example

```yaml
graylog:
  serverFiles:
    server.cert: |
      -----BEGIN CERTIFICATE-----
      MIIFYTCCA0mgAwIBAgICEAIwDQYJKoZIhvcNAQELBQAwcjELMAkGA1UEBhMCVEgx
      EDAOBgNVBAgMB0Jhbmdrb2sxEDAOBgNVBAcMB0Jhbmdrb2sxGDAWBgNVBAoMD09t
      aXNlIENvLiwgTHRkLjEPMA0GA1UECwwGRGV2b3BzMRQwEgYDVQQDDAtjYS5vbWlz
      ZS5jbzAeFw0xNzA2MDEwOTQ0NTJaFw0xOTA2MjEwOTQ0NTJaMHkxCzAJBgNVBAYT
      AlRIMRAwDgYDVQQIDAdCYW5na29rMRAwDgYDVQQHDAdCYW5na29rMRgwFgYDVQQK
      DA9PbWlzZSBDby4sIEx0ZC4xDzANBgNVBAsMBkRldm9wczEbMBkGA1UEAwwSZ3Jh
      4YE6FOKJmiDV7KsmoSO2JTEaZAK6sdxI7zFJJH0TNFIuKewEBsVH/W5RccjwK/z/
      BHwoTQc95zbfFjt1JwDiq8jGTVnQoXH99wAIW+HDYq6hqHyqW3YuQ8QvXfi/ebAs
      rn0urmEC7JhsZIg92AqVYEgdp5H6uFqPIK1U6aYrz5zzZpRfEA==
      -----END CERTIFICATE-----
    server.key: |
      -----BEGIN PRIVATE KEY-----
      MIIEugIBADANBgkqhkiG9w0BAQEFAASCBKQwggSgAgEAAoIBAQC1zwgrnurQGlwe
      ZcKe2RXLs9XzQo4PzNsbxRQXSZef/siUZ/X3phd7Tt7QbQv8sxoZFR1/R4neN3KV
      tsWJ6YL3CY1IwqzxtR6SHzkg/CgUFgP4Jq9NDodOFRlmkZBK9iO9x/VITxLZPBQt
      f+ygeNhfG/oZZxlLSWNC/adlFfUGI8TujCGGyydxAegyWRYmhkLM7F3vRqMXiUn2
      UP/nPEMasHiHS7r99RzJILbU494aNYTxprfBAoGAdWwO/4I/r3Zo672AvCs2s/P6
      G85cX2hKMFy3B4/Ww53jFA3bsWTOyXBv4srl3v9C3xkQmDwUxPDshEN45JX1AMIc
      vxQkW5cm2IaPHB1BsuQpAuW6qIBT/NZqLmexb4jipAjTN4wQ2dkjI/zK2/SST5wb
      vNufGafZ1IpvkUsDkA0=
      -----END PRIVATE KEY-----
```

### Input TLS

The certificates will be mounted into the `/etc/graylog/server`, so Inputs (e.g. TCP/UDP) can be configured to leverage
those certificates with the following Input API configuration:

| Parameter      | Value                           |
| -------------- | ------------------------------- |
| tls_cert_file: | /etc/graylog/server/server.cert |
| tls_enable:    | true                            |
| tls_key_file:  | /etc/graylog/server/server.key  |

### Web HTTPS

Graylog can be auto configured to run in HTTPS mode when provided certificates by setting the `graylog.tls.enabled` value to `true`.

If the certificates are different than those provided above (different hostname for example), then the web-specific
certificates can be added to `graylog.serverFiles` and you can configure the `graylog.tls.certPath` and `graylog.tls.keyPath` to match.

Each Graylog node coordinates with each other through the DNS entry exposed via the headless service, so when generating
the certificates, be sure to include a SAN entry for `*.graylog[-<suffix>].<namespace>.cluster.local` (or your configured FQDN).

## Get Graylog status

You can get your Graylog status by running the command

```bash
kubectl get po -L graylog-role
```

Output

```output
NAME                        READY     STATUS    RESTARTS   AGE       graylog-ROLE
graylog-0                   1/1       Running     0          1d        master
graylog-1                   1/1       Running     0          1d        coordinating
graylog-2                   1/1       Running     0          1m        coordinating
```

## Troubleshooting

If you are encounter "Unprocessed Messages" or Journal files corrupted, you may need to delete all journal files before staring Graylog.
You can do this automatically by setting `graylog.journal.deleteBeforeStart` to `true`

The chart will delete all journal files before starting Graylog.

Note: All uncommitted logs will be permanently DELETED when this value is true

[1]: https://www.graylog.org/
[2]: https://kubernetes-sigs.github.io/aws-alb-ingress-controller/guide/ingress/annotation/#actions
[3]: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-pod-that-uses-your-secret
[4]: https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
[5]: https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/#preserving-the-client-source-ip
[6]: https://docs.graylog.org/en/4.0/pages/searching/result_highlighting.html
