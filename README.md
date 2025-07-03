# Pharos Testnet Helm Chart

This Helm chart deploys the Pharos testnet using a Kubernetes StatefulSet and Service. It is designed for easy configuration, and management of the Pharos blockchain testnet nodes.

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
- [Template Structure](#template-structure)
- [Contributing](#contributing)
- [License](#license)

---

## Prerequisites

- Kubernetes 1.12+ cluster
- Helm 3.x installed
- Sufficient cluster resources for the requested CPU, memory, and storage

---

## Installation

1. **Clone the repository or copy the chart directory:**

   ```sh
   git clone <repo-url>
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

## Upgrading

To upgrade the release with new values:

```sh
helm upgrade <release-name> . -f values.yaml
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

| Parameter                       | Description                                         | Default                                      |
|----------------------------------|-----------------------------------------------------|----------------------------------------------|
| `namespace`                     | Namespace to deploy resources                       | `pharos-testnet`                             |
| `nameOverride`                  | Override the release name                           | `""`                                         |
| `image.repository`              | Container image repository                          | `public.ecr.aws/k2g7b7g1/pharos/testnet`     |
| `image.tag`                     | Image tag/version                                   | `rpc_community_0629`                         |
| `image.pullPolicy`              | Image pull policy                                   | `IfNotPresent`                               |
| `resources.requests.cpu`        | CPU request for the container                       | `8`                                          |
| `resources.requests.memory`     | Memory request for the container                    | `16Gi`                                       |
| `resources.limits.cpu`          | CPU limit for the container                         | `16`                                         |
| `resources.limits.memory`       | Memory limit for the container                      | `32Gi`                                       |
| `persistence.storageClass`      | Storage class for persistent volume                 | `standard`                                   |
| `persistence.accessModes`       | Access modes for persistent volume                  | `[ReadWriteOnce]`                            |
| `persistence.size`              | Size of the persistent volume                       | `2000Gi`                                     |
| `customLabels`                  | Additional custom labels for resources              | `{}`                                         |
| `snapshot.enabled`              | Enable the snapshot restore job (Helm hook)         | `false`                                      |
| `snapshot.url`                  | URL to the snapshot archive                         | `""`                                         |
| `snapshot.dataDir`              | Path to the data directory inside the container     | `/data/pharos-node/domain/light/data/public`  |

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

1. **Scale down the main StatefulSet and wait for the pod to terminate:**

   ```sh
   kubectl scale statefulset <release-name> --replicas=0 -n <namespace>
   kubectl get pods -l app.kubernetes.io/instance=<release-name> -n <namespace>
   # Wait until no pods are running
   ```

2. **Edit `values.yaml` or use `--set` to enable the snapshot job and set the snapshot URL:**

   ```yaml
   snapshot:
     enabled: true
     url: "https://your-snapshot-url/snapshot.tar.gz"
     dataDir: "/data/pharos-node/domain/light/data/public"
   ```

   Or via CLI:

   ```sh
   helm upgrade --install <release-name> . \
     --set snapshot.enabled=true \
     --set snapshot.url="https://your-snapshot-url/snapshot.tar.gz" \
     --set snapshot.dataDir="/data/pharos-node/domain/light/data/public"
   ```

3. **Upgrade or install the chart:**

   ```sh
   helm upgrade --install <release-name> .
   ```

   The snapshot Job will be created and run alongside other resources.  
   **Ensure the main StatefulSet is scaled down before this step.**

4. **After the Job completes, disable the snapshot job:**

   ```sh
   helm upgrade <release-name> . --set snapshot.enabled=false
   ```

5. **Scale the main StatefulSet back up:**

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
```

You can customize the snapshot URL and data directory via `values.yaml` or `--set` as shown above.

---

## Template Structure

- statefulset.yaml: Defines the main Pharos node StatefulSet, including container image, ports, resources, and persistent storage.
- service.yaml: Exposes the Pharos node via a Kubernetes Service.
- snapshot/snapshot_job.yaml: Defines the snapshot restore Job, passing values directly as environment variables.
- _helpers.tpl: Contains helper template functions for consistent naming and labeling.

---

## Example values.yaml

```yaml
namespace: pharos-testnet

image:
  repository: public.ecr.aws/k2g7b7g1/pharos/testnet
  tag: "rpc_community_0629"
  pullPolicy: IfNotPresent

resources:
  requests:
    cpu: "8"
    memory: "16Gi"
  limits:
    cpu: "16"
    memory: "32Gi"

persistence:
  storageClass: "standard"
  accessModes:
    - ReadWriteOnce
  size: 2000Gi

customLabels: {}

snapshot:
  enabled: false
  url: ""
  dataDir: "/data/pharos-node/domain/light/data/public"
```

---

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request.

