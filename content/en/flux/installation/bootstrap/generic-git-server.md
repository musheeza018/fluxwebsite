---
title: Generic Git Server
linkTitle: Generic Git server
description: "How to bootstrap Flux with Generic Git Server"
weight: 20
---
### Generic Git Server

The `bootstrap git` command takes an existing Git repository, clones it and
commits the Flux components manifests to the specified branch. Then it
configures the target cluster to synchronize with that repository.

{{% alert color="warning" %}}
:warning: Note that if set, your SSH hostname and port could be overwritten by your [ssh_config](https://linux.die.net/man/5/ssh_config).
{{% /alert %}}

Run bootstrap for a Git repository and authenticate with your SSH agent:

```sh
flux bootstrap git \
  --url=ssh://git@<host>/<org>/<repository> \
  --branch=<my-branch> \
  --path=clusters/my-cluster
```

The above command will generate an SSH key (defaults to ECDSA P-384 but can be changed with `--ssh-key-algorithm` and 
`--ssh-ecdsa-curve`), and it will prompt you to add the SSH public key as a deploy key to your repository.

If you want to use your own SSH key, you can provide a private key using
`--private-key-file=<path/to/private.key>` (you can supply the passphrase with `--password=<key-passphrase>`).
This option can also be used if no SSH agent is available on your machine.

{{% alert color="info" title="Bootstrap options" %}}
There are many options available when bootstrapping Flux, such as installing a subset of Flux components,
setting the Kubernetes context, changing the Git author name and email, enabling Git submodules, and more.
To list all the available options run `flux bootstrap git --help`.
{{% /alert %}}

If your Git server doesn't support SSH, you can run bootstrap for Git over HTTPS:

```sh
flux bootstrap git \
  --url=https://<host>/<org>/<repository> \
  --username=<my-username> \
  --password=<my-password> \
  --token-auth=true \
  --path=clusters/my-cluster
```

If your Git server uses a self-signed TLS certificate, you can specify the CA file with
`--ca-file=<path/to/ca.crt>`.

With `--path` you can configure the directory which will be used to reconcile the target cluster.
To control multiple clusters from the same Git repository, you have to set a unique path per
cluster e.g. `clusters/staging` and `clusters/production`:

```sh
./clusters/
├── staging # <- path=clusters/staging
│   └── flux-system # <- namespace dir generated by bootstrap
│       ├── gotk-components.yaml
│       ├── gotk-sync.yaml
│       └── kustomization.yaml
└── production # <- path=clusters/production
    └── flux-system
```

After running bootstrap you can place Kubernetes YAMLs inside a dir under path
e.g. `clusters/staging/my-app`, and Flux will reconcile them on your cluster.

For examples on how you can structure your Git repository see:

* [flux2-kustomize-helm-example](https://github.com/fluxcd/flux2-kustomize-helm-example)
* [flux2-multi-tenancy](https://github.com/fluxcd/flux2-multi-tenancy)

