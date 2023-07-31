---
title: Disable Kubernetes cluster role aggregations
linkTitle: Disable Kubernetes cluster role aggregations
description: "How to disable Kubernetes cluster role aggregations in Flux"
weight: 170
---


### Disable Kubernetes cluster role aggregations

By default, Flux [RBAC](/flux/security/#controller-permissions) grants Kubernetes builtin `view`, `edit` and `admin` roles
access to Flux custom resources. To disable the RBAC aggregation, you can remove the `flux-view` and `flux-edit`
cluster roles.
 
To disable role aggregation [during bootstrap](_index.md) add the following patches to the flux-system `kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - gotk-components.yaml
  - gotk-sync.yaml
patches:
  - patch: |
      apiVersion: rbac.authorization.k8s.io/v1
      kind: ClusterRole
      metadata:
        name: flux
      $patch: delete
    target:
      kind: ClusterRole
      name: "(flux-view|flux-edit)-flux-system"
```

