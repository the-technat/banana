# 04 - Infrastructure Addons

After the core addons have been installed we can start onboarding all apps properly. Argo CD and Cilium aren't finally configured with the default. This was just necessary to make sure we can continue with the next step.

Infrastructure addons are important too, but not crucial for the cluster to work properly, therefore they get the sync-wave `-3` and the priorityClass `infra`.

## Onboarding

To start with the infrastrucutre addons, we deploy the app-of-apps for argocd:

```bash
kubectl apply -f default-app-of-apps.yaml
```

This will onboard the app-of-apps which inturn will onboard all other apps from Git, including Cilium and Argo CD, so we got a self-managed Argo CD. Of course some things will break now and more than one reconcile-loop is necessary to get things up and running, but this is how K8s works ;).

## Akeyless Gateway

The most important thing we need to reconcile manually is the Akeyless gateway that provides us with secrets from akeyless.io, a managed vault service.
