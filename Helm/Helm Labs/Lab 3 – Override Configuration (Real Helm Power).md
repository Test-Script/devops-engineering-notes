==========================================================================================================================
                                    Override Configuration (Real Helm Power)
==========================================================================================================================

Let’s change replica count.

    helm upgrade my-nginx bitnami/nginx --set replicaCount=3

What Happens Internally

    Helm:

- Re-renders templates
- Compares old vs new manifest
- Applies PATCH via Kubernetes API

This is not a blind reapply — it performs a 3-way merge:

- Old manifest
- Live cluster state
- New manifest