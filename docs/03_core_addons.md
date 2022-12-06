# 03 - Core Addons

Without any tools, the cluster is not fully functional. Two of the most critical addons we need are [Cilium](https://cilium.io) and [Argo CD](https://argoproj.io). They are installed manually in a first run, but later on managed by Argo CD using GitOps.

This menas that the default values shown here differ from what is actually set in the repo's configs.

## CNI - Cilium

We're doing this using the helm chart:

```bash
helm repo add cilium https://helm.cilium.io/
helm upgrade -i cilium cilium/cilium -n kube-system -f cilium-default-values.yaml
```

The default-values are the following:

```yaml
rollOutCiliumPods: true
priorityClassName: "system-node-critical"
annotateK8sNode: true
policyEnforcementMode: "always"
operator:
  replicas: 1
  rollOutPods: true
containerRuntime:
  integration: containerd
  socketPath: /var/run/containerd/containerd.sock
hubble:
  enabled: true
  rollOutPods: true
  relay:
    enabled: true
    rollOutPods: true
  ui:
    enabled: true
    rollOutPods: true
ipam:
  mode: kubernetes
kubeProxyReplacement: "strict"
k8sServiceHost: "hawk.alleaffengaffen.ch"
k8sServicePort: "6443"
```

## GitOps - Argo CD

Installed as well using helm:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm upgrade -i argocd argo/argo-cd -n argocd --create-namespace -f argocd-default-values.yaml
```

The default-values are the following:

```yaml
global:
  networkPolicy:
    create: true

extraObjects:
- apiVersion: argoproj.io/v1alpha1
  kind: AppProject
  metadata:
    name: addons
    namespace: argocd
    annotations:
      argocd.argoproj.io/hook: PreSync
  spec:
    clusterResourceWhitelist:
    - group: '*'
      kind: '*'
    description: Cluster infrastructure addons
    destinations:
    - name: in-cluster
      namespace: '*'
      server: https://kubernetes.default.svc
    namespaceResourceWhitelist:
    - group: '*'
      kind: '*'
    sourceRepos:
    - '*'
- apiVersion: argoproj.io/v1alpha1
  kind: AppProject
  metadata:
    name: apps
    namespace: argocd
    annotations:
      argocd.argoproj.io/hook: PreSync
  spec:
    clusterResourceWhitelist:
    - group: '*'
      kind: '*'
    description: Applications
    destinations:
    - name: in-cluster
      namespace: '*'
      server: https://kubernetes.default.svc
    namespaceResourceWhitelist:
    - group: '*'
      kind: '*'
    sourceRepos:
    - https://github.com/alleaffengaffen/banana.git
- apiVersion: argoproj.io/v1alpha1
  kind: Application
  metadata:
    name: app-of-apps
    namespace: argocd
    annotations:
      argocd.argoproj.io/hook: PreSync
  spec:
    project: addons
    source:
      repoURL: https://github.com/alleaffengaffen/banana.git
      targetRevision: HEAD
      path: apps
    destination:
      server: https://kubernetes.default.svc
      namespace: argocd
    syncPolicy:
      automated:
        prune: true
        selfHeal: true
        allowEmpty: false
      syncOptions:
      - CreateNamespace=true
      - ServerSideApply=true
      retry:
        limit: 5
        backoff:
          duration: 5s
          factor: 2
          maxDuration: 5m
- apiVersion: cilium.io/v2
  kind: CiliumNetworkPolicy
  metadata:
    name: "allow-argocd-egress"
    namespace: argocd
  spec:
    endpointSelector:
      matchLabels:
        app.kubernetes.io/component: repo-server
        app.kubernetes.io/part-of: argocd
    egress:
    - toEntities:
      - "all"
- apiVersion: cilium.io/v2
  kind: CiliumNetworkPolicy
  metadata:
    name: coredns
    namespace: kube-system
  spec:
    endpointSelector: {}
    egress:
      - toEndpoints:
          - matchLabels:
              io.kubernetes.pod.namespace: kube-system
              k8s-app: kube-dns
        toPorts:
          - ports:
              - port: "53"
                protocol: UDP
            rules:
              dns:
                - matchPattern: "*"
- apiVersion: cilium.io/v2
  kind: CiliumClusterwideNetworkPolicy
  metadata:
    name: "allow-egress-to-cluster"
  spec:
    endpointSelector: {}
    egress:
    - toEntities:
      - "cluster"
- apiVersion: "cilium.io/v2"
  kind: CiliumClusterwideNetworkPolicy
  metadata:
    name: "cilium-health-checks"
  spec:
    endpointSelector:
      matchLabels:
        'reserved:health': ''
    ingress:
      - fromEntities:
        - remote-node
    egress:
      - toEntities:
        - remote-node
- apiVersion: cilium.io/v2
  kind: CiliumNetworkPolicy
  metadata:
    name: kube-system
    namespace: kube-system
  spec:
    endpointSelector: {}
    ingress:
      - fromEntities:
        - all
    egress:
      - toEntities:
        - all
```

Note: this will add some objects that onboard all other apps from Git. They require some reconcilation loops but that's fine.

For now you can connect to Argo CD using the following command:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl port-forward svc/argocd-server 8080:80 # open http://localhost:8080
```

## Next Steps

-> [04 Infrastructure Addons](./04_infra_addons.md)
