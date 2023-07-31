---
title: "Flux vertical scaling"
linkTitle: "Vertical scaling"
description: "How to configure vertical scaling for Flux controllers"
weight: 60
---

When Flux is managing hundreds of applications that are deployed multiple times per day, cluster admins
can fine tune the Flux controller at [bootstrap time](_index.md) to run at scale by:

- [Increasing the number of workers and resource limits](#increase-the-number-of-workers-and-limits)
- [Enabling Helm repository caching to reduce memory usage](#enable-helm-repositories-caching)
- [Enabling persistent storage for internal artifacts](/flux/installation/configuration/internal-artifacts)
- [Running the Flux controllers on dedicated nodes](#node-affinity-and-tolerations)

{{% alert color="info" title="Horizontal scaling" %}}
When vertical scaling is not an option, you can use sharding to horizontally scale
the Flux controllers. For more details please see the [sharding guide](sharding.md).
{{% /alert %}}

## Increase the number of workers and limits

If Flux is managing hundreds of applications, it is advised to increase the number of reconciliations
that can be performed in parallel and to bump the resources limits:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - gotk-components.yaml
  - gotk-sync.yaml
patches:
  - patch: |
      - op: add
        path: /spec/template/spec/containers/0/args/-
        value: --concurrent=20
      - op: add
        path: /spec/template/spec/containers/0/args/-
        value: --requeue-dependency=5s
    target:
      kind: Deployment
      name: "(kustomize-controller|helm-controller|source-controller)"
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
                resources:
                  limits:
                    cpu: 2000m
                    memory: 2Gi
    target:
      kind: Deployment
      name: "(kustomize-controller|helm-controller|source-controller)"
```

## Enable Helm repositories caching

If Flux connects to Helm repositories hosting hundreds of Helm charts,
it is advised to enable caching to reduce the memory footprint of source-controller:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - gotk-components.yaml
  - gotk-sync.yaml
patches:
  - patch: |
      - op: add
        path: /spec/template/spec/containers/0/args/-
        value: --helm-cache-max-size=10
      - op: add
        path: /spec/template/spec/containers/0/args/-
        value: --helm-cache-ttl=60m
      - op: add
        path: /spec/template/spec/containers/0/args/-
        value: --helm-cache-purge-interval=5m
    target:
      kind: Deployment
      name: source-controller
```

When `helm-cache-max-size` is reached, an error is logged and the index is instead
read from file. Cache hits are exposed via the `gotk_cache_events_total` Prometheus
metrics. Use this data to fine-tune the configuration flags.
