# 02 Cluster Initialization

## Init config

This is my final init config based on the [Kubeadm config reference](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta3/#kubeadm-k8s-io-v1beta3-JoinConfiguration):

```yaml
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
skipPhases:
  - addon/kube-proxy
localAPIEndpoint:
  advertiseAddress: 100.95.10.95 # IP on tailscale0 NIC
---
apiVersion: kubeadm.k8s.io/v1beta3
clusterName: banana
kind: ClusterConfiguration
kubernetesVersion: 1.25.4
networking:
  dnsDomain: banana.k8s
  serviceSubnet: 10.111.0.0/16
  podSubnet: 10.222.0.0/16
controlPlaneEndpoint: hawk.alleaffengaffen.ch:6443 # public DNS entry
etcd:
  local:
    extraArgs:
      advertise-client-urls: "https://100.95.10.95:2379"
      initial-advertise-peer-urls: "https://100.95.10.95:2380"
      initial-cluster: "hawk=https://100.95.10.95:2380"
      listen-client-urls: "https://127.0.0.1:2379,https://100.95.10.95:2379"
      listen-peer-urls: "https://100.95.10.95:2380"
    serverCertSANs:
    - "100.95.10.95"
    peerCertSANs:
    - "100.95.10.95"
apiServer:
  certSANs:
  - "100.95.10.95"
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd # must match the value you set for containerd
```

Note: I'm using an DNS record here for the kubeapi to make sure we could add more masters in the future + I make sure the public IP is nowhere hardcoded in the configs. For single master node that's not necessary as the FW is closed and noone connects to the etcd. But if at a future time I'd like to add more masters, they need to communicate over the tailnet.

## Graceful node shutdown

See this [guide](https://kubernetes.io/docs/concepts/architecture/nodes/#graceful-node-shutdown) for more details, the fields are already present in `/var/lib/kubelet/config.yaml` so just change them to `30s`, respectively `20s` for cricial pods.

## Multiple master nodes

Not yet a requirement, but they would require more configration than the join command prints out.

## Worker nodes

They don't need a special join config as the kubelet listens on `0.0.0.0`. Just copy the join command and paste it on the worker nodes.

## Next Step

-> [03 Core Addons](./03_core_addons.md)
