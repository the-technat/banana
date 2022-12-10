# 03 - Core Addons

Without any tools, the cluster is not fully functional. Two of the most critical addons we need are [Cilium](https://cilium.io) and [Argo CD](https://argoproj.io). They are installed manually in a first run, but later on managed by Argo CD using GitOps.

This menas that the default values shown here differ from what is actually set in the repo's configs.

## CNI - Cilium

We're doing this using the helm chart:

```bash
helm repo add cilium https://helm.cilium.io/
helm upgrade -i cilium cilium/cilium -n kube-system -f cilium-default-values.yaml
```

Note: CoreDNS will try to start now (since the nodes are not `Ready`) but this will fail because we have no netpol for it. We'll fix that with Argo CD.

## GitOps - Argo CD

Installed as well using helm:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm upgrade -i argocd argo/argo-cd -n argocd --create-namespace -f argocd-default-values.yaml
```

Note: this will add some objects that onboard all other apps from Git. They require some reconcilation loops but that's fine as we do this in the next section.

For now you can connect to Argo CD using the following command:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl port-forward svc/argocd-server 8080:80 # open http://localhost:8080
```

** Change the default admin password! **

## Next Steps

-> [04 Infrastructure Addons](./04_infra_addons.md)
