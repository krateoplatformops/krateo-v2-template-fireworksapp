# Krateo V2 Template Fireworks app

This is a template used to scaffold a toolchain to host and deploy a fully functional frontend App (FireworksApp).

This Template implements the following steps:
1. Create an empty Github repository (on github.com) - [link](https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/blob/main/chart/templates/git-repo.yaml)
2. Push the code from the [skeleton](https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/tree/main/skeleton) to the previously create repository - [link](https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/blob/main/chart/templates/git-clone.yaml)
3. A Continuous Integration pipeline (GitHub [workflow](https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/blob/main/skeleton/.github/workflows/ci.yml)) will build the Dockerfile of the frontend app and the resulting image will be published as a Docker image on the GitHub Package registry
4. An ArgoCD Application will be deployed to listen to the Helm Chart of the FireworksApp application and deploy the chart on the same Kubernetes cluster where ArgoCD is hosted
5. The FireworksApp will be deployed with a Service type of NodePort kind exposed on the chosen port.

## How to install

```sh
apiVersion: core.krateo.io/v1alpha1
kind: CompositionDefinition
metadata:
  annotations:
     "krateo.io/connector-verbose": "true"
  name: fireworksapp-tgz
  namespace: demo-system
spec:
  chart:
    url: https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/releases/download/0.1.0/fireworks-app-0.1.0.tgz
```

## Requirements

```sh
helm repo add krateo https://charts.krateo.io
helm repo update krateo
helm install github-provider krateo/github-provider --namespace krateo-system --create-namespace
helm install git-provider krateo/git-provider --namespace krateo-system --create-namespace
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update argo
helm install argocd argo/argo-cd --namespace krateo-system --create-namespace --wait

cat <<EOF | kubectl apply -f -
apiVersion: v1
stringData:
  token: <PAT>
kind: Secret
metadata:
  name: github-repo-creds
  namespace: krateo-system
type: Opaque
EOF
```
