# GOIT ArgoCD Project

Automated deployment to **AWS EKS** using **Terraform + Helm + ArgoCD**.

------------------------------------------------------------------------

## Objective

-   Deploy **ArgoCD** to an existing EKS cluster via **Terraform** (as a
    Helm release in `infra-tools`).
-   Keep a **Git repository** with Kubernetes/Helm configuration.
-   Manage an **ArgoCD Application** that auto-syncs from Git and
    deploys a demo app (Nginx).
-   Verify the deployment and show how to access the service.

------------------------------------------------------------------------

## Repository Structure

    goit-argo/
    ├── application.yaml                 # ArgoCD Application (auto-sync, self-heal, CreateNamespace)
    └── namespaces/
        ├── infra-tools/
        │   └── ns.yaml                  # Namespace for ArgoCD
        └── application/
            ├── ns.yaml                  # Namespace for demo app
            └── nginx.yaml               # Demo Nginx Deployment + Service

> **Note:** ArgoCD itself is installed via Terraform (separate infra
> repo), using a values file `values/argocd-values.yaml` and Helm
> release into `infra-tools`.

------------------------------------------------------------------------

## How to Run

### 1) Deploy / Verify ArgoCD (Terraform)

In your **infrastructure** repo (not this one), under something like:
`eks-vpc-cluster/goit-argo/terraform/argocd/`

``` bash
terraform init
terraform apply
```

ArgoCD will be installed into the namespace `infra-tools`.

Verify pods:

``` bash
kubectl get pods -n infra-tools
```

> You should see several pods with the `argocd-` prefix in
> `Running`/`Completed` state.

------------------------------------------------------------------------

### 2) Open ArgoCD UI

Port-forward the ArgoCD server service:

``` bash
kubectl port-forward svc/argocd-server -n infra-tools 8080:443
```

Open the UI: **https://localhost:8080**

Login credentials:

``` bash
# username
admin

# password
kubectl get secret argocd-initial-admin-secret -n infra-tools -o jsonpath="{.data.password}" | base64 --decode
```

------------------------------------------------------------------------

### 3) Create ArgoCD Application (from this repo)

This repository contains `application.yaml` that defines an ArgoCD
Application: - `repoURL` → this Git repository - `path` →
`namespaces/application` - `syncPolicy.automated` → enabled (auto-sync +
self-heal) - `syncOptions: CreateNamespace=true` → ArgoCD will create
the target namespace automatically

Apply it straight from GitHub raw:

``` bash
kubectl apply -f https://raw.githubusercontent.com/MykolaKushka/mlops-cicd-lesson-7/main/application.yaml
```

You should see:

    application.argoproj.io/nginx-demo created

------------------------------------------------------------------------

### 4) Observe Sync & Check Pods

In the ArgoCD UI you'll see **nginx-demo**: - **Status:** `Healthy` -
**Sync:** `Synced`

Or via CLI:

``` bash
kubectl get applications -n infra-tools
kubectl get pods -n application
```

Expected:

    NAME                      READY   STATUS    RESTARTS   AGE
    nginx-xxxxxxxxx-xxxxx     1/1     Running   0          <age>

------------------------------------------------------------------------

### 5) Access the Demo Service

Use port-forward to reach Nginx locally:

``` bash
kubectl port-forward svc/nginx -n application 8081:80
```

Open: **http://localhost:8081**

------------------------------------------------------------------------

## What's Implemented (per assignment)

-   **ArgoCD** installed via **Terraform** as a Helm release in
    `infra-tools`
-   **argocd-values.yaml** (in infra repo) with
    service/rbac/args/timeouts
-   **application.yaml** in this repo with:
    -   `repoURL`, `path`, and Helm/K8s configs for the app
    -   `automated` sync + `selfHeal`
    -   `CreateNamespace=true`
-   After `git push`, ArgoCD **auto-deploys** manifests →
    **Deployment/Service/Pod**
-   Service accessible via `kubectl port-forward`

------------------------------------------------------------------------

## Clean Up

> To avoid cloud costs, destroy resources when done.

-   Delete ArgoCD Application:

``` bash
kubectl delete -f https://raw.githubusercontent.com/MykolaKushka/mlops-cicd-lesson-7/main/application.yaml
```

-   (In your infra repo) destroy ArgoCD / cluster as needed:

``` bash
terraform destroy
```

> You may keep the **S3 backend** (Terraform state) to avoid losing
> state history.

------------------------------------------------------------------------

## Useful Links

-   Repo: **https://github.com/MykolaKushka/mlops-cicd-lesson-7**
-   ArgoCD UI (local via port-forward): **https://localhost:8080**
-   Demo access (local via port-forward): **http://localhost:8081**

------------------------------------------------------------------------

## Author


This project is part of the GOIT DS&DA DevOps track (ArgoCD homework).
