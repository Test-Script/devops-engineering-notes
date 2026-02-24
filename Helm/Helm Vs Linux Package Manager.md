==========================================================================================================
                            Kubernetes Package Manager and Linux Package Managers
==========================================================================================================

1. Helm : Package manager for Kubernetes applications.

It packages:

- Kubernetes manifests (Deployment, Service, ConfigMap, etc.)
- Configuration templates.
- Versioned releases.

It installs applications inside a Kubernetes cluster.

2. Linux Package Manager: Linux package managers install system-level software on an operating system.

Examples:

- apt (Debian/Ubuntu)
- yum / dnf (RHEL/CentOS/Fedora)
- rpm
- pacman

They manage:

- Binaries.
- Libraries.
- System services.
- OS dependencies.

==========================================================================================================
                                Core Architectural Difference
==========================================================================================================

| Dimension         | Helm                         | Linux Package Manager       |
| ----------------- | ---------------------------- | --------------------------- |
| Layer             | Kubernetes Control Plane     | Operating System            |
| Scope             | Cluster-scoped               | Host-scoped                 |
| Installs          | Kubernetes resources         | OS packages                 |
| Package Format    | `.tgz` (Helm Chart)          | `.deb`, `.rpm`              |
| Dependency Type   | Chart dependencies           | Binary/library dependencies |
| State Tracking    | Stored as Kubernetes Secrets | Stored in local package DB  |
| Upgrade Mechanism | Declarative re-apply         | File replacement + scripts  |

==========================================================================================================
                                Technical Deep Dive Comparison
==========================================================================================================

## Installation Target: 

Installs resources via Kubernetes API Server, Uses REST calls, and Stores release metadata as Secrets in cluster.

    helm install nginx bitnami/nginx

This will install these components inside the Clusters : Deployment, Service, ReplicaSet, Pods, and ConfigMaps. 

## Dependency Resolution:

Dependencies are Other helm charts, and defined in charts.yaml

dependencies:
  - name: redis
    version: 17.x.x

## State Management:

Helm maintains release state as Kubernetes Secret (type: helm.sh/release.v1), and Includes rendered manifest + values.

This enables : helm rollback, helm history, and helm upgrade.

Cluster-aware state.

========================================================================================================
