---
title: Configure test release candidate
linkTitle: Configure test release candidate
description: "How to configure test release candidate in Flux"
weight: 190
---

### Test release candidates configuration in Flux

To enable test release candidates [during bootstrap](/flux/installation/configuration/bootstrap-customization) add the following patches to the flux-system `kustomization.yaml`:

To test release candidates, you can patch the container image tags:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - gotk-components.yaml
  - gotk-sync.yaml
images:
  - name: ghcr.io/fluxcd/source-controller
    newTag: rc-254ba51d
  - name: ghcr.io/fluxcd/kustomize-controller
    newTag: rc-ca0a9b8
```

