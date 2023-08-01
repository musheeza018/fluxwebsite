---
title: Configure Git repository access via SOCKS5 SSH proxy
linkTitle: Configure Git repository access via SOCKS5 SSH proxy
description: "How to configure Git repository access via SOCKS5 SSH proxy in Flux"
weight: 140
---

SOCKS5 SSH proxy for Git repository access establishs a secure and encrypted tunnel, 
ensuring safe communication between the local machine and the remote Git repository.

### Create SSH Key Secret

Create a Kubernetes Secret that contains your SSH private key. This key will be used for 
authenticating with the Git repository. 
Save your private key into a file (e.g., id_rsa) and then create the Secret

### Configure proxy settings

Identify the address and port of the SOCKS5 SSH proxy server you want to use. 
Obtain this information from your network administrator or the proxy service provider.


### Git repository access via SOCKS5 SSH proxy

If your cluster has Internet restrictions, requiring egress traffic to go
through a proxy, you must use a SOCKS5 SSH proxy to be able to reach GitHub
(or other external Git servers) via SSH.

### Set environment variable

To configure a SOCKS5 proxy set the environment variable `ALL_PROXY` to allow
both source-controller and image-automation-controller to connect through the
proxy.

```
ALL_PROXY=socks5://<proxy-address>:<port>
```

The following is an example of patching the Flux setup kustomization to add the
`ALL_PROXY` environment variable in source-controller and
image-automation-controller.

To enable Git repository access via SOCKS5 SSH proxy [during bootstrap](_index.md) add the following patches to the flux-system `kustomization.yaml`:

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
                  - name: "ALL_PROXY"
                    value: "socks5://proxy.example.com:1080"
    target:
      kind: Deployment
      labelSelector: app.kubernetes.io/part-of=flux
      name: "(source-controller|image-automation-controller)"
```
