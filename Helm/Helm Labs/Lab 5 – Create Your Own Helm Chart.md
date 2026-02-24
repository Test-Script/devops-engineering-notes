==========================================================================================================================
                                                Create Our Own Helm Chart
==========================================================================================================================

Now real learning begins.

1. Create Chart Skeleton

    helm create my-app

Folder Structure

my-app/
  Chart.yaml
  values.yaml
  templates/
    deployment.yaml
    service.yaml
    _helpers.tpl

## Chart.yaml

Metadata

    name: my-app
    version: 0.1.0
    appVersion: "1.0"

Purpose:

- Chart version
- App version
- Dependencies

## values.yaml

Default configurable values

replicaCount: 1
image:
  repository: nginx
  tag: latest

Purpose: Central configuration layer.

## templates/deployment.yaml

Contains Go template syntax:

    replicas: {{ .Values.replicaCount }}

This is templating engine.

