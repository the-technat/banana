# 03 - Core Addons

Without any tools, the cluster is not fully functional. Three of the most critical addons we need are [Cilium](https://cilium.io), [Argo CD](https://argoproj.io) and the [HCloud-CCM](https://github.com/hetznercloud/hcloud-cloud-controller-manager). They are installed manually in a first run since they all depend on each other, but later on they will be managed by Argo CD using GitOps.

Note: Core Addons are also the only addons that are assigned to the `system-cluster-critical` priorityClass and get the sync-wave `-5` in argocd.

Since we later on configure those tools to integrate into systems they manage (e.g Argo CD into Dex and so on), the inital config differs from the actual config from GitOps.

Please also note, that secrets in this stage of the installation are all created manually if necessary and later-on injected using proper methods.

## CNI - Cilium

My prefered CNI of choice is cilium. Mostly due to it's extensive usage of eBPF but also it's feature set which allows me configure a host firewall for each nodes dynamically and encrypting all traffic between nodes using wireguard.

Note that we cannot enable the host-firewall at this stage of the installation as it would break many things. The same accounts for default-deny within the cluster using network policies.

We're doing the installation using the helm chart:

```bash
helm repo add cilium https://helm.cilium.io/
helm upgrade -i cilium cilium/cilium -n kube-system -f cilium-default-values.yaml
```

### CCM - HCloud

We don't have a fixed cloud we are running on. But we need someone that manages LoadBalancers for us. So we go for the majority which tends to be Hetzner since most of my nodes (including the control-plane) are running there.

They have a CCM that can be installed using their YAML files:

```bash
kubectl -n kube-system create secret generic hcloud --from-literal=token=<hcloud API token>
kubectl apply -f  https://github.com/hetznercloud/hcloud-cloud-controller-manager/releases/latest/download/ccm.yaml
```

Note that the successful initialization of hcloud-ccm is required for nodes to become ready, since we added the `--cloud-provider=external` flag to the kubelet.
We also ignore the fact that not all nodes are running on hcloud, since the hcloud-ccm can also configure nodes that run somewhere else.

## GitOps - Argo CD

Now finally we have Argo CD as our GitOps tool of choice. Since we are missing an ingress-controller we don't yet expose the UI but rely on the controller to interact with our apps we will soon install.

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
