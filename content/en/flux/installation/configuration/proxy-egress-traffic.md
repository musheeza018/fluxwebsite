---
title: Configure HTTP/S proxy for egress traffic
linkTitle: Configure HTTP/S proxy for egress traffic
description: "How to configure HTTP/S proxy for egress traffic in Flux"
weight: 130
---
HTTP/S proxy for egress traffic in Flux is to act as an intermediary between the 
Flux components running in a Kubernetes cluster and external resources, such as 
container registries, Git repositories, or any other services accessed during 
the deployment and synchronization process.

### Set HTTP/S proxy for cluster controller

If your cluster must use an HTTP proxy to reach GitHub or other external services,
you must set `NO_PROXY=.cluster.local.,.cluster.local,.svc`
to allow the Flux controllers to talk to each other.

To enable HTTP/S proxy for egress traffic [during bootstrap](/flux/installation/configuration/bootstrap-customization) add the following patches to the flux-system `kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - gotk-components.yaml
  - gotk-sync.yaml
patches:
  - patch: |
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: all
      spec:
        template:
          spec:
            containers:
              - name: manager
                env:
                  - name: "HTTPS_PROXY"
                    value: "http://proxy.example.com:3129"
                  - name: "NO_PROXY"
                    value: ".cluster.local.,.cluster.local,.svc"
    target:
      kind: Deployment
      labelSelector: app.kubernetes.io/part-of=flux
```
