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
   cd pharos-testnet-chart
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

## Template Structure

- statefulset.yaml: Defines the main Pharos node StatefulSet, including container image, ports, resources, and persistent storage.
- service.yaml: Exposes the Pharos node via a Kubernetes Service.
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
```

---

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request.

