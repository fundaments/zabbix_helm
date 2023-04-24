# Zabbix Helm Chart

# Introduction

This Helm chart installs Zabbix in a Kubernetes cluster.

# Prerequisites

- Kubernetes cluster 1.18+
- Helm 3.0+
- Zabbix server 6.0+
- kube-state-metrics 2.13.2+

*Note*

kube-state-metrics (Zabbix helm chart dependency) with Openshift installation requires one modification on Replica Set level:

```bash
      securityContext:
        runAsUser: 65534
        runAsGroup: 65534
        fsGroup: 65534
```
This block must be removed or changed.

## Zabbix Agent

**Zabbix agent** is deployed on a monitoring target to actively monitor local resources and applications (hard drives, memory, processor statistics etc).

## Zabbix Proxy


**Zabbix proxy** is a process that may collect monitoring data from one or more monitored devices and send the information to the Zabbix server, essentially working on behalf of the server. All collected data is buffered locally and then transferred to the external **Zabbix server** the proxy belongs to.

# Installation

Install requirement [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) and [helm](https://helm.sh/docs/) following the instructions.


# How to Deploy Zabbix in Kubernetes

Clone the repository:

```bash
cd ~
git clone https://git.zabbix.com/scm/zt/kubernetes-helm.git
cd kubernetes-helm

```

Export default values of chart ``helm-zabbix`` to file ``$HOME/zabbix_values.yaml``:

```bash
helm show values . > $HOME/zabbix_values.yaml
```
Change the values according to the environment in the file ``$HOME/zabbix_values.yaml``.


List the namespaces of cluster.

```bash
kubectl get namespaces
```

Create the namespaces ``monitoring`` if it not exists in cluster.

```bash
kubectl create namespace monitoring
```

Deploy Zabbix in the Kubernetes cluster. (Update the YAML files paths if necessary).

```bash
helm install zabbix . --dependency-update -f $HOME/zabbix_values.yaml -n monitoring

```

View the pods.

```bash
kubectl get pods -n monitoring
```

View informations of pods.

```bash
kubectl describe pods/POD_NAME -n monitoring
```

View all containers of pod.

```bash
kubectl get pods POD_NAME -n monitoring -o jsonpath='{.spec.containers[*].name}*'
```

View the logs container of pods.

```bash
kubectl logs -f pods/POD_NAME -c CONTAINER_NAME -n monitoring
```

Access prompt of container.

```bash
kubectl exec -it pods/POD_NAME -c CONTAINER_NAME -n monitoring -- sh
```

# Uninstallation

To uninstall/delete the ``zabbix`` deployment:

```bash
helm delete zabbix -n monitoring
```

# How to access Zabbix

After deploying the chart in your cluster, you can use the following command to access the zabbix proxy service:

View informations of ``zabbix`` services.

View informations of service Zabbix.

```bash
kubectl get svc -n monitoring
kubectl get pods --output=wide -n monitoring
kubectl describe services zabbix -n monitoring
```

# License

[GNU GPL v2](/LICENSE)

# Configuration

The following tables lists the main configurable parameters of the chart and their default values.

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| nameOverride | string | | replaces the name of the chart in the Chart.yaml |
| fullnameOverride | string | | replaces the generated name |
| kube-state-metrics.enabled | bool | `true`| If true, deploys the kube-state-metrics deployment |
| rbac.create |	bool |	`true` |	Specifies whether the RBAC resources should be created |
| rbac.additionalRulesForClusterRole |	list |	`[]` |	Specifies additional rules  for clusterRole |
| serviceAccount.create	| bool	| `true` | Specifies whether a service account should be created
| serviceAccount.name |	string| `zabbix-service-account` |	The name of the service account to use. If not set name is generated using the fullname template |
| zabbixProxy.enabled | bool | `false` | Enables use of Zabbix proxy |
| zabbixProxy.resources | object | `{}` | Set resources requests/limits for Zabbix proxy |
| zabbixProxy.image.repository | string | `"zabbix/zabbix-proxy-sqlite3"` | Zabbix proxy Docker image name |
| zabbixProxy.image.tag | string | `"alpine-trunk"` | Tag of Docker image of Zabbix proxy |
| zabbixProxy.image.pullPolicy | string | `"IfNotPresent"` | Pull policy of Docker image |
| zabbixProxy.image.pullSecrets | list | `[]` | List of dockerconfig secrets names to use when pulling images |
| zabbixProxy.env.ZBX_PROXYMODE | int | `0` | The variable allows to switch Zabbix proxy mode. Bu default, value is 1 - passive proxy. Allowed values are 0 and 1. |
| zabbixProxy.env.ZBX_SERVER_HOST | string | `"127.0.0.1"` | Zabbix server host |
| zabbixProxy.env.ZBX_SERVER_PORT | int | `10051` | Zabbix server port |
| zabbixProxy.env.ZBX_DEBUGLEVEL | int | `3` |  The variable is used to specify debug level. By default, value is 3|
| zabbixProxy.env.ZBX_JAVAGATEWAY_ENABLE | bool | `false` | The variable enable communication with Zabbix Java Gateway to collect Java related checks. By default, value is false |
| zabbixProxy.env.ZBX_CACHESIZE | string | `"128M"` | Cache size |
| zabbixProxy.env.ZBX_CONFIGFREQUENCY | string | `60` | How often proxy retrieves configuration data from Zabbix server in seconds. Active proxy parameter. Ignored for passive proxies |
| zabbixProxy.service.port | int | `10051` | Port to expose service |
| zabbixProxy.service.annotations | object | `{}` |  Zabbix proxy data Service labels annotations |
| zabbixProxy.service.labels | object | `{}` | Zabbix proxy data Service labels |
| zabbixProxy.service.targetPort | int | `10051` | Port of application pod |
| zabbixProxy.service.type | string | `"ClusterIP"` | Type of service for Zabbix proxy |
| zabbixProxy.service.externalIPs | list | `[]` | External IP for Zabbix proxy |
| zabbixProxy.service.loadBalancerIP | string | `""` | Only use if service.type is "LoadBalancer" |
| zabbixProxy.service.loadBalancerSourceRanges | list | `[]` | Only use if service.type is "LoadBalancer" |
| zabbixProxy.nodeSelector | object | `{}` | Node selector configurations |
| zabbixProxy.tolerations | object | `{}`| Tolerations configurations for Zabbix proxy |
| zabbixProxy.affinity | object | `{}`| Affinity configurations for Zabbix proxy |
| zabbixProxy.persistentVolume.enabled |  bool | `false` | If true, Zabbix proxy will create/use a Persistent Volume Claim |
| zabbixProxy.persistentVolume.accessModes | list | `- ReadWriteOnce` | Zabbix proxy data Persistent Volume access modes |
| zabbixProxy.persistentVolume.annotations | object | `{}` | Zabbix proxy data Persistent Volume Claim annotations |
| zabbixProxy.persistentVolume.existingClaim | string | `''` | Zabbix proxy data Persistent Volume existing claim name |
| zabbixProxy.persistentVolume.mountPath | string | `/data` | Zabbix proxy data Persistent Volume mount root path |
| zabbixProxy.persistentVolume.size | string | `2Gi` | Zabbix proxy data Persistent Volume size |
| zabbixProxy.persistentVolume.storageClass | string | `"-"` | Zabbix proxy data Persistent Volume Storage Class |
| zabbixProxy.persistentVolume.volumeBindingMode | string | `''` | Zabbix proxy data Persistent Volume Binding Mode |
| zabbixProxy.persistentVolume.subPath | string | `''` | Subdirectory of Zabbix proxy data Persistent Volume to mount |
| zabbixAgent.enabled | bool | `true` | Enables use of Zabbix agent |
| zabbixAgent.resources | object | `{}` |  Set resources requests/limits for Zabbix agents |
| zabbixAgent.volumes_host | bool | `true` | If a preconfigured set of volumes to be mounted (`/`, `/etc`, `/sys`, `/proc`, `/var/run`)|
| zabbixAgent.volumes | list | `[]`  | Add additional volumes to be mounted |
| zabbixAgent.volumeMounts | list | `[]` | Add additional volumes to be mounted |
| zabbixAgent.image.repository | string | `"zabbix/zabbix-agent2"` | Zabbix agent Docker image name |
| zabbixAgent.image.tag | string | `"alpine-trunk"` | Tag of Docker image of Zabbix agent |
| zabbixAgent.image.pullPolicy | string | `"IfNotPresent"` | Pull policy of Docker image |
| zabbixAgent.image.pullSecrets | list | `[]` | List of dockerconfig secrets names to use when pulling images |
| zabbixAgent.env.ZBX_HOSTNAME | string | `"zabbix-agent"` | Zabbix agent hostname Case sensitive hostname |
| zabbixAgent.env.ZBX_SERVER_HOST | string | `"0.0.0.0/0"` | Zabbix server host |
| zabbixAgent.env.ZBX_SERVER_PORT | int | `10051` | Zabbix server port |
| zabbixAgent.env.ZBX_PASSIVE_ALLOW | bool | `true` | This variable is boolean (true or false) and enables or disables feature of passive checks. By default, value is true |
| zabbixAgent.env.ZBX_PASSIVESERVERS | string | `"0.0.0.0/0"` | The variable is comma separated list of allowed Zabbix server or proxy hosts for connections to Zabbix agent container |
| zabbixAgent.env.ZBX_ACTIVE_ALLOW | bool | `false` | This variable is boolean (true or false) and enables or disables feature of active checks |
| zabbixAgent.env.ZBX_DEBUGLEVEL | int | `3` |  The variable is used to specify debug level. By default, value is 3|
| zabbixAgent.env.ZBX_TIMEOUT | int | 4 |  The variable is used to specify timeout for processing checks. By default, value is 4|
| zabbixAgent.service.type | string | `"ClusterIP"` | Type of service for Zabbix agent |
| zabbixAgent.service.port | int | `10050` | Port to expose service |
| zabbixAgent.service.annotations | object | ` agent.zabbix/monitor: "true"` |  Zabbix agent data Service labels annotations |
| zabbixAgent.service.labels | object | `{}` | Zabbix agent data Service labels |
| zabbixAgent.service.targetPort | int | `10050` | Port of application pod |
| zabbixAgent.service.externalIPs | list | `[]` | External IP for Zabbix agent |
| zabbixAgent.serviceAccount.create	| bool	| `true` | Specifies whether a service account should be created
| zabbixAgent.serviceAccount.name |	string| `zabbix-agent-service-account` |	The name of the service account to use. If not set name is generated using the fullname template |
| zabbixAgent.nodeSelector | object | `kubernetes.io/os: linux` | nodeSelector configurations |
| zabbixAgent.tolerations | list | ` - effect: NoSchedule key: node-role.kubernetes.io/control-plane`| Allows to schedule Zabbix agents on tainted nodes |
| zabbixAgent.rbac.create |	bool |	`true` |	Specifies whether the RBAC resources should be created |
| zabbixAgent.rbac.pspEnabled |	bool |	`true` |	If true, create & use Pod Security Policy resources |

### `agent.volumes_host`

The following directories will be mounted from the host, inside the pod:

Host | Pod |
---- | ----
`/` | `/hostfs`
`/sys` | `/hostfs/sys`
`/proc` | `/hostfs/proc`
