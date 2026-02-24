==========================================================================================================================
                    Helm : Organized by lifecycle phase for production-grade Kubernetes environments.
==========================================================================================================================

## 1️⃣ Repository Management:

| Operation    | Command                      | Purpose                      |
| ------------ | ---------------------------- | ---------------------------- |
| Add repo     | `helm repo add <name> <url>` | Register chart repository    |
| List repos   | `helm repo list`             | Show configured repositories |
| Update repos | `helm repo update`           | Refresh repo index           |
| Remove repo  | `helm repo remove <name>`    | Delete repository            |
| Search repo  | `helm search repo <keyword>` | Search charts in repos       |
| Search hub   | `helm search hub <keyword>`  | Search ArtifactHub           |

## 2️⃣ Chart Discovery & Inspection:

| Operation       | Command                     | Purpose               |
| --------------- | --------------------------- | --------------------- |
| Show chart info | `helm show chart <chart>`   | Metadata (Chart.yaml) |
| Show values     | `helm show values <chart>`  | Default values.yaml   |
| Show readme     | `helm show readme <chart>`  | Documentation         |
| Inspect all     | `helm show all <chart>`     | Full chart details    |
| Pull chart      | `helm pull <chart>`         | Download chart        |
| Pull & untar    | `helm pull <chart> --untar` | Extract locally       |

## 3️⃣ Install Operations:

| Operation                | Command                                             | Purpose               |
| ------------------------ | --------------------------------------------------- | --------------------- |
| Install release          | `helm install <release> <chart>`                    | Deploy chart          |
| Install specific version | `helm install <r> <chart> --version x.x.x`          | Version pin           |
| Install with values file | `helm install <r> <chart> -f values.yaml`           | Custom config         |
| Set inline values        | `helm install <r> <chart> --set key=value`          | Override values       |
| Dry run                  | `helm install <r> <chart> --dry-run`                | Simulate install      |
| Debug mode               | `helm install <r> <chart> --debug`                  | Verbose output        |
| Atomic install           | `helm install <r> <chart> --atomic`                 | Rollback on failure   |
| Create namespace         | `helm install <r> <chart> -n ns --create-namespace` | Namespace auto-create |

## 4️⃣ Upgrade Operations:

| Operation           | Command                                   | Purpose                |
| ------------------- | ----------------------------------------- | ---------------------- |
| Upgrade release     | `helm upgrade <release> <chart>`          | Update deployment      |
| Upgrade with values | `helm upgrade <r> <chart> -f values.yaml` | Apply new config       |
| Reuse values        | `helm upgrade <r> <chart> --reuse-values` | Keep existing config   |
| Atomic upgrade      | `helm upgrade <r> <chart> --atomic`       | Rollback if fails      |
| Install if missing  | `helm upgrade --install <r> <chart>`      | Idempotent CI/CD usage |

## 5️⃣ Rollback & History:

| Operation          | Command                              | Purpose            |
| ------------------ | ------------------------------------ | ------------------ |
| View history       | `helm history <release>`             | List revisions     |
| Rollback           | `helm rollback <release> <revision>` | Revert version     |
| Rollback with wait | `helm rollback <r> <rev> --wait`     | Wait for readiness |

## 6️⃣ Release Management:

| Operation           | Command                       | Purpose                |
| ------------------- | ----------------------------- | ---------------------- |
| List releases       | `helm list`                   | Show deployed releases |
| List all namespaces | `helm list -A`                | Cluster-wide view      |
| Status              | `helm status <release>`       | Detailed release state |
| Get values          | `helm get values <release>`   | Current config         |
| Get manifest        | `helm get manifest <release>` | Rendered YAML          |
| Get all             | `helm get all <release>`      | Full release details   |
| Uninstall           | `helm uninstall <release>`    | Remove release         |

## 7️⃣ Template & Debugging:

| Operation         | Command                           | Purpose               |
| ----------------- | --------------------------------- | --------------------- |
| Render templates  | `helm template <release> <chart>` | Generate YAML locally |
| Lint chart        | `helm lint <chart>`               | Validate chart        |
| Dependency build  | `helm dependency build`           | Download subcharts    |
| Dependency update | `helm dependency update`          | Update dependencies   |
| Verify chart      | `helm verify <chart>`             | Check chart signature |

## 8️⃣ Chart Creation & Packaging:

| Operation        | Command                    | Purpose             |
| ---------------- | -------------------------- | ------------------- |
| Create new chart | `helm create <name>`       | Scaffold chart      |
| Package chart    | `helm package <chart-dir>` | Create `.tgz`       |
| Sign chart       | `helm package --sign`      | Add signature       |
| Repo index       | `helm repo index`          | Generate index.yaml |

## 9️⃣ Advanced / Production Flags:

| Flag                | Usage           | Meaning                 |
| ------------------- | --------------- | ----------------------- |
| `--wait`            | install/upgrade | Wait for pods ready     |
| `--timeout`         | install/upgrade | Set operation timeout   |
| `--atomic`          | install/upgrade | Auto rollback           |
| `--force`           | upgrade         | Recreate resources      |
| `--cleanup-on-fail` | install         | Delete failed resources |
| `--dry-run`         | all             | Preview changes         |
| `--debug`           | all             | Detailed logs           |

