# 04 - Infrastructure Addons

After the core addons have been installed and argocd started importing a bunch of addons from git, we can now start reconciling the things that need reconcilation.

Infrastructure addons are important too, but not crucial for the cluster to work properly, therefore they get the sync-wave `-3` and the priorityClass `infra` that was created by the Argo CD addititonal values.

## Sealed Secrets Reencryption

This component is used to decrypt all secrets in the cluster, and since it is deployed after argocd, we now need to reencrypt all the secrets we want to deploy and we already deployed.

To do this, you need the original values of your secrets again and need to go through every app.

Here's how you encrypt a secret written in K8s yaml:

```bash
kubeseal --format yaml <input.yaml >output.yaml
```

## Kyverno

Nothing to do here, it got applied automatically and is hopefully already reconciled and injected a bunch of policies.

## Cert-Manager

Reencrypt the `infomaniak-api-credentials` secret.

## Ingress-nginx

Reencrypt the `tailscale-config` secret so that it can reach tailscale.

Note: The ingress resource will report the ClusterIP of the ingress-nginx service as it's external IP. This will cause external-dns to not work. Solve this either by manually adding the DNS record or adding the following annotation to your ingress:

```yaml
annotations:
  external-dns.alpha.kubernetes.io/target: ingress-nginx-proxy.little-cloud.ts.net
```

Don't forget to authorize and tag the new machine in tailscale once it comes up the first time! Otherwise it won't be able to talk to the other services.
