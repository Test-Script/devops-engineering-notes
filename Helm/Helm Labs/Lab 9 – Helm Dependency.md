==============================================================================================================
                                            Helm Dependency
==============================================================================================================

Add in Chart.yaml:

    dependencies:
  - name: redis
    version: 17.0.0
    repository: https://charts.bitnami.com/bitnami

Then:

    helm dependency update

Purpose: Pull child charts.