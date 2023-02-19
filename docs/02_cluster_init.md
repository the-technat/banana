# 02 Cluster Initialization

## Init config

This is my final init config based on the [Kubeadm config reference](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta3/#kubeadm-k8s-io-v1beta3-JoinConfiguration):

```yaml
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
skipPhases:
  - addon/kube-proxy
---
apiVersion: kubeadm.k8s.io/v1beta3
clusterName: banana
kind: ClusterConfiguration
kubernetesVersion: 1.26.1
networking:
  dnsDomain: banana.k8s
  serviceSubnet: 10.111.0.0/16
  podSubnet: 10.222.0.0/16
controlPlaneEndpoint: api.alleaffengaffen.ch:6443
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd # must match the value you set for containerd
```

Note: I'm using an DNS record here for the kubeapi to make sure we could add more masters in the future.

## Graceful node shutdown

See this [guide](https://kubernetes.io/docs/concepts/architecture/nodes/#graceful-node-shutdown) for more details, the fields are already present in `/var/lib/kubelet/config.yaml` so just change them to `30s`, respectively `20s` for cricial pods.

## Multiple master nodes

Not yet a requirement, but they would require more configration than the join command prints out.

## Worker nodes

They don't need a special join config as the kubelet listens on `0.0.0.0`. Just copy the join command and paste it on the worker nodes.

## Next Step

- > [03 Core Addons](./03_core_addons.md)
