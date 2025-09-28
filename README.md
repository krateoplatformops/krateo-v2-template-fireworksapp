# Krateo V2 Template Fireworks app

This is a template used to scaffold a toolchain to host and deploy a fully functional frontend App (FireworksApp).

> [!NOTE]
> This template is deprecated from Krateo v2.5.1 - it has been migrated to https://github.com/krateoplatformops-blueprints/github-scaffolding-with-composition-page

This Template implements the following steps:
1. Create an empty Github repository (on github.com) - [link](https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/blob/main/chart/templates/git-repo.yaml)
2. Push the code from the [skeleton](https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/tree/main/skeleton) to the previously create repository - [link](https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/blob/main/chart/templates/git-clone.yaml)
3. A Continuous Integration pipeline (GitHub [workflow](https://github.com/krateoplatformops/krateo-v2-template-fireworksapp/blob/main/skeleton/.github/workflows/ci.yml)) will build the Dockerfile of the frontend app and the resulting image will be published as a Docker image on the GitHub Package registry
4. An ArgoCD Application will be deployed to listen to the Helm Chart of the FireworksApp application and deploy the chart on the same Kubernetes cluster where ArgoCD is hosted
5. The FireworksApp will be deployed with a Service type of NodePort kind exposed on the chosen port.

## Requirements

- Install argo CLI: https://argo-cd.readthedocs.io/en/stable/cli_installation/
- Have Krateo PlatformOps installed: https://docs.krateo.io/

<details>
  <summary>Krateo <= 2.4.3</summary>

  ### Setup toolchain on krateo-system namespace

  ```sh
  helm repo add krateo https://charts.krateo.io
  helm repo update krateo
  helm install github-provider krateo/github-provider --namespace krateo-system --create-namespace --wait --version 0.2.2
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
    name: github-repo-creds
    namespace: krateo-system
  type: Opaque
  EOF
  ```

  ### Create a *fireworksapp-system* namespace

  ```sh
  kubectl create ns fireworksapp-system
  ```

  ## How to install

  ```sh
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/1.1.15/compositiondefinition.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/1.1.15/customform.yaml
  ```

</details>

<details>
  <summary>Krateo == 2.5.0</summary>

  ### Setup toolchain on krateo-system namespace

  ```sh
  helm repo add krateo https://charts.krateo.io
  helm repo update krateo
  helm install github-provider-kog krateo/github-provider-kog --namespace krateo-system --create-namespace --wait --version 0.0.7
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
    name: github-repo-creds
    namespace: krateo-system
  type: Opaque
  EOF
  ```

  ### Wait for GitHub Provider to be ready

  ```sh
  until kubectl get deployment github-provider-kog-repo-controller -n krateo-system &>/dev/null; do
    echo "Waiting for Repo controller deployment to be created..."
    sleep 5
  done
  kubectl wait deployments github-provider-kog-repo-controller --for condition=Available=True --namespace krateo-system --timeout=300s

  ```

  ### Create a *fireworksapp-system* namespace

  ```sh
  kubectl create ns fireworksapp-system
  ```

  ### Create a BearerAuth Custom Resource

  Create a BearerAuth Custom Resource to make the GitHub Provider able to authenticate with the GitHub API using the previously created token.

  ```sh
  cat <<EOF | kubectl apply -f -
  apiVersion: github.kog.krateo.io/v1alpha1
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
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/compositiondefinition.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/restaction.fireworksapp-compositiondefinition.yaml
  ```

  ## Form not ordered in alphabetical order

  ```sh
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/not-ordered/button.fireworksapp-template-panel-button-schema-notordered.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/not-ordered/form.fireworksapp-form-notordered.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/not-ordered/panel.fireworksapp-template-panel-schema-notordered.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/not-ordered/paragraph.fireworksapp-template-panel-paragraph-schema-notordered.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/not-ordered/restaction.fireworksapp-schema-notordered.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/not-ordered/restaction.fireworksapp-template-restaction-cleanup-schema-notordered.yaml
  ```

  ## Form ordered in alphabetical order

  ```sh
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/ordered/button.fireworksapp-template-panel-button-schema-ordered.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/ordered/form.fireworksapp-form-ordered.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/ordered/panel.fireworksapp-template-panel-schema-ordered.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/ordered/paragraph.fireworksapp-template-panel-paragraph-schema-ordered.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/ordered/restaction.fireworksapp-schema-ordered.yaml
  kubectl apply -f https://raw.githubusercontent.com/krateoplatformops/krateo-v2-template-fireworksapp/refs/tags/2.0.2/portal/ordered/restaction.fireworksapp-template-restaction-cleanup-schema-ordered.yaml
  ```

</details>

<details>
  <summary>Krateo == 2.5.1</summary>

  ### Setup toolchain on krateo-system namespace

  ```sh
  helm repo add krateo https://charts.krateo.io
  helm repo update krateo
  helm install github-provider-kog krateo/github-provider-kog --namespace krateo-system --create-namespace --wait --version 0.1.0
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
    name: github-repo-creds
    namespace: krateo-system
  type: Opaque
  EOF
  ```

  ### Wait for GitHub Provider to be ready

  ```sh
  until kubectl get deployment github-provider-kog-repo-controller -n krateo-system &>/dev/null; do
    echo "Waiting for Repo controller deployment to be created..."
    sleep 5
  done
  kubectl wait deployments github-provider-kog-repo-controller --for condition=Available=True --namespace krateo-system --timeout=300s

  ```

  ### Create a *fireworksapp-system* namespace

  ```sh
  kubectl create ns fireworksapp-system
  ```

  ### Create a BearerAuth Custom Resource

  Create a BearerAuth Custom Resource to make the GitHub Provider able to authenticate with the GitHub API using the previously created token.

  ```sh
  cat <<EOF | kubectl apply -f -
  apiVersion: github.kog.krateo.io/v1alpha1
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

  ### Install using Krateo Composable Operation

  Install the CompositionDefinition for the *Blueprint*:

  ```sh
  cat <<EOF | kubectl apply -f -
  apiVersion: core.krateo.io/v1alpha1
  kind: CompositionDefinition
  metadata:
    name: fireworksapp
    namespace: fireworksapp-system
  spec:
    chart:
      repo: fireworks-app
      url: https://charts.krateo.io
      version: 2.0.3
  EOF
  ```

  Install the Blueprint using, as metadata.name, the *Composition* name (the Helm Chart name of the composition):

  ```sh
  cat <<EOF | kubectl apply -f -
  apiVersion: composition.krateo.io/v2-0-3
  kind: FireworksApp
  metadata:
    name: 2-5-0
    namespace: fireworksapp-system
  spec:
    app:
      service:
        port: 31180
        type: NodePort
    argocd:
      application:
        destination:
          namespace: fireworks-app
          server: https://kubernetes.default.svc
        project: default
        source:
          path: chart/
        syncPolicy:
          automated:
            prune: true
            selfHeal: true
      namespace: krateo-system
    git:
      deletionPolicy: Orphan
      fromRepo:
        branch: main
        credentials:
          authMethod: generic
          secretRef:
            key: token
            name: github-repo-creds
            namespace: krateo-system
        name: krateo-v2-template-fireworksapp
        org: krateoplatformops
        path: skeleton/
        scmUrl: https://github.com
      insecure: true
      toRepo:
        apiUrl: https://api.github.com
        branch: main
        credentials:
          authMethod: generic
          secretRef:
            key: token
            name: github-repo-creds
            namespace: krateo-system
        deletionPolicy: delete
        initialize: true
        name: 2-5-0
        org: krateoplatformops-archive
        path: /
        private: false
        scmUrl: https://github.com
      unsupportedCapabilities: true
  EOF
  ```

  ### Install using Krateo Composable Portal

  ```sh
  cat <<EOF | kubectl apply -f -
  apiVersion: core.krateo.io/v1alpha1
  kind: CompositionDefinition
  metadata:
    name: portal-blueprint-page
    namespace: krateo-system
  spec:
    chart:
      repo: portal-blueprint-page
      url: https://marketplace.krateo.io
      version: 1.0.0
  EOF
  ```

  Install the Blueprint using, as metadata.name, the *Blueprint* name (the Helm Chart name of the blueprint):

  ```sh
  cat <<EOF | kubectl apply -f -
  apiVersion: composition.krateo.io/v1-0-0
  kind: PortalBlueprintPage
  metadata:
    name: fireworks-app
    namespace: krateo-system
  spec:
    blueprint:
      version: 2.0.3 # this is the Blueprint version
      hasPage: true
    form:
      alphabeticalOrder: false
    panel:
      title: FireworksApp
      icon:
        name: fa-cubes
  EOF
  ```

</details>

<details>
  <summary>Krateo == 2.6.0</summary>

  ### Setup toolchain on krateo-system namespace

  ```sh
  helm repo add krateo https://charts.krateo.io
  helm repo update krateo
  helm install github-provider-kog krateo/github-provider-kog --namespace krateo-system --create-namespace --wait --version 0.0.7
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
    name: github-repo-creds
    namespace: krateo-system
  type: Opaque
  EOF
  ```

  ### Wait for GitHub Provider to be ready

  ```sh
  until kubectl get deployment github-provider-kog-repo-controller -n krateo-system &>/dev/null; do
    echo "Waiting for Repo controller deployment to be created..."
    sleep 5
  done
  kubectl wait deployments github-provider-kog-repo-controller --for condition=Available=True --namespace krateo-system --timeout=300s

  ```

  ### Create a *fireworksapp-system* namespace

  ```sh
  kubectl create ns fireworksapp-system
  ```

  ### Create a BearerAuth Custom Resource

  Create a BearerAuth Custom Resource to make the GitHub Provider able to authenticate with the GitHub API using the previously created token.

  ```sh
  cat <<EOF | kubectl apply -f -
  apiVersion: github.kog.krateo.io/v1alpha1
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

  ### Install using Krateo Composable Operation

  Install the CompositionDefinition for the *Blueprint*:

  ```sh
  cat <<EOF | kubectl apply -f -
  apiVersion: core.krateo.io/v1alpha1
  kind: CompositionDefinition
  metadata:
    name: fireworksapp
    namespace: fireworksapp-system
  spec:
    chart:
      repo: fireworks-app
      url: https://charts.krateo.io
      version: 2.0.4
  EOF
  ```

  Install the Blueprint using, as metadata.name, the *Composition* name (the Helm Chart name of the composition):

  ```sh
  cat <<EOF | kubectl apply -f -
  apiVersion: composition.krateo.io/v2-0-4
  kind: FireworksApp
  metadata:
    name: 2-6-0
    namespace: fireworksapp-system
  spec:
    app:
      service:
        port: 31180
        type: NodePort
    argocd:
      application:
        destination:
          namespace: fireworks-app
          server: https://kubernetes.default.svc
        project: default
        source:
          path: chart/
        syncPolicy:
          automated:
            prune: true
            selfHeal: true
      namespace: krateo-system
    git:
      deletionPolicy: Orphan
      fromRepo:
        branch: main
        credentials:
          authMethod: generic
          secretRef:
            key: token
            name: github-repo-creds
            namespace: krateo-system
        name: krateo-v2-template-fireworksapp
        org: krateoplatformops
        path: skeleton/
        scmUrl: https://github.com
      insecure: true
      toRepo:
        apiUrl: https://api.github.com
        branch: main
        credentials:
          authMethod: generic
          secretRef:
            key: token
            name: github-repo-creds
            namespace: krateo-system
        deletionPolicy: delete
        initialize: true
        name: 2-5-0
        org: krateoplatformops-archive
        path: /
        private: false
        scmUrl: https://github.com
      unsupportedCapabilities: true
  EOF
  ```

  ### Install using Krateo Composable Portal

  ```sh
  cat <<EOF | kubectl apply -f -
  apiVersion: core.krateo.io/v1alpha1
  kind: CompositionDefinition
  metadata:
    name: portal-blueprint-page
    namespace: krateo-system
  spec:
    chart:
      repo: portal-blueprint-page
      url: https://marketplace.krateo.io
      version: 1.0.5
  EOF
  ```

  Install the Blueprint using, as metadata.name, the *Blueprint* name (the Helm Chart name of the blueprint):

  ```sh
  cat <<EOF | kubectl apply -f -
  apiVersion: composition.krateo.io/v1-0-5
  kind: PortalBlueprintPage
  metadata:
    name: fireworks-app
    namespace: krateo-system
  spec:
    blueprint:
      version: 2.0.4 # this is the Blueprint version
      hasPage: true
    form:
      alphabeticalOrder: false
    panel:
      title: FireworksApp
      icon:
        name: fa-cubes
  EOF

</details>