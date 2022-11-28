# Cluster Initialization

This is done primarly using my guide [here](https://github.com/the-technat/the-technat/blob/main/content/Kubernetes/k8s_kubeadm.md).

But there are of course some steps to highlight.

## Init config

This is my final init config:

- [Kubeadm config reference](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta3/#kubeadm-k8s-io-v1beta3-JoinConfiguration)

```yaml
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
skipPhases:
  - addon/kube-proxy
localAPIEndpoint:
  advertiseAddress: 100.96.250.68
---
apiVersion: kubeadm.k8s.io/v1beta3
clusterName: banana
kind: ClusterConfiguration
kubernetesVersion: 1.25.4
networking:
  dnsDomain: banana.k8s
  serviceSubnet: 10.111.0.0/16
  podSubnet: 10.222.0.0/16
controlPlaneEndpoint: 100.96.250.68:6443
etcd:
  local:
    extraArgs:
    peerCertSANs:
    - "100.96.250.68"
apiServer:
  certSANs:
  - "100.96.250.68"
scheduler:
  extraArgs:
    address: "100.96.250.68"
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd # must match the value you set for containerd
```

## Graceful node shutdown

See this [guide](https://kubernetes.io/docs/concepts/architecture/nodes/#graceful-node-shutdown) for more details, the fields are already present in `/var/lib/kubelet/config.yaml` so just change this.

## CNI Installation

For now, just install cilium with default values (except for kube-proxy). We're configuring the features we want to have later on:

```bash
helm upgrade -i cilium cilium/cilium -n kube-system --set kubeProxyReplacement=strict --set k8sServiceHost=100.96.250.68 --set k8sServicePort=6443
```

## Argo CD

Now that we have a CNI, install Argo CD with default values:

```bash
helm upgrade -i argocd argo/argo-cd -n argocd --create-namespace
```

We will see that argo cd get's the right values later on...
