# Core Addons

Cilium is installed, Argo CD is installed but both with the default config.

The next thing we are going to do is installing some core addons, but in a very special way.

## App of Apps

The first thing to do is creating an Argo CD app that watches our repository for app definitions, so that we they are automatically deployed.

Here's our app of apps:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: addons
  namespace: argocd
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
  - https://github.com/alleaffengaffen/banana.git
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app-of-apps
spec:
  destination:
    name: ''
    namespace: argocd
    server: 'https://kubernetes.default.svc'
  source:
    path: definitions
    repoURL: 'https://github.com/alleaffengaffen/banana.git'
    targetRevision: HEAD
  project: addons
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply this to the cluster into the argocd namespace and you're almost done, as Argo CD now starts onboarding your apps.

## Argo CD

## Cilium

# Sealed Secrets Controller
