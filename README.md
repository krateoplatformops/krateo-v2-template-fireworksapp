# Krateo V2 Template Fireworks app

This is a template used to scaffold a toolchain to host and deploy a fully functional frontend App (FireworksApp).

## Requirements

- Install argo CLI: https://argo-cd.readthedocs.io/en/stable/cli_installation/
- Have Krateo PlatformOps installed: https://docs.krateo.io/

<details>
  <summary>Krateo <= 2.4.3</summary>

  ### Setup toolchain on krateo-system namespace

  ```sh
  helm repo add krateo https://charts.krateo.io
  helm repo update krateo
  helm install gitlab-provider-kog krateo/gitlab-provider-kog --namespace krateo-system --create-namespace --wait --version 0.0.1
  helm install git-provider krateo/git-provider --namespace krateo-system --create-namespace --wait --version 0.10.1
  helm repo add argo https://argoproj.github.io/argo-helm
  helm repo update argo
  helm install argocd argo/argo-cd --namespace krateo-system --create-namespace --wait --version 8.0.17
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
    name: gitlab-repo-creds
    namespace: krateo-system
  type: Opaque
  EOF
  ```

  ### Create a *gitlab-system* namespace

  ```sh
  kubectl create ns gitlab-system
  ```

  ## How to install

  ```sh
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-gitlab-scaffolding/refs/tags/0.1.0/compositiondefinition.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-gitlab-scaffolding/refs/tags/0.1.0/customform.yaml
  ```

</details>