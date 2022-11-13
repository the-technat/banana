# Core Addons

Cilium is installed, some pods are coming up, others won't because of missing NPs and more.
The next thing we are going to do is installing some core addons.

## Argo CD

We need argo-cd for everthing. So make sure our already installed Argo CD can manage itself.

[Guide](https://github.com/the-technat/the-technat/blob/main/content/Kubernetes/argocd.md)

My values file can be found within this folder.

For the "Login with Github" I created a secret first:

```bash
kubectl create namespace argocd
kubectl create secret generic github --from-literal clientSecret=<pasteSecretHere>
```

Then Argo CD can be installed. Some components won't be healthy at start, but that's normal because some controllers are missing.

## Manage Argo CD with Argo CD

Now that Argo CD is installed, make sure it managed itself using the GH repo.

## Argo CD configures cilium

Now that Argo CD is in place, make sure it replaces the original default values of cilium with our desired ones.

## Onboard addons

Now make sure Argo CD can read the git repo and onboards all the other apps we need + their secrets (we wont create any secret manualy other than the github clientSecret).
