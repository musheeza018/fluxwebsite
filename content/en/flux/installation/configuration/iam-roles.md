---
title: Configure IAM Roles for Service Accounts
linkTitle: Configure IAM Roles for Service Accounts
description: "How to configure IAM Roles for Service Accounts"
weight: 160
---

IAM roles play an essential part in controlling access to cloud resources for Flux and other Kubernetes components.

### IAM Roles for setup controller service

To allow Flux access to an AWS service such as KMS or S3, after setting up IRSA,
you can annotate the controller service account with the role ARN.

To enable IAM roles [during bootstrap](_index.md) add the following patches to the flux-system `kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - gotk-components.yaml
  - gotk-sync.yaml
patches:
  - patch: |
      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: kustomize-controller
        annotations:
          eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/<KMS-ROLE-NAME>
    target:
      kind: ServiceAccount
      name: kustomize-controller
  - patch: |
      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: source-controller
        annotations:
          eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/<S3-ROLE-NAME>
    target:
      kind: ServiceAccount
      name: source-controller
```
