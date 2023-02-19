# 03 - Core Addons

Without any tools, the cluster is not fully functional. Two of the most critical addons we need are [Cilium](https://cilium.io), [Argo CD](https://argoproj.io) and the [Hcloud-CCM](https://github.com/hetznercloud/hcloud-cloud-controller-manager). They are installed manually in a first run, but later on managed by Argo CD using GitOps.

Note: Core Addons are also addons that are assigned to the `system-cluster-critical` priorityClass and get the sync-wave `-5` in argocd.

This means that the default values shown here differ from what is actually set in the repo's configs.

Please note also, that secrets in this stage of the installation are all created manually if necessary and later-on injected using proper methods.

## CNI - Cilium

We're doing this using the helm chart:

```bash
helm repo add cilium https://helm.cilium.io/
helm upgrade -i cilium cilium/cilium -n kube-system -f cilium-default-values.yaml
```

Note: CoreDNS will try to start now (since the nodes are not `Ready`) but this will fail because we have no netpol for it. We'll fix that with Argo CD.

### CCM - Hcloud

Since the majority of worker nodes will be running on hcloud, we install the related ccm.

```bash
kubectl -n kube-system create secret generic hcloud --from-literal=token=<hcloud API token>
kubectl apply -f  https://github.com/hetznercloud/hcloud-cloud-controller-manager/releases/latest/download/ccm.yaml
```

Note: Argo CD with some default netpols is required to get this up and running the first time.

## GitOps - Argo CD

Installed as well using helm:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm upgrade -i argocd argo/argo-cd -n argocd --create-namespace -f argocd-default-values.yaml
```

For now you can connect to Argo CD using the following command:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl port-forward svc/argocd-server 8080:80 # open http://localhost:8080
```

**Change the default admin password!**

## Next Steps

-> [04 Infrastructure Addons](./04_infra_addons.md)
