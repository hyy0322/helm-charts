# Dragonfly Helm Chart

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/dragonfly)](https://artifacthub.io/packages/search?repo=dragonfly)

Provide efficient, stable, secure, low-cost file and image distribution services to be the best practice and standard solution in the related Cloud-Native area.

## TL;DR

```shell
helm repo add dragonfly https://dragonflyoss.github.io/helm-charts/
helm install --create-namespace --namespace dragonfly-system dragonfly dragonfly/dragonfly
```

## Introduction

Dragonfly is an open source intelligent P2P based image and file distribution system. Its goal is to tackle all distribution problems in cloud native scenarios. Currently Dragonfly focuses on being:

- Simple: well-defined user-facing API (HTTP), non-invasive to all container engines;
- Efficient: Seed peer support, P2P based file distribution to save enterprise bandwidth;
- Intelligent: host level speed limit, intelligent flow control due to host detection;
- Secure: block transmission encryption, HTTPS connection support.

Dragonfly is now hosted by the Cloud Native Computing Foundation (CNCF) as an Incubating Level Project. Originally it was born to solve all kinds of distribution at very large scales, such as application distribution, cache distribution, log distribution, image distribution, and so on.

## Prerequisites

- Kubernetes cluster 1.20+
- Helm v3.8.0+

## Installation Guide

When use Dragonfly in Kubernetes, a container runtime must be configured. These work can be done by init script in this charts.

For more detail about installation is available in [Kubernetes with Dragonfly](https://d7y.io/docs/getting-started/quick-start/kubernetes/)

We recommend read the details about [Kubernetes with Dragonfly](https://d7y.io/docs/getting-started/quick-start/kubernetes/) before install.

> **We did not recommend to using dragonfly with docker in Kubernetes** due to many reasons: 1. no fallback image pulling policy. 2. deprecated in Kubernetes.

## Installation

### Install with custom configuration

Create the `values.yaml` configuration file. It is recommended to use external redis and mysql instead of containers. This example uses external mysql and redis.

```yaml
mysql:
  enable: false

externalMysql:
  migrate: true
  host: mysql-host
  username: dragonfly
  password: dragonfly
  database: manager
  port: 3306

redis:
  enable: false

externalRedis:
  host: redis-host
  password: dragonfly
  port: 6379
```

Install dragonfly chart with release name `dragonfly`:

```shell
helm repo add dragonfly https://dragonflyoss.github.io/helm-charts/
helm install --create-namespace --namespace dragonfly-system dragonfly dragonfly/dragonfly -f values.yaml
```

### Install with an existing manager

Create the `values.yaml` configuration file. Need to configure the cluster id associated with scheduler and seed peer. This example is to deploy a cluster using the existing manager and redis.

```yaml
scheduler:
  config:
    manager:
      schedulerClusterID: 1

seedPeer:
  config:
    scheduler:
      manager:
        seedPeer:
          enable: true
          clusterID: 1

manager:
  enable: false

externalManager:
  enable: true
  host: "dragonfly-manager.dragonfly-system.svc.cluster.local"
  restPort: 8080
  grpcPort: 65003

redis:
  enable: false

externalRedis:
  host: redis-host
  password: dragonfly
  port: 6379

mysql:
  enable: false
```

Install dragonfly chart with release name `dragonfly`:

```shell
helm repo add dragonfly https://dragonflyoss.github.io/helm-charts/
helm install --create-namespace --namespace dragonfly-system dragonfly dragonfly/dragonfly -f values.yaml
```

## Uninstall

Uninstall the `dragonfly` deployment:

```shell
helm delete dragonfly --namespace dragonfly-system
```

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| clusterDomain | string | `"cluster.local"` | Install application cluster domain |
| containerRuntime | object | `{"containerd":{"configFileName":"","configPathDir":"/etc/containerd","enable":false,"injectConfigPath":false,"injectRegistryCredencials":{"auth":"","enable":false,"identitytoken":"","password":"","username":""},"registries":["https://ghcr.io","https://quay.io","https://harbor.example.com:8443"]},"crio":{"enable":false,"registries":["https://ghcr.io","https://quay.io","https://harbor.example.com:8443"]},"docker":{"caCert":{"commonName":"Dragonfly Authority CA","countryName":"CN","localityName":"Hangzhou","organizationName":"Dragonfly","stateOrProvinceName":"Hangzhou"},"enable":false,"injectHosts":true,"insecure":false,"registryDomains":["ghcr.io","quay.io"],"registryPorts":[443],"restart":false,"skipHosts":["127.0.0.1","docker.io"]},"extraInitContainers":[],"initContainerImage":"dragonflyoss/openssl"}` | [Experimental] Container runtime support Choose special container runtime in Kubernetes. Support: Containerd, Docker, CRI-O |
| containerRuntime.containerd | object | `{"configFileName":"","configPathDir":"/etc/containerd","enable":false,"injectConfigPath":false,"injectRegistryCredencials":{"auth":"","enable":false,"identitytoken":"","password":"","username":""},"registries":["https://ghcr.io","https://quay.io","https://harbor.example.com:8443"]}` | [Experimental] Containerd support |
| containerRuntime.containerd.configFileName | string | `""` | Custom config file name, default is config.toml This is workaround for kops provider, see https://github.com/kubernetes/kops/pull/13090 for more details |
| containerRuntime.containerd.configPathDir | string | `"/etc/containerd"` | Custom config path directory, default is /etc/containerd e.g. rke2 generator config path is /var/lib/rancher/rke2/agent/etc/containerd/config.toml, docs: https://github.com/rancher/rke2/blob/master/docs/advanced.md#configuring-containerd |
| containerRuntime.containerd.enable | bool | `false` | Enable containerd support Inject mirror config into ${containerRuntime.containerd.configPathDir}/config.toml, if config_path is enabled in ${containerRuntime.containerd.configPathDir}/config.toml, the config take effect real time, but if config_path is not enabled in ${containerRuntime.containerd.configPathDir}/config.toml, need restart containerd to take effect. When the version in ${containerRuntime.containerd.configPathDir}/config.toml is "1", inject dfdaemon.config.proxy.registryMirror.url as registry mirror and restart containerd. When the version in ${containerRuntime.containerd.configPathDir}/config.toml is "2":   1. when config_path is enabled in ${containerRuntime.containerd.configPathDir}/config.toml, inject containerRuntime.containerd.registries into config_path,   2. when containerRuntime.containerd.injectConfigPath=true, inject config_path into ${containerRuntime.containerd.configPathDir}/config.toml and inject containerRuntime.containerd.registries into config_path,   3. when not config_path in ${containerRuntime.containerd.configPathDir}/config.toml and containerRuntime.containerd.injectConfigPath=false, inject dfdaemon.config.proxy.registryMirror.url as registry mirror and restart containerd. |
| containerRuntime.containerd.injectConfigPath | bool | `false` | Config path for multiple registries By default, init container will check ${containerRuntime.containerd.configPathDir}/config.toml, whether is config_path configured, if not, init container will just add the dfdaemon.config.proxy.registryMirror.url for registry mirror. When configPath is true, init container will inject config_path=${containerRuntime.containerd.configPathDir}/certs.d and configure all registries. |
| containerRuntime.containerd.injectRegistryCredencials | object | `{"auth":"","enable":false,"identitytoken":"","password":"","username":""}` | Credencials for authenticating to private registries By default this is aplicable for single registry mode, for reference see docs: https://github.com/containerd/containerd/blob/v1.6.4/docs/cri/registry.md#configure-registry-credentials |
| containerRuntime.crio | object | `{"enable":false,"registries":["https://ghcr.io","https://quay.io","https://harbor.example.com:8443"]}` | [Experimental] CRI-O support |
| containerRuntime.crio.enable | bool | `false` | Enable CRI-O support Inject drop-in mirror config into /etc/containers/registries.conf.d. |
| containerRuntime.docker | object | `{"caCert":{"commonName":"Dragonfly Authority CA","countryName":"CN","localityName":"Hangzhou","organizationName":"Dragonfly","stateOrProvinceName":"Hangzhou"},"enable":false,"injectHosts":true,"insecure":false,"registryDomains":["ghcr.io","quay.io"],"registryPorts":[443],"restart":false,"skipHosts":["127.0.0.1","docker.io"]}` | [Experimental] Support docker, when use docker-shim in Kubernetes, please set containerRuntime.docker.enable to true. For supporting docker, we need generate CA and update certs, then hijack registries traffic, By default, it's unnecessary to restart docker daemon when pull image from private registries, this feature is support explicit registries in containerRuntime.registry.domains, default domains is ghcr.io, quay.io, please update your registries by `--set containerRuntime.registry.domains='{harbor.example.com,harbor.example.net}' --set containerRuntime.registry.injectHosts=true --set containerRuntime.docker.enable=true` Caution:   **We did not recommend to using dragonfly with docker in Kubernetes** due to many reasons: 1. no fallback image pulling policy. 2. deprecated in Kubernetes.   Because the original `daemonset` in Kubernetes did not support `Surging Rolling Update` policy.   When kill current dfdaemon pod, the new pod image can not be pulled anymore.   If you can not change runtime from docker to others, remind to choose a plan when upgrade dfdaemon:     Option 1: pull newly dfdaemon image manually before upgrade dragonfly, or use [ImagePullJob](https://openkruise.io/docs/user-manuals/imagepulljob) to pull image automate.     Option 2: keep the image registry of dragonfly is different from common registries and add host in `containerRuntime.docker.skipHosts`. Caution: docker hub image is not supported without restart docker daemon. When need pull image from docker hub or any other registries not in containerRuntime.registry.domains, set containerRuntime.docker.restart=true this feature will inject proxy config into docker.service and restart docker daemon. Caution: set restart to true only when live restore is enable. Requirement: Docker Engine v1.2.0+ without Rootless. |
| containerRuntime.docker.caCert | object | `{"commonName":"Dragonfly Authority CA","countryName":"CN","localityName":"Hangzhou","organizationName":"Dragonfly","stateOrProvinceName":"Hangzhou"}` | CA cert info for generating |
| containerRuntime.docker.enable | bool | `false` | Enable docker support Inject ca cert into /etc/docker/certs.d/, Refer: https://docs.docker.com/engine/security/certificates/. |
| containerRuntime.docker.injectHosts | bool | `true` | Inject domains into /etc/hosts to force redirect traffic to dfdaemon. Caution: This feature need dfdaemon to implement SNI Proxy, confirm image tag is greater than or equal to v2.0.0. When use certs and inject hosts in docker, no necessary to restart docker daemon. |
| containerRuntime.docker.insecure | bool | `false` | Skip verify remote tls cert in dfdaemon If registry cert is private or self-signed, set to true. Caution: this option is test only. When deploy in production, should not skip verify tls cert. |
| containerRuntime.docker.registryDomains | list | `["ghcr.io","quay.io"]` | Registry domains By default, docker pull image via https, currently, by default 443 port with https. If not standard port, update registryPorts. |
| containerRuntime.docker.registryPorts | list | `[443]` | Registry ports |
| containerRuntime.docker.restart | bool | `false` | Restart docker daemon to redirect traffic to dfdaemon When containerRuntime.docker.restart=true, containerRuntime.docker.injectHosts and containerRuntime.registry.domains is ignored. If did not want restart docker daemon, keep containerRuntime.docker.restart=false and containerRuntime.docker.injectHosts=true. |
| containerRuntime.docker.skipHosts | list | `["127.0.0.1","docker.io"]` | Skip hosts Some traffic did not redirect to dragonfly, like 127.0.0.1, and the image registries of dragonfly itself. The format likes NO_PROXY in golang, refer: https://github.com/golang/net/blob/release-branch.go1.15/http/httpproxy/proxy.go#L39. Caution: Some registries use s3 or oss for backend storage, when add registries to skipHosts, don't forget add the corresponding backend storage. |
| containerRuntime.extraInitContainers | list | `[]` | Additional init containers |
| containerRuntime.initContainerImage | string | `"dragonflyoss/openssl"` | The image name of init container, need include openssl for ca generating |
| dfdaemon.config.aliveTime | string | `"0s"` | Daemon alive time, when sets 0s, daemon will not auto exit, it is useful for longtime running |
| dfdaemon.config.cacheDir | string | `""` | Dynconfig cache storage directory |
| dfdaemon.config.console | bool | `false` | Console shows log on console |
| dfdaemon.config.dataDir | string | `"/var/lib/dragonfly"` | Daemon data storage directory |
| dfdaemon.config.download.calculateDigest | bool | `true` | Calculate digest, when only pull images, can be false to save cpu and memory |
| dfdaemon.config.download.downloadGRPC.security | object | `{"insecure":true,"tlsVerify":true}` | Download grpc security option |
| dfdaemon.config.download.downloadGRPC.unixListen | object | `{"socket":""}` | Download service listen address current, only support unix domain socket |
| dfdaemon.config.download.peerGRPC.security | object | `{"insecure":true}` | Peer grpc security option |
| dfdaemon.config.download.peerGRPC.tcpListen.listen | string | `"0.0.0.0"` | Listen address |
| dfdaemon.config.download.peerGRPC.tcpListen.port | int | `65000` | Listen port |
| dfdaemon.config.download.perPeerRateLimit | string | `"100Mi"` | Per peer task limit per second |
| dfdaemon.config.download.totalRateLimit | string | `"200Mi"` | Total download limit per second |
| dfdaemon.config.gcInterval | string | `"1m0s"` | Daemon gc task running interval |
| dfdaemon.config.health.path | string | `"/server/ping"` |  |
| dfdaemon.config.health.tcpListen.listen | string | `"0.0.0.0"` |  |
| dfdaemon.config.health.tcpListen.port | int | `40901` |  |
| dfdaemon.config.host.advertiseIP | string | `"0.0.0.0"` | Access ip for other peers when local ip is different with access ip, advertiseIP should be set |
| dfdaemon.config.host.idc | string | `""` | IDC deployed by daemon |
| dfdaemon.config.host.listenIP | string | `"0.0.0.0"` | TCP service listen address port should be set by other options |
| dfdaemon.config.host.location | string | `""` | Geographical location, separated by "|" characters |
| dfdaemon.config.host.netTopology | string | `""` | Network topology, separated by "|" characters |
| dfdaemon.config.host.securityDomain | string | `""` | Security domain deployed by daemon, network isolation between different security domains |
| dfdaemon.config.jaeger | string | `""` |  |
| dfdaemon.config.keepStorage | bool | `false` | When daemon exit, keep peer task data or not it is usefully when upgrade daemon service, all local cache will be saved default is false |
| dfdaemon.config.logDir | string | `""` | Log storage directory |
| dfdaemon.config.metrics | string | `""` | Metrics listen config, eg: 127.0.0.1:8000 |
| dfdaemon.config.objectStorage.enable | bool | `false` | Enable object storage service |
| dfdaemon.config.objectStorage.filter | string | `"Expires&Signature&ns"` | Filter is used to generate a unique Task ID by filtering unnecessary query params in the URL, it is separated by & character. When filter: "Expires&Signature&ns", for example:  http://localhost/xyz?Expires=111&Signature=222&ns=docker.io and http://localhost/xyz?Expires=333&Signature=999&ns=docker.io is same task |
| dfdaemon.config.objectStorage.maxReplicas | int | `3` | MaxReplicas is the maximum number of replicas of an object cache in seed peers. |
| dfdaemon.config.objectStorage.security | object | `{"insecure":true,"tlsVerify":true}` | Object storage service security option |
| dfdaemon.config.objectStorage.tcpListen.listen | string | `"0.0.0.0"` | Listen address |
| dfdaemon.config.objectStorage.tcpListen.port | int | `65004` | Listen port |
| dfdaemon.config.pprofPort | int | `-1` | Listen port for pprof, only valid when the verbose option is true default is -1. If it is 0, pprof will use a random port. |
| dfdaemon.config.proxy.defaultFilter | string | `"Expires&Signature&ns"` | Filter for hash url. when defaultFilter: "Expires&Signature&ns", for example: http://localhost/xyz?Expires=111&Signature=222&ns=docker.io and http://localhost/xyz?Expires=333&Signature=999&ns=docker.io is same task, it is also possible to override the default filter by adding the X-Dragonfly-Filter header through the proxy. |
| dfdaemon.config.proxy.defaultTag | string | `""` | Tag the task. when the value of the default tag is different, the same download url can be divided into different tasks according to the tag, it is also possible to override the default tag by adding the X-Dragonfly-Tag header through the proxy. |
| dfdaemon.config.proxy.proxies[0] | object | `{"regx":"blobs/sha256.*"}` | Proxy all http image layer download requests with dfget |
| dfdaemon.config.proxy.registryMirror.dynamic | bool | `true` | When enabled, use value of "X-Dragonfly-Registry" in http header for remote instead of url host |
| dfdaemon.config.proxy.registryMirror.insecure | bool | `false` | When the cert of above url is secure, set insecure to true |
| dfdaemon.config.proxy.registryMirror.url | string | `"https://index.docker.io"` | URL for the registry mirror |
| dfdaemon.config.proxy.security | object | `{"insecure":true,"tlsVerify":false}` | Proxy security option |
| dfdaemon.config.proxy.tcpListen.listen | string | `"0.0.0.0"` | Listen address |
| dfdaemon.config.proxy.tcpListen.namespace | string | `"/run/dragonfly/net"` | Namespace stands the linux net namespace, like /proc/1/ns/net it's useful for running daemon in pod with ip allocated and listening the special port in host net namespace Linux only |
| dfdaemon.config.scheduler | object | `{"disableAutoBackSource":false,"manager":{"enable":true,"netAddrs":null,"refreshInterval":"5m","seedPeer":{"clusterID":1,"enable":false,"type":"super"}},"scheduleTimeout":"30s"}` | Scheduler config, netAddrs is auto-configured in templates/dfdaemon/dfdaemon-configmap.yaml |
| dfdaemon.config.scheduler.disableAutoBackSource | bool | `false` | Disable auto back source in dfdaemon |
| dfdaemon.config.scheduler.manager.enable | bool | `true` | Get scheduler list dynamically from manager |
| dfdaemon.config.scheduler.manager.netAddrs | string | `nil` | Manager service address, netAddr is a list, there are two fields type and addr |
| dfdaemon.config.scheduler.manager.refreshInterval | string | `"5m"` | Scheduler list refresh interval |
| dfdaemon.config.scheduler.manager.seedPeer.clusterID | int | `1` | Associated seed peer cluster id |
| dfdaemon.config.scheduler.manager.seedPeer.enable | bool | `false` | Enable seed peer mode. |
| dfdaemon.config.scheduler.manager.seedPeer.type | string | `"super"` | Seed peer supports "super", "strong" and "weak" types. |
| dfdaemon.config.scheduler.scheduleTimeout | string | `"30s"` | Schedule timeout |
| dfdaemon.config.storage.diskGCThreshold | string | `"50Gi"` | Disk GC Threshold |
| dfdaemon.config.storage.multiplex | bool | `true` | Set to ture for reusing underlying storage for same task id |
| dfdaemon.config.storage.strategy | string | `"io.d7y.storage.v2.simple"` | Storage strategy when process task data io.d7y.storage.v2.simple : download file to data directory first, then copy to output path, this is default action                           the download file in date directory will be the peer data for uploading to other peers io.d7y.storage.v2.advance: download file directly to output path with postfix, hard link to final output,                            avoid copy to output path, fast than simple strategy, but:                            the output file with postfix will be the peer data for uploading to other peers                            when user delete or change this file, this peer data will be corrupted default is io.d7y.storage.v2.advance |
| dfdaemon.config.storage.taskExpireTime | string | `"6h"` | Task data expire time when there is no access to a task data, this task will be gc. |
| dfdaemon.config.upload.rateLimit | string | `"100Mi"` | Upload limit per second |
| dfdaemon.config.upload.security | object | `{"insecure":true,"tlsVerify":false}` | Upload grpc security option |
| dfdaemon.config.upload.tcpListen.listen | string | `"0.0.0.0"` | Listen address |
| dfdaemon.config.upload.tcpListen.port | int | `65002` | Listen port |
| dfdaemon.config.verbose | bool | `false` | Whether to enable debug level logger and enable pprof |
| dfdaemon.config.workHome | string | `""` | Daemon work directory |
| dfdaemon.containerPort | int | `65001` | Pod containerPort |
| dfdaemon.daemonsetAnnotations | object | `{}` | Daemonset annotations |
| dfdaemon.enable | bool | `true` | Enable dfdaemon |
| dfdaemon.extraVolumeMounts | list | `[{"mountPath":"/var/log/dragonfly/daemon","name":"logs"}]` | Extra volumeMounts for dfdaemon. |
| dfdaemon.extraVolumes | list | `[{"emptyDir":{},"name":"logs"}]` | Extra volumes for dfdaemon. |
| dfdaemon.fullnameOverride | string | `""` | Override dfdaemon fullname |
| dfdaemon.hostAliases | list | `[]` | Host Aliases |
| dfdaemon.hostNetwork | bool | `false` | Using hostNetwork when pod with host network can communicate with normal pods with cni network |
| dfdaemon.hostPort | int | `65001` | When .hostNetwork == false, and .config.proxy.tcpListen.namespace is empty many network add-ons do not yet support hostPort https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/#hostport-services-do-not-work by default, dfdaemon injects the 65001 port to host network by sharing host network namespace, if you want to use hostPort, please empty .config.proxy.tcpListen.namespace below, and keep .hostNetwork == false for performance, injecting the 65001 port to host network is better than hostPort |
| dfdaemon.image | string | `"dragonflyoss/dfdaemon"` | Image repository |
| dfdaemon.mountDataDirAsHostPath | bool | `false` | Mount data directory from host when enabled, mount host path to dfdaemon, or just emptyDir in dfdaemon |
| dfdaemon.name | string | `"dfdaemon"` | Dfdaemon name |
| dfdaemon.nameOverride | string | `""` | Override dfdaemon name |
| dfdaemon.nodeSelector | object | `{}` | Node labels for pod assignment |
| dfdaemon.podAnnotations | object | `{}` | Pod annotations |
| dfdaemon.podLabels | object | `{}` | Pod labels |
| dfdaemon.priorityClassName | string | `""` | Pod priorityClassName |
| dfdaemon.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| dfdaemon.resources | object | `{"limits":{"cpu":"2","memory":"2Gi"},"requests":{"cpu":"0","memory":"0"}}` | Pod resource requests and limits |
| dfdaemon.tag | string | `"v2.0.5"` | Image tag |
| dfdaemon.terminationGracePeriodSeconds | string | `nil` | Pod terminationGracePeriodSeconds |
| dfdaemon.tolerations | list | `[]` | List of node taints to tolerate |
| externalManager.grpcPort | int | `65003` | External GRPC service port |
| externalManager.host | string | `nil` | External manager hostname |
| externalManager.restPort | int | `8080` | External REST service port |
| externalMysql.database | string | `"manager"` | External mysql database name |
| externalMysql.host | string | `nil` | External mysql hostname |
| externalMysql.migrate | bool | `true` | Running GORM migration |
| externalMysql.password | string | `"dragonfly"` | External mysql password |
| externalMysql.port | int | `3306` | External mysql port |
| externalMysql.username | string | `"dragonfly"` | External mysql username |
| externalRedis.host | string | `""` | External redis hostname |
| externalRedis.password | string | `"dragonfly"` | External redis password |
| externalRedis.port | int | `6379` | External redis port |
| fullnameOverride | string | `""` | Override dragonfly fullname |
| jaeger.agent.enabled | bool | `false` |  |
| jaeger.allInOne.enabled | bool | `true` |  |
| jaeger.collector.enabled | bool | `false` |  |
| jaeger.enable | bool | `false` | Enable an all-in-one jaeger for tracing every downloading event should not use in production environment |
| jaeger.provisionDataStore.cassandra | bool | `false` |  |
| jaeger.query.enabled | bool | `false` |  |
| jaeger.storage.type | string | `"none"` |  |
| manager.config.cache.local.size | int | `10000` | Size of LFU cache |
| manager.config.cache.local.ttl | string | `"10s"` | Local cache TTL duration |
| manager.config.cache.redis.ttl | string | `"30s"` | Redis cache TTL duration |
| manager.config.console | bool | `false` | Console shows log on console |
| manager.config.jaeger | string | `""` |  |
| manager.config.objectStorage.accessKey | string | `""` | Access key ID |
| manager.config.objectStorage.enable | bool | `false` | Enable object storage |
| manager.config.objectStorage.endpoint | string | `""` | Datacenter endpoint |
| manager.config.objectStorage.name | string | `"s3"` | Object storage name of type, it can be s3 or oss |
| manager.config.objectStorage.region | string | `""` | Storage region |
| manager.config.objectStorage.secretKey | string | `""` | Access key secret |
| manager.config.pprofPort | int | `-1` | Listen port for pprof, only valid when the verbose option is true default is -1. If it is 0, pprof will use a random port. |
| manager.config.verbose | bool | `false` | Whether to enable debug level logger and enable pprof |
| manager.deploymentAnnotations | object | `{}` | Deployment annotations |
| manager.enable | bool | `true` | Enable manager |
| manager.extraVolumeMounts | list | `[{"mountPath":"/var/log/dragonfly/manager","name":"logs"}]` | Extra volumeMounts for manager. |
| manager.extraVolumes | list | `[{"emptyDir":{},"name":"logs"}]` | Extra volumes for manager. |
| manager.fullnameOverride | string | `""` | Override manager fullname |
| manager.grpcPort | int | `65003` | GRPC service port |
| manager.hostAliases | list | `[]` | Host Aliases |
| manager.image | string | `"dragonflyoss/manager"` | Image repository |
| manager.ingress.annotations | object | `{}` | Ingress annotations |
| manager.ingress.className | string | `""` | Ingress class name. Requirement: kubernetes >=1.18 |
| manager.ingress.enable | bool | `false` | Enable ingress |
| manager.ingress.hosts | list | `[]` | Manager ingress hosts |
| manager.ingress.path | string | `"/"` | Ingress host path |
| manager.ingress.pathType | string | `"ImplementationSpecific"` | Ingress path type. Requirement: kubernetes >=1.18 |
| manager.ingress.tls | list | `[]` | Ingress TLS configuration |
| manager.initContainer.image | string | `"busybox"` | Init container image repository |
| manager.initContainer.pullPolicy | string | `"IfNotPresent"` | Container image pull policy |
| manager.initContainer.tag | string | `"latest"` | Init container image tag |
| manager.metrics.enable | bool | `false` | Enable manager metrics |
| manager.metrics.prometheusRule.additionalLabels | object | `{}` | Additional labels |
| manager.metrics.prometheusRule.enable | bool | `false` | Enable prometheus rule ref: https://github.com/coreos/prometheus-operator |
| manager.metrics.prometheusRule.rules | list | `[]` | Prometheus rules |
| manager.metrics.service.annotations | object | `{}` | Service annotations |
| manager.metrics.service.labels | object | `{}` | Service labels |
| manager.metrics.service.type | string | `"ClusterIP"` | Service type |
| manager.metrics.serviceMonitor.additionalLabels | object | `{}` | Additional labels |
| manager.metrics.serviceMonitor.enable | bool | `false` | Enable prometheus service monitor ref: https://github.com/coreos/prometheus-operator |
| manager.metrics.serviceMonitor.interval | string | `"30s"` | Interval at which metrics should be scraped |
| manager.metrics.serviceMonitor.scrapeTimeout | string | `"10s"` | Timeout after which the scrape is ended |
| manager.name | string | `"manager"` | Manager name |
| manager.nameOverride | string | `""` | Override manager name |
| manager.nodeSelector | object | `{}` | Node labels for pod assignment |
| manager.podAnnotations | object | `{}` | Pod annotations |
| manager.podLabels | object | `{}` | Pod labels |
| manager.priorityClassName | string | `""` | Pod priorityClassName |
| manager.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| manager.replicas | int | `3` | Number of Pods to launch |
| manager.resources | object | `{"limits":{"cpu":"2","memory":"4Gi"},"requests":{"cpu":"0","memory":"0"}}` | Pod resource requests and limits |
| manager.restPort | int | `8080` | REST service port |
| manager.service.annotations | object | `{}` | Service annotations |
| manager.service.labels | object | `{}` | Service labels |
| manager.service.type | string | `"ClusterIP"` | Service type |
| manager.tag | string | `"v2.0.5"` | Image tag |
| manager.terminationGracePeriodSeconds | string | `nil` | Pod terminationGracePeriodSeconds |
| manager.tolerations | list | `[]` | List of node taints to tolerate |
| mysql.auth.database | string | `"manager"` | Mysql database name |
| mysql.auth.host | string | `""` | Mysql hostname |
| mysql.auth.password | string | `"dragonfly"` | Mysql password |
| mysql.auth.rootPassword | string | `"dragonfly-root"` | Mysql root password |
| mysql.auth.username | string | `"dragonfly"` | Mysql username |
| mysql.clusterDomain | string | `"cluster.local"` | Cluster domain |
| mysql.enable | bool | `true` | Enable mysql with docker container. |
| mysql.migrate | bool | `true` | Running GORM migration |
| mysql.primary.service.port | int | `3306` | Mysql port |
| nameOverride | string | `""` | Override dragonfly name |
| redis.auth.enabled | bool | `true` | Enable password authentication |
| redis.auth.password | string | `"dragonfly"` | Redis password |
| redis.clusterDomain | string | `"cluster.local"` | Cluster domain |
| redis.enable | bool | `true` | Enable redis cluster with docker container |
| redis.master.service.ports.redis | int | `6379` | Redis master service port |
| scheduler.config.console | bool | `false` | Console shows log on console |
| scheduler.config.dynconfig.refreshInterval | string | `"10s"` | Dynamic config refresh interval |
| scheduler.config.dynconfig.type | string | `"manager"` | Type is deprecated and is no longer used. Please remove it from your configuration. |
| scheduler.config.host.idc | string | `""` | IDC is the idc of scheduler instance |
| scheduler.config.host.location | string | `""` | Location is the location of scheduler instance |
| scheduler.config.host.netTopology | string | `""` | NetTopology is the net topology of scheduler instance |
| scheduler.config.jaeger | string | `""` |  |
| scheduler.config.manager.keepAlive.interval | string | `"5s"` | Manager keepalive interval |
| scheduler.config.manager.schedulerClusterID | int | `1` | Associated scheduler cluster id |
| scheduler.config.pprofPort | int | `-1` | Listen port for pprof, only valid when the verbose option is true default is -1. If it is 0, pprof will use a random port. |
| scheduler.config.scheduler.algorithm | string | `"default"` | Algorithm configuration to use different scheduling algorithms, default configuration supports "default" and "ml" "default" is the rule-based scheduling algorithm, "ml" is the machine learning scheduling algorithm It also supports user plugin extension, the algorithm value is "plugin", and the compiled `d7y-scheduler-plugin-evaluator.so` file is added to the dragonfly working directory plugins |
| scheduler.config.scheduler.backSourceCount | int | `3` | Number of backsource clients when the seed peer is unavailable |
| scheduler.config.scheduler.gc.peerGCInterval | string | `"10m"` | Peer's gc interval |
| scheduler.config.scheduler.gc.peerTTL | string | `"24h"` | Peer's TTL duration |
| scheduler.config.scheduler.gc.taskGCInterval | string | `"10m"` | Task's gc interval |
| scheduler.config.scheduler.gc.taskTTL | string | `"24h"` | Task's TTL duration |
| scheduler.config.scheduler.retryBackSourceLimit | int | `5` | Retry scheduling back-to-source limit times |
| scheduler.config.scheduler.retryInterval | string | `"50ms"` | Retry scheduling interval |
| scheduler.config.scheduler.retryLimit | int | `10` | Retry scheduling limit times |
| scheduler.config.seedPeer.enable | bool | `true` | scheduler enable seed peer as P2P peer, if the value is false, P2P network will not be back-to-source through seed peer but by dfdaemon and preheat feature does not work |
| scheduler.config.server.cacheDir | string | `""` | Dynconfig cache storage directory |
| scheduler.config.server.dataDir | string | `""` | Storage directory |
| scheduler.config.server.logDir | string | `""` | Log storage directory |
| scheduler.config.server.workHome | string | `""` | Service work directory |
| scheduler.config.verbose | bool | `false` | Whether to enable debug level logger and enable pprof |
| scheduler.containerPort | int | `8002` | Pod containerPort |
| scheduler.enable | bool | `true` | Enable scheduler |
| scheduler.extraVolumeMounts | list | `[{"mountPath":"/var/log/dragonfly/scheduler","name":"logs"}]` | Extra volumeMounts for scheduler. |
| scheduler.extraVolumes | list | `[{"emptyDir":{},"name":"logs"}]` | Extra volumes for scheduler. |
| scheduler.fullnameOverride | string | `""` | Override scheduler fullname |
| scheduler.hostAliases | list | `[]` | Host Aliases |
| scheduler.image | string | `"dragonflyoss/scheduler"` | Image repository |
| scheduler.initContainer.image | string | `"busybox"` | Init container image repository |
| scheduler.initContainer.pullPolicy | string | `"IfNotPresent"` | Container image pull policy |
| scheduler.initContainer.tag | string | `"latest"` | Init container image tag |
| scheduler.metrics.enable | bool | `false` | Enable scheduler metrics |
| scheduler.metrics.enablePeerHost | bool | `false` | Enable peer host metrics |
| scheduler.metrics.prometheusRule.additionalLabels | object | `{}` | Additional labels |
| scheduler.metrics.prometheusRule.enable | bool | `false` | Enable prometheus rule ref: https://github.com/coreos/prometheus-operator |
| scheduler.metrics.prometheusRule.rules | list | `[]` | Prometheus rules |
| scheduler.metrics.service.annotations | object | `{}` | Service annotations |
| scheduler.metrics.service.labels | object | `{}` | Service labels |
| scheduler.metrics.service.type | string | `"ClusterIP"` | Service type |
| scheduler.metrics.serviceMonitor.additionalLabels | object | `{}` | Additional labels |
| scheduler.metrics.serviceMonitor.enable | bool | `false` | Enable prometheus service monitor ref: https://github.com/coreos/prometheus-operator |
| scheduler.metrics.serviceMonitor.interval | string | `"30s"` | Interval at which metrics should be scraped |
| scheduler.metrics.serviceMonitor.scrapeTimeout | string | `"10s"` | Timeout after which the scrape is ended |
| scheduler.name | string | `"scheduler"` | Scheduler name |
| scheduler.nameOverride | string | `""` | Override scheduler name |
| scheduler.nodeSelector | object | `{}` | Node labels for pod assignment |
| scheduler.podAnnotations | object | `{}` | Pod annotations |
| scheduler.podLabels | object | `{}` | Pod labels |
| scheduler.priorityClassName | string | `""` | Pod priorityClassName |
| scheduler.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| scheduler.replicas | int | `3` | Number of Pods to launch |
| scheduler.resources | object | `{"limits":{"cpu":"4","memory":"8Gi"},"requests":{"cpu":"0","memory":"0"}}` | Pod resource requests and limits |
| scheduler.statefulsetAnnotations | object | `{}` | Statefulset annotations |
| scheduler.tag | string | `"v2.0.5"` | Image tag |
| scheduler.terminationGracePeriodSeconds | string | `nil` | Pod terminationGracePeriodSeconds |
| scheduler.tolerations | list | `[]` | List of node taints to tolerate |
| seedPeer.config.aliveTime | string | `"0s"` | Daemon alive time, when sets 0s, daemon will not auto exit, it is useful for longtime running |
| seedPeer.config.cacheDir | string | `""` | Dynconfig cache storage directory |
| seedPeer.config.console | bool | `false` | Console shows log on console |
| seedPeer.config.dataDir | string | `"/var/lib/dragonfly"` | Daemon data storage directory |
| seedPeer.config.download.calculateDigest | bool | `true` | Calculate digest, when only pull images, can be false to save cpu and memory |
| seedPeer.config.download.downloadGRPC.security | object | `{"insecure":true,"tlsVerify":true}` | Download grpc security option |
| seedPeer.config.download.downloadGRPC.unixListen | object | `{"socket":""}` | Download service listen address current, only support unix domain socket |
| seedPeer.config.download.peerGRPC.security | object | `{"insecure":true}` | Peer grpc security option |
| seedPeer.config.download.peerGRPC.tcpListen.listen | string | `"0.0.0.0"` | Listen address |
| seedPeer.config.download.peerGRPC.tcpListen.port | int | `65000` | Listen port |
| seedPeer.config.download.perPeerRateLimit | string | `"1024Mi"` | Per peer task limit per second |
| seedPeer.config.download.totalRateLimit | string | `"2048Mi"` | Total download limit per second |
| seedPeer.config.gcInterval | string | `"1m0s"` | Daemon gc task running interval |
| seedPeer.config.health.path | string | `"/server/ping"` |  |
| seedPeer.config.health.tcpListen.listen | string | `"0.0.0.0"` |  |
| seedPeer.config.health.tcpListen.port | int | `40901` |  |
| seedPeer.config.host.idc | string | `""` | IDC deployed by daemon |
| seedPeer.config.host.listenIP | string | `"0.0.0.0"` | TCP service listen address port should be set by other options |
| seedPeer.config.host.location | string | `""` | Geographical location, separated by "|" characters |
| seedPeer.config.host.netTopology | string | `""` | Network topology, separated by "|" characters |
| seedPeer.config.host.securityDomain | string | `""` | Security domain deployed by daemon, network isolation between different security domains |
| seedPeer.config.jaeger | string | `""` |  |
| seedPeer.config.keepStorage | bool | `false` | When daemon exit, keep peer task data or not it is usefully when upgrade daemon service, all local cache will be saved default is false |
| seedPeer.config.logDir | string | `""` | Log storage directory |
| seedPeer.config.objectStorage.enable | bool | `false` | Enable object storage service |
| seedPeer.config.objectStorage.filter | string | `"Expires&Signature&ns"` | Filter is used to generate a unique Task ID by filtering unnecessary query params in the URL, it is separated by & character. When filter: "Expires&Signature&ns", for example:  http://localhost/xyz?Expires=111&Signature=222&ns=docker.io and http://localhost/xyz?Expires=333&Signature=999&ns=docker.io is same task |
| seedPeer.config.objectStorage.maxReplicas | int | `3` | MaxReplicas is the maximum number of replicas of an object cache in seed peers. |
| seedPeer.config.objectStorage.security | object | `{"insecure":true,"tlsVerify":true}` | Object storage service security option |
| seedPeer.config.objectStorage.tcpListen.listen | string | `"0.0.0.0"` | Listen address |
| seedPeer.config.objectStorage.tcpListen.port | int | `65004` | Listen port |
| seedPeer.config.pprofPort | int | `-1` | Listen port for pprof, only valid when the verbose option is true default is -1. If it is 0, pprof will use a random port. |
| seedPeer.config.proxy.defaultFilter | string | `"Expires&Signature&ns"` | Filter for hash url. when defaultFilter: "Expires&Signature&ns", for example: http://localhost/xyz?Expires=111&Signature=222&ns=docker.io and http://localhost/xyz?Expires=333&Signature=999&ns=docker.io is same task, it is also possible to override the default filter by adding the X-Dragonfly-Filter header through the proxy. |
| seedPeer.config.proxy.defaultTag | string | `""` | Tag the task. when the value of the default tag is different, the same download url can be divided into different tasks according to the tag, it is also possible to override the default tag by adding the X-Dragonfly-Tag header through the proxy. |
| seedPeer.config.proxy.proxies[0] | object | `{"regx":"blobs/sha256.*"}` | Proxy all http image layer download requests with dfget |
| seedPeer.config.proxy.registryMirror.dynamic | bool | `true` | When enabled, use value of "X-Dragonfly-Registry" in http header for remote instead of url host |
| seedPeer.config.proxy.registryMirror.insecure | bool | `false` | When the cert of above url is secure, set insecure to true |
| seedPeer.config.proxy.registryMirror.url | string | `"https://index.docker.io"` | URL for the registry mirror |
| seedPeer.config.proxy.security | object | `{"insecure":true,"tlsVerify":false}` | Proxy security option |
| seedPeer.config.proxy.tcpListen.listen | string | `"0.0.0.0"` | Listen address |
| seedPeer.config.scheduler | object | `{"disableAutoBackSource":false,"manager":{"enable":true,"netAddrs":null,"refreshInterval":"5m","seedPeer":{"clusterID":1,"enable":true,"keepAlive":{"interval":"5s"},"type":"super"}},"scheduleTimeout":"30s"}` | Scheduler config, netAddrs is auto-configured in templates/dfdaemon/dfdaemon-configmap.yaml |
| seedPeer.config.scheduler.disableAutoBackSource | bool | `false` | Disable auto back source in dfdaemon |
| seedPeer.config.scheduler.manager.enable | bool | `true` | Get scheduler list dynamically from manager |
| seedPeer.config.scheduler.manager.netAddrs | string | `nil` | Manager service address, netAddr is a list, there are two fields type and addr |
| seedPeer.config.scheduler.manager.refreshInterval | string | `"5m"` | Scheduler list refresh interval |
| seedPeer.config.scheduler.manager.seedPeer.clusterID | int | `1` | Associated seed peer cluster id |
| seedPeer.config.scheduler.manager.seedPeer.enable | bool | `true` | Enable seed peer mode |
| seedPeer.config.scheduler.manager.seedPeer.keepAlive.interval | string | `"5s"` | Manager keepalive interval |
| seedPeer.config.scheduler.manager.seedPeer.type | string | `"super"` | Seed peer supports "super", "strong" and "weak" types |
| seedPeer.config.scheduler.scheduleTimeout | string | `"30s"` | Schedule timeout |
| seedPeer.config.storage.diskGCThresholdPercent | int | `90` | Disk GC Threshold Percent, when the disk usage is above 90%, start to gc the oldest tasks |
| seedPeer.config.storage.multiplex | bool | `true` | Set to ture for reusing underlying storage for same task id |
| seedPeer.config.storage.strategy | string | `"io.d7y.storage.v2.simple"` | Storage strategy when process task data io.d7y.storage.v2.simple : download file to data directory first, then copy to output path, this is default action                           the download file in date directory will be the peer data for uploading to other peers io.d7y.storage.v2.advance: download file directly to output path with postfix, hard link to final output,                            avoid copy to output path, fast than simple strategy, but:                            the output file with postfix will be the peer data for uploading to other peers                            when user delete or change this file, this peer data will be corrupted default is io.d7y.storage.v2.advance |
| seedPeer.config.storage.taskExpireTime | string | `"6h"` | Task data expire time when there is no access to a task data, this task will be gc. |
| seedPeer.config.upload.rateLimit | string | `"2048Mi"` | Upload limit per second |
| seedPeer.config.upload.security | object | `{"insecure":true,"tlsVerify":false}` | Upload grpc security option |
| seedPeer.config.upload.tcpListen.listen | string | `"0.0.0.0"` | Listen address |
| seedPeer.config.upload.tcpListen.port | int | `65002` | Listen port |
| seedPeer.config.verbose | bool | `false` | Whether to enable debug level logger and enable pprof |
| seedPeer.config.workHome | string | `""` | Daemon work directory |
| seedPeer.enable | bool | `true` | Enable dfdaemon seed peer |
| seedPeer.extraVolumeMounts | list | `[{"mountPath":"/var/log/dragonfly/daemon","name":"logs"}]` | Extra volumeMounts for dfdaemon. |
| seedPeer.extraVolumes | list | `[{"emptyDir":{},"name":"logs"}]` | Extra volumes for dfdaemon. |
| seedPeer.fullnameOverride | string | `""` | Override scheduler fullname |
| seedPeer.hostAliases | list | `[]` | Host Aliases |
| seedPeer.image | string | `"dragonflyoss/dfdaemon"` | Image repository |
| seedPeer.initContainer.image | string | `"busybox"` | Init container image repository |
| seedPeer.initContainer.pullPolicy | string | `"IfNotPresent"` | Container image pull policy |
| seedPeer.initContainer.tag | string | `"latest"` | Init container image tag |
| seedPeer.metrics.enable | bool | `false` | Enable scheduler metrics |
| seedPeer.metrics.enablePeerHost | bool | `false` | Enable peer host metrics |
| seedPeer.metrics.prometheusRule.additionalLabels | object | `{}` | Additional labels |
| seedPeer.metrics.prometheusRule.enable | bool | `false` | Enable prometheus rule ref: https://github.com/coreos/prometheus-operator |
| seedPeer.metrics.prometheusRule.rules | list | `[]` | Prometheus rules |
| seedPeer.metrics.service.annotations | object | `{}` | Service annotations |
| seedPeer.metrics.service.labels | object | `{}` | Service labels |
| seedPeer.metrics.service.type | string | `"ClusterIP"` | Service type |
| seedPeer.metrics.serviceMonitor.additionalLabels | object | `{}` | Additional labels |
| seedPeer.metrics.serviceMonitor.enable | bool | `false` | Enable prometheus service monitor ref: https://github.com/coreos/prometheus-operator |
| seedPeer.metrics.serviceMonitor.interval | string | `"30s"` | Interval at which metrics should be scraped |
| seedPeer.metrics.serviceMonitor.scrapeTimeout | string | `"10s"` | Timeout after which the scrape is ended |
| seedPeer.name | string | `"seed-peer"` | Seed peer name |
| seedPeer.nameOverride | string | `""` | Override scheduler name |
| seedPeer.nodeSelector | object | `{}` | Node labels for pod assignment |
| seedPeer.persistence.accessModes | list | `["ReadWriteOnce"]` | Persistence access modes |
| seedPeer.persistence.annotations | object | `{}` | Persistence annotations |
| seedPeer.persistence.enable | bool | `true` | Enable persistence for seed peer |
| seedPeer.persistence.size | string | `"8Gi"` | Persistence persistence size |
| seedPeer.podAnnotations | object | `{}` | Pod annotations |
| seedPeer.podLabels | object | `{}` | Pod labels |
| seedPeer.priorityClassName | string | `""` | Pod priorityClassName |
| seedPeer.pullPolicy | string | `"IfNotPresent"` | Image pull policy |
| seedPeer.replicas | int | `3` | Number of Pods to launch |
| seedPeer.resources | object | `{"limits":{"cpu":"2","memory":"4Gi"},"requests":{"cpu":"0","memory":"0"}}` | Pod resource requests and limits |
| seedPeer.statefulsetAnnotations | object | `{}` | Statefulset annotations |
| seedPeer.tag | string | `"v2.0.5"` | Image tag |
| seedPeer.terminationGracePeriodSeconds | string | `nil` | Pod terminationGracePeriodSeconds |
| seedPeer.tolerations | list | `[]` | List of node taints to tolerate |

## Chart dependencies

| Repository | Name | Version |
|------------|------|---------|
| https://charts.bitnami.com/bitnami | mysql | 9.1.2 |
| https://charts.bitnami.com/bitnami | redis | 16.10.1 |
| https://jaegertracing.github.io/helm-charts | jaeger | 0.56.5 |
