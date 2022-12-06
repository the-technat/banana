# 04 - Infrastructure Addons

After the core addons have been installed and argocd started importing a bunch of addons from git, we can now start reconciling the things that need reconcilation.

## Argo CD

Must be adopted by itself to self manage it, adding more configs and also deploying the default netpols again.

## Cilium

Must be adopted by Argo CD to now manage it's values.

## Sealed Secrets Reencryption

TODO
