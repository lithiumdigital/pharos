# Pharos Helm Chart

This Helm chart deploys the Pharos node using a Kubernetes StatefulSet and Service. It is designed for easy configuration, and management of the Pharos blockchain nodes.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Upgrading](#upgrading)
- [Uninstalling](#uninstalling)
- [Configuration](#configuration)
  - [Parameters](#parameters)
- [Usage](#usage)
- [Snapshot Job (Restore from Snapshot)](#snapshot-job-restore-from-snapshot)
- [Monitoring Job](#configuring-monitoring-in-pharos-node)
- [Template Structure](#template-structure)
- [Contributing](#contributing)
- [License](#license)

---

## Prerequisites

- Kubernetes 1.12+ cluster
- Helm 3.x installed
- Sufficient cluster resources for the requested CPU, memory, and storage
- Gateway API CRDs and Controller for external access (if needed)
- Certmanager for TLS (if needed). Here is a guide: [Certmanager Guide](https://cert-manager.io/docs/installation/helm/) 

---

## Installation

1. **Clone the repository or copy the chart directory:**

   ```sh
   git clone https://github.com/lithiumdigital/pharos.git pharos
   cd pharos
   ```

2. **Customize your deployment:**

   Edit `values.yaml` to fit your requirements (see [Parameters](#parameters) below).

3. **Install the chart:**

   ```sh
   helm install <release-name> .
   ```

   Replace `<release-name>` with your desired Helm release name.

---

## Alternative Installation with Custom Values

You can add the helm chart to your local Helm repository and install it with custom values:

```sh
helm repo add pharos https://lithiumdigital.github.io/pharos

helm repo update
```

Then install the chart with custom values:

```sh
helm install pharos-demo pharos/pharos -f values.yaml
```

---

## Upgrading

To upgrade the release with new values:

```sh
helm upgrade <release-name> . -f values.yaml


helm upgrade --install pharos-demo . \
  --set prometheus.enabled=true \
  --set alertmanager.enabled=true \
  --set prom2teams.enabled=true \
  --set pushGateway.enabled=true \
  --set grafana.enabled=true \
  --set snapshot.enabled=false \
  --set monitoring.enabled=false
```
---

## Uninstalling

To uninstall and delete all resources created by this chart:

```sh
helm uninstall <release-name>
```

---

## Configuration

All configurable parameters are defined in values.yaml. You can override any value using `--set key=value` or by editing the file.

### Parameters

| Parameter                        | Description                                                 | Default                                      |
|----------------------------------|-------------------------------------------------------------|----------------------------------------------|
| `nameOverride`                   | Override the release name                                   | `""`                                         |
| `namespaceOverride`              | Override the default namespace                              | `""`                                         |
| `network.type`                   | Type of network to deploy (`mainnet`, `testnet`, etc.)      | `testnet`                                    |
| `image.repository`               | Container image repository                                  | `public.ecr.aws/k2g7b7g1/pharos/testnet`     |
| `image.tag`                      | Image tag/version                                           | `rpc_community_0629`                         |
| `image.pullPolicy`              | Image pull policy                                           | `IfNotPresent`                               |
| `resources.requests.cpu`        | CPU request for the container                               | `8`                                          |
| `resources.requests.memory`     | Memory request for the container                            | `16Gi`                                       |
| `resources.limits.cpu`          | CPU limit for the container                                 | `16`                                         |
| `resources.limits.memory`       | Memory limit for the container                              | `32Gi`                                       |
| `persistence.storageClass`      | Storage class for persistent volume                         | `standard`                                   |
| `persistence.accessModes`       | Access modes for persistent volume                          | `[ReadWriteOnce]`                            |
| `persistence.size`              | Size of the persistent volume                               | `2000Gi`                                     |
| `customLabels`                  | Additional custom labels for resources                      | `{}`                                         |
| `snapshot.enabled`              | Enable the snapshot restore job (Helm hook)                 | `false`                                      |
| `snapshot.url`                  | URL to the snapshot archive                                 | `""`                                         |
| `snapshot.skipDownload`         | Skip downloading the snapshot archive                       | `false`                                      |
| `snapshot.dataDir`              | Path to the data directory inside the container             | `/data/pharos-node/domain/light/data/public` |
| `monitoring.enabled`            | Enable monitoring                                            | `false`                                      |
| `monitoring.confPath`           | Path to monitoring config file                              | `/data/pharos-node/domain/light/conf/monitor.conf` |
| `monitoring.pushGatewayAddress` | Pushgateway address for metrics                             | `""`                                         |
| `monitoring.pushInterval`       | Push interval in seconds for metrics                        | `5`                                          |
| `monitoring.prometheus.enabled` | Enable Prometheus monitoring                                | `false`                                      |
| `monitoring.alertmanager.enabled` | Enable Alertmanager for alerts                            | `false`                                      |
| `monitoring.prom2teams.enabled` | Enable Prom2Teams for Teams alert integration               | `false`                                      |
| `monitoring.pushGateway.enabled`| Enable PushGateway                                           | `false`                                      |
| `monitoring.grafana.enabled`    | Enable Grafana                                               | `false`                                      |
| `taints.enabled`                | Enable tolerating taints                                    | `true`                                       |
| `taints.key`                    | Taint key to tolerate                                       | `reserved`                                   |
| `taints.operator`               | Taint operator (`Equal`, `Exists`)                          | `Equal`                                      |
| `taints.value`                  | Taint value                                                  | `alpha`                                      |
| `taints.effect`                 | Taint effect (`NoSchedule`, etc.)                           | `NoSchedule`                                 |
| `nodeSelector`                  | Node selector to schedule pods on specific nodes            | `{ kubernetes.io/hostname: mynode.name.com }` |
| `gateway.enabled`               | Enable Envoy Gateway configuration                          | `false`                                      |
| `gateway.controllerName`        | Controller name for the gateway                             | `gateway.envoyproxy.io/gatewayclass-controller` |
| `gateway.className`             | Gateway class name                                          | `envoy`                                      |
| `gateway.allowedRoutes.namespaces.from` | Allowed route namespaces (e.g., `All`)           | `All`                                        |
| `certmanager.enabled`           | Enable cert-manager integration                             | `false`                                      |
| `certmanager.acme.email`        | ACME registration email                                     | `<your-email@example.com>`                   |
| `certmanager.acme.server`       | ACME server URL                                             | `https://acme-v02.api.letsencrypt.org/directory` |
| `certmanager.certificate.name`  | Certificate name                                            | `domain-cert`                                |
| `certmanager.certificate.secretName` | Certificate secret name                               | `domain-cert`                                |
| `certmanager.certificate.dnsNames` | DNS names for certificate                               | `["example.com", "www.example.com"]`         |
| `certmanager.clusterIssuer.name`| ClusterIssuer name                                          | `letsencrypt-demo`                           |
| `certmanager.clusterIssuer.secretName` | ClusterIssuer secret name                         | `letsencrypt-demo`                           |
| `prometheus.enabled`            | Enable Prometheus                                           | `false`                                      |
| `alertmanager.enabled`          | Enable Alertmanager                                         | `false`                                      |
| `prom2teams.enabled`            | Enable Prom2Teams                                           | `false`                                      |
| `pushGateway.enabled`           | Enable PushGateway                                          | `false`                                      |
| `grafana.enabled`               | Enable Grafana                                              | `false`                                      |



---

## Usage

- **Check pod status:**

  ```sh
  kubectl get pods -n <namespace>
  ```

- **Access the service:**

  For `ClusterIP` (default), use port-forwarding:

  ```sh
  kubectl port-forward svc/<release-name> 18100:18100 -n <namespace>
  ```

- **Persistent Data:**

  The chart creates a PersistentVolumeClaim for `/data` inside the container, ensuring blockchain data is retained across pod restarts.

---

## Snapshot Job (Restore from Snapshot)

This chart supports restoring blockchain data from a snapshot using a Kubernetes Job. The snapshot restore is performed by a Job that you can enable or disable via values. **You must manually control when the snapshot Job runs to avoid conflicts with the main StatefulSet.**

### How it works

- The snapshot Job is only created if `snapshot.enabled=true`.
- The Job passes the snapshot URL and data directory directly as environment variables from `values.yaml`.
- The Job will mount the same PVC as the main StatefulSet, so you must ensure the main pod is **not running** before running the snapshot job.

### Steps to Restore from Snapshot

1. **Edit `values.yaml` or use `--set` to enable the snapshot job and set the snapshot URL:**

   ```yaml
   snapshot:
     enabled: true
     url: "https://your-snapshot-url/snapshot.tar.gz"
     dataDir: "/data/pharos-node/domain/light/data/public"
   ```

   ```sh
   helm upgrade --install <release-name> . 
   ```

   Or via CLI:

   ```sh
   helm upgrade --install <release-name> . \
     --set snapshot.enabled=true \
     --set snapshot.url="https://your-snapshot-url/snapshot.tar.gz" \
     --set snapshot.dataDir="/data/pharos-node/domain/light/data/public"
   ```

   **Setting snapshot.enabled=true scales down the pharos-node StatefulSet to 0 replicas.**

2. **After the Job completes, disable the snapshot job:**

   ```sh
   helm upgrade <release-name> . --set snapshot.enabled=false
   ```
    This will remove the snapshot Job and scale the main StatefulSet back up.

3. **You can manually scale the main StatefulSet back up:**

   ```sh
   kubectl scale statefulset <release-name> --replicas=1 -n <namespace>
   ```

### Passing Snapshot Values

The snapshot Job now receives its configuration directly from `values.yaml` via environment variables:

```yaml
env:
  - name: SNAPSHOT_URL
    value: "{{ .Values.snapshot.url }}"
  - name: DATA_DIR
    value: "{{ .Values.snapshot.dataDir }}"
  - name: SKIP_DOWNLOAD
    value: "{{ .Values.snapshot.skipDownload | default 'false' }}"
```

You can customize the snapshot URL and data directory via `values.yaml` or `--set` as shown above.

---


## Configuring Monitoring in Pharos Node

This chart supports configuring the `monitor.conf` file at `/data/pharos-node/domain/light/conf/monitor.conf`. The configuration is performed by a Kubernetes Job that you can enable or disable via values.  
**You must manually control when the monitoring Job runs to avoid conflicts with the main StatefulSet**, as the job mounts the same PVC used by the node.

### How it works

- The monitoring Job is only created if `monitoring.enabled=true`.
- The Job uses values defined in `values.yaml` to populate `monitor.conf`.
- The Job mounts the same PersistentVolumeClaim as the main StatefulSet.
- The main StatefulSet is scaled down when the monitoring Job is enabled to avoid file access conflicts.

### Steps to Run the Monitoring Job

1. **Edit `values.yaml` or use `--set` to enable the monitoring job and set the configuration:**

   ```yaml
   monitoring:
     enabled: true
     confPath: /data/pharos-node/domain/light/conf/monitor.conf
     pushGatewayAddress: "http://pushgateway.example.com:9091"
     pushInterval: 5
   ```

   ```sh
   helm upgrade --install <release-name> . 
   ```

   Or via CLI:

   ```sh
   helm upgrade --install <release-name> . \
     --set monitoring.enabled=true \
     --set monitoring.confPath="/data/pharos-node/domain/light/conf/monitor.conf" \
     --set monitoring.pushGatewayAddress="http://pushgateway.example.com:9091" \
     --set monitoring.pushInterval=5
   ```

   **Setting monitoring.enabled=true scales down the pharos-node StatefulSet to 0 replicas.**

2. **After the Job completes, disable the monitoring job:**

   ```sh
   helm upgrade <release-name> . --set monitoring.enabled=false
   ```
    This will remove the monitoring Job and scale the main StatefulSet back up.

3. **You can manually scale the main StatefulSet back up:**

   ```sh
   kubectl scale statefulset <release-name> --replicas=1 -n <namespace>
   ```

### Uninstalling All Resources

To remove all resources created by this chart, run:

```sh
helm uninstall <release-name> -n <namespace>
```

This command deletes the release and all associated Kubernetes resources, including the StatefulSet, Services, PVCs, and Jobs.  
If you want to keep the persistent data, manually delete the PVC after uninstalling the chart if needed:

```sh
kubectl delete pvc -l app.kubernetes.io/instance=<release-name> -n <namespace>
```


---

## Template Structure

- `statefulset.yaml`: Defines the main Pharos node StatefulSet, including container image, ports, resources, and persistent storage.
- `service.yaml`: Exposes the Pharos node via a Kubernetes Service.
- `snapshot/snapshot_job.yaml`: Defines the snapshot restore Job, passing values directly as environment variables.
- `_helpers.tpl`: Contains helper template functions for consistent naming and labeling.

### Monitoring

- `monitoring/config-job/config-job.yaml`: Defines the Job that configures `monitor.conf` on the mounted PVC.
- `monitoring/prometheus/*`: Templates to deploy Prometheus, including:
  - `configmap.yaml`: Prometheus configuration (scrape configs, alerts).
  - `deployment.yaml`: Prometheus Deployment resource.
  - `httproute.yaml`: Exposes Prometheus via HTTPRoute (Gateway API).
  - `pvc.yaml`: Defines persistent storage for Prometheus.
  - `service.yaml`: Exposes Prometheus as a Service.
- `monitoring/alertmanager/*`: Templates for Alertmanager:
  - `configmap.yaml`: Alertmanager routing and receivers config.
  - `deployment.yaml`: Alertmanager Deployment resource.
  - `httproute.yaml`: Exposes Alertmanager via Gateway API.
  - `pvc.yaml`: Persistent volume for alert storage.
  - `service.yaml`: Service definition for Alertmanager.
- `monitoring/prom2teams/*`: Templates for Prom2Teams integration with Microsoft Teams:
  - `configmap.yaml`, `secrets.yaml`, `deployment.yaml`, `service.yaml`
- `monitoring/grafana/*`: Templates for Grafana dashboard:
  - `configmap.yaml`: Grafana configuration.
  - `dashboard-json.yaml`: Preloaded dashboard definition.
  - `deployment.yaml`: Grafana Deployment.
  - `httproute.yaml`: Gateway exposure.
  - `pvc.yaml`: Persistent volume for Grafana data.
  - `secrets.yaml`: Admin credentials and other secrets.
  - `service.yaml`: Exposes Grafana.
- `monitoring/push-gateway/*`: PushGateway templates:
  - `statefulset.yaml`: Deploys PushGateway with storage.
  - `servicemonitor.yaml`: For Prometheus to scrape PushGateway.
  - `service.yaml`: PushGateway Service definition.

### Gateway

- `gateway/gateway-class.yaml`: Defines a GatewayClass for Envoy Gateway.
- `gateway/gateway.yaml`: Instantiates the Gateway using the class.

### CertManager

- `certmanager/certificate.yaml`: Defines the TLS certificate resource.
- `certmanager/clusterissuer.yaml`: Defines the ACME ClusterIssuer used to sign the certificate.

---

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request.

---
## License
This project is licensed under the Apache License 2.0. See the [LICENSE](LICENSE) file for details.

