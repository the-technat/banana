# 04 - Infrastructure Addons

After the core addons have been installed and argocd started importing a bunch of addons from git, we can now start reconciling the things that need reconcilation.

## Argo CD

Must be adopted by itself to self manage it, adding more configs and also deploying the default netpols again. Sync it has sync-wave -5, you should probably already see how it has read the config from git and is trying to match this state. This won't work since the ingress controller is missing, but that's fine.

## Cilium

Should also have wave -5 and should be in sync as the config doesn't really change or only a bit.

## Sealed Secrets Reencryption

In sync-wave -5 we also got the sealed secrets controller. This component is used to decrypt all secrets in the cluster, and since it is deployed after argocd, we now need to reencrypt all the secrets we want to deploy and we already deployed.

To do this, you need the original values of your secrets again and need to go through every app.

Here's how you encrypt a secret written in K8s yaml:

```bash
kubeseal --format yaml <input.yaml >output.yaml
```
