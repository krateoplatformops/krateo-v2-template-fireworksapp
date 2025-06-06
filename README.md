# Krateo V2 Template Fireworks app

This is a template used to scaffold a toolchain to host and deploy a fully functional frontend App (FireworksApp).

This Template implements the following steps:
1. Create an empty Github repository (on github.com) - [link](https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/blob/main/chart/templates/git-repo.yaml)
2. Push the code from the [skeleton](https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/tree/main/skeleton) to the previously create repository - [link](https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/blob/main/chart/templates/git-clone.yaml)
3. A Continuous Integration pipeline (GitHub [workflow](https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/blob/main/skeleton/.github/workflows/ci.yml)) will build the Dockerfile of the frontend app and the resulting image will be published as a Docker image on the GitHub Package registry
4. An ArgoCD Application will be deployed to listen to the Helm Chart of the FireworksApp application and deploy the chart on the same Kubernetes cluster where ArgoCD is hosted
5. The FireworksApp will be deployed with a Service type of NodePort kind exposed on the chosen port.

## Requirements

- Install argo CLI: https://argo-cd.readthedocs.io/en/stable/cli_installation/
- Have Krateo PlatformOps installed: https://docs.krateo.io/

### Setup toolchain on krateo-system namespace

```sh
helm repo add krateo https://charts.krateo.io
helm repo update krateo
helm install github-provider-kog-chart krateo/github-provider-kog-chart --namespace krateo-system --create-namespace
helm install git-provider krateo/git-provider --namespace krateo-system --create-namespace
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update argo
helm install argocd argo/argo-cd --namespace krateo-system --create-namespace --wait
```

### Create a *krateo-account* user on ArgoCD

```sh
kubectl patch configmap argocd-cm -n krateo-system --patch '{"data": {"accounts.krateo-account": "apiKey, login"}}'
kubectl patch configmap argocd-rbac-cm -n krateo-system --patch '{"data": {"policy.default": "role:readonly"}}'
```

### Generate a token for *krateo-account* user

In order to generate a token, follow this instructions:

```sh
kubectl port-forward service/argocd-server -n krateo-system 8443:443
```

Open a new terminal and execute the following commands:

```sh
PASSWORD=$(kubectl -n krateo-system get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
argocd login localhost:8443 --insecure --username admin --password $PASSWORD
argocd account list
TOKEN=$(argocd account generate-token --account krateo-account)

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: argocd-endpoint
  namespace: krateo-system
stringData:
  insecure: "true"
  server-url: https://argocd-server.krateo-system.svc:443
  token: $TOKEN
EOF
```

### Generate a token for GitHub user

In order to generate a token, follow this instructions: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic

Give the following permissions: delete:packages, delete_repo, repo, workflow, write:packages

Substitute the <PAT> value with the generated token:

```sh
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

### Check if github-provider is ready

```sh
TODO
```

### Create a *fireworksapp-system* namespace

```sh
kubectl create ns fireworksapp-system
```

### Create a BearerAuth Custom Resource

```sh
cat <<EOF | kubectl apply -f -
apiVersion: github.krateo.io/v1alpha1
kind: BearerAuth
metadata:
  name: bearer-github-ref
  namespace: fireworksapp-system
spec:
  tokenRef:
    key: token
    name: github-repo-creds
    namespace: krateo-system
EOF
```

## How to install

```sh
kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/heads/main/compositiondefinition.yaml
```

### With *Krateo Composable Portal*

#### With classic form

```sh
kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/heads/main/form.yaml
```

#### With custom form

```sh
kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/heads/main/customform.yaml
```