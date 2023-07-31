---
title: Enable Helm repositories caching 
linkTitle: Enable Helm repositories caching
description: "How to enable Helm repositories caching in Flux"
weight: 90
---

### Enable Helm repositories caching

For large Helm repository index files, you can enable
caching to reduce the memory footprint of source-controller. To enable Helm repositories caching [during bootstrap](_index.md) add the following patches to the flux-system `kustomization.yaml`:

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

