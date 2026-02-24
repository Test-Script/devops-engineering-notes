==========================================================================================================================
                                    Install a Public Chart (Fastest Way to Start)
==========================================================================================================================

1. Adding the Helm Repository:

    helm repo add bitnami https://charts.bitnami.com/bitnami

2. Update Repository:

    helm repo update        | Refresh chart index metadata.

3. Install Chart:

    helm install my-nginx bitnami/nginx

    ## What This Means
    
    my-nginx → Release name
    bitnami/nginx → Chart reference

- What Helm Does Internally
- Downloads chart package (.tgz)
- Extracts it
- Renders templates using:
- values.yaml
- Default chart values
- Sends final YAML to Kubernetes API server
- Stores release metadata as:

    Secret:
    sh.helm.release.v1.my-nginx.v1

- Check:

    kubectl get all
    kubectl get secrets | grep helm

## Architecture Flow:

Helm CLI
   ↓
Render Templates (Go Templating)
   ↓
Final Kubernetes YAML
   ↓
Kubernetes API Server
   ↓
Objects Created