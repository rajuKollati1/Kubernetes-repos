
# 🚀 Complete FluxCD Setup Guide (GitHub Personal Repository)

This guide explains how to install and set up **FluxCD** from scratch on a Kubernetes cluster and connect it to your personal GitHub repository for full GitOps automation.

---

## 🧩 Prerequisites

Before you begin, ensure you have:

- ✅ A running **Kubernetes cluster**
- ✅ `kubectl` configured and pointing to your cluster
- ✅ Installed **Flux CLI**
- ✅ A **GitHub account**
- ✅ Internet access for your cluster to reach GitHub

---

## ⚙️ Step 1 — Install Flux CLI

To install Flux CLI on your local machine:

### On macOS / Linux:

```bash
curl -s https://fluxcd.io/install.sh | sudo bash
```

### On Windows (PowerShell):

```powershell
iwr -useb https://fluxcd.io/install.ps1 | iex
```

Then verify installation:

```bash
flux --version
```

---

## 🔑 Step 2 — Create a GitHub Personal Access Token (PAT)

Flux needs a GitHub token with permission to push and pull from your repository.

1. Go to [GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)](https://github.com/settings/tokens)
2. Click **Generate new token (classic)**
3. Name it (e.g., `flux-bootstrap-token`)
4. Set the following scopes:
   - ✅ `repo` (Full control)
   - ✅ `workflow`
5. Click **Generate token** and copy it securely

---

## 🧠 Step 3 — Set Your Token in Environment

For PowerShell (Windows):

```powershell
$env:GITHUB_TOKEN="<YOUR_TOKEN>"
```

For Linux/macOS:

```bash
export GITHUB_TOKEN=<YOUR_TOKEN>
```

---

## ⚙️ Step 4 — Bootstrap FluxCD with GitHub

Run the following command (replace with your GitHub username and repo name):

```bash
flux bootstrap github   --owner=rajuKollati1   --repository=TestTest   --branch=main   --path=clusters/my-lab   --personal   --token-auth
```

### What this does:
- Installs Flux components in your Kubernetes cluster
- Creates or updates the GitHub repository (`TestTest`)
- Pushes generated Flux manifests to the repo
- Configures Flux to continuously sync cluster state with the Git repository

---

## 🔍 Step 5 — Verify Installation

After a few minutes, check that Flux components are running:

```bash
kubectl get pods -n flux-system
```

Expected output (pods should all be `Running`):

```
NAME                                      READY   STATUS    RESTARTS   AGE
helm-controller-xxxxxxxxxx-xxxxx          1/1     Running   0          2m
kustomize-controller-xxxxxxxxxx-xxxxx     1/1     Running   0          2m
notification-controller-xxxxxxxxxx-xxxxx  1/1     Running   0          2m
source-controller-xxxxxxxxxx-xxxxx        1/1     Running   0          2m
```

---

## ✅ Step 6 — Verify Git Sync

Run the following command:

```bash
flux get all -A
```

Expected output:

```
NAMESPACE   NAME                          READY   STATUS
flux-system GitRepository/flux-system     True    Fetched revision: main/<commit>
flux-system Kustomization/flux-system     True    Applied revision: main/<commit>
```

If both are **READY=True**, your GitOps setup is fully functional.

---

## 🧩 Step 7 — Deploy Your First Application

Now that Flux is syncing with Git, you can deploy an app declaratively.

Create a folder inside your repo:

```
apps/nginx/
```

Add a file `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

Commit and push it:

```bash
git add .
git commit -m "Add Nginx deployment"
git push origin main
```

Flux will detect and apply it automatically within a few minutes.

---

## 🧾 Step 8 — Check Application Deployment

Verify the app was deployed:

```bash
kubectl get deployments
kubectl get pods
```

You should see `nginx` pods running.

---

## 🎉 Done!

You now have a complete **GitOps pipeline** using **FluxCD** and **GitHub**:

- Any change in your Git repo automatically updates your cluster.
- Your entire Kubernetes configuration is stored and versioned in Git.
- You can reproduce or roll back environments anytime.

---

## 🧰 Optional: Uninstall FluxCD

If you want to remove Flux:

```bash
flux uninstall --namespace flux-system
```

---

> **Author:** Kollati Raju  
> **Purpose:** Full end-to-end FluxCD setup for personal GitHub GitOps workflow
