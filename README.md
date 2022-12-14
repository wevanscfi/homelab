# Description
This is a homelab setup for getting a local k8s environment up and running, utilizing argocd and the ingress-nginx controller

# Dependencies
- git
- kubectl
- argocd-cli
- linkerd-cli
- step-cli
- kubeseal
- go-yq (yq v4)

# Installation
You will need a fork of this repo for your own homelab. Make sure to update the references to git repo in charts and manifests.

## Argocd
Assuming that you have a local k8s cluster up and running, and have kubectl / helm pointed to it:

run `make install` from the root dir

This will run an initial helm install of argocd and set it up to self manage and automatically sync, and will also go ahead and install critical cluster services.

To access argo cd once installed, get the admin user password from the k8s secret `make argocd-password`

Then visit https://argocd.localhost 

## Linkerd
The linkerd service is split into four applications; linkerd-crds, linkerd-bootstrap, linkerd, and linkerd viz.

The CRDs are automatically installed, however we will need to use the clusters sealed secrets service to manage a new trust anchor root CA for linkerd.

Generate your new root CA and seal with kubeseal.
```
make generate-trust-anchor

```

Validate that the trust anchor sealed secret and public cert are updated and then commit in your new sealed trust anchor.
```
git diff

git add --all

git commit -m 'generate trust anchor'

git push origin main
```

Finally, we will need to sync all of the linkerd applications in order.
```
# Set the sealed secret with our trush anchor and use it to create our issuer certificate
argocd app sync linkerd-bootstrap

# Make sure to wait for certificate from bootstrap to be finsihed being issued by cert-manager
argocd app sync linkerd

# Finally, enable the linkerd-viz plugin if you want it running
argocd app sync linkerd-viz

# Validate your deployment
linkerd check
linkerd viz dashboard
```

# Cluster services
## Ingress
For cluster ingress, this homelab utilized the ingress-nginx controller. For most local k8s installations, this should bind to your localhost without any additional configuration or resolvers needed.

## Prometheus
This homelab utilizes kube-prom-stack for observability. This stack is not automatically created, so it will need to be synced from the argocd ui or cli.
```
argocd app sync prometheus
```

To access the grafana dashboard, get the admin user secret
```
make grafana-password
```

Then visit http://grafana.localhost

## Cert-Manager
cert-manager is installed to handle automatic issue and rotation of certificates.

## Sealed-Secrets
sealed-secrets is installed to handle asymetric encryption of secrets that need to be committed into source control. This is a Development use case, and it is not recommended to utilize this method in a production environment.

## Vault
vault is installed to handle storage of secrets that may not need to be committed into source control. 

