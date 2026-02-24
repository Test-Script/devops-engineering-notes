==========================================================================================================================
                                            Rollback (Release Versioning)
==========================================================================================================================

1. Check history:

    helm history my-nginx

2. Rollback:

    helm rollback my-nginx 1

================================================================
                        Internal Mechanism
================================================================

Helm stores every revision in a Kubernetes Secret:

    sh.helm.release.v1.my-nginx.v1
    sh.helm.release.v1.my-nginx.v2

Rollback:

    Retrieves stored manifest
    Re-applies it
