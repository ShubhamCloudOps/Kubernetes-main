# A Complete Overview of ArgoCD with a Practical Example


---

## Table of Contents

- [What is ArgoCD?](#what-is-argocd)
- [What is GitOps?](#what-is-gitops)
- [How to Achieve GitOps Using Argo CD?](#how-to-achieve-gitops-using-argo-cd)
- [Benefits of Argo CD](#benefits-of-argo-cd)
- [Prerequisites](#prerequisites)
- [How to Install ArgoCD](#how-to-install-argocd)
- [Application Manifest Files](#application-manifest-files)
- [ArgoCD Application Configuration](#argocd-application-configuration)
- [Applying the Configuration](#applying-the-configuration)
- [Making Changes via GitOps](#making-changes-via-gitops)

---

## What is ArgoCD?

Argo CD is a **Kubernetes-native continuous deployment (CD) tool**. Unlike external CD tools that only enable push-based deployments, Argo CD can pull updated code from Git repositories and deploy it directly to Kubernetes resources.

---

## What is GitOps?

GitOps is a way of implementing **Continuous Deployment for cloud-native applications**. It focuses on a developer-centric experience when operating infrastructure, by using tools developers are already familiar with, including Git and Continuous Deployment tools.

The core idea of GitOps is to have a Git repository that always contains **declarative descriptions** of the infrastructure currently desired in the production environment and an automated process to make the production environment match the described state in the repository.

If you want to deploy a new application or update an existing one, you only need to update the repository — the automated process handles everything else. It's like having **cruise control** for managing your applications in production.

---

## How to Achieve GitOps Using Argo CD?

Every enterprise uses Git as its source code management software to store code. Developers can commit their infrastructure configurations, such as Kubernetes resource definitions, in Git to create environments needed for application deployment.

The workflow is:

1. Developer implements a feature (with new application + K8s configs) and merges to the main branch.
2. The CI process is initiated for generating and testing an image.
3. After review and approval, the pull request is merged with the main branch.
4. The GitOps agent (Argo CD) identifies the new version of configuration recently merged and compares it with the running application in the destination environment (pre-prod or prod).
5. In case of a mismatch, Argo CD highlights **out-of-sync** status and uses the Kubernetes controller to reconcile the new changes to cluster resources.
6. Once Kubernetes resources are ready, it informs the user the application is **in sync**.

Argo CD also uses an agent to **constantly monitor** the end environment and check its status with Git. As all records of changes are stored in Git, Argo CD helps **roll back applications to previous states in a single click**.

---

## Benefits of Argo CD

### 1. Improve Developer Productivity
Argo CD provides developers with a self-service environment for application deployment. Software development teams can focus on creativity and writing business logic instead of time and energy on manual and remedial deployments.

### 2. Improved Software Delivery Compliance
Allow your developers, Ops, and DevOps teams to use a single platform for infrastructure change management. Apply organizational policies to restrict access to Kubernetes resources and minimize your application downtime and outages.

### 3. Increased Collaboration in SDLC
While working on Argo CD, every team member can work from the same system to achieve GitOps and understand the status of individual processes. The single Git repository fosters collaboration amongst team members by assigning tasks to individuals and deploying code from each person as necessary.

### 4. Faster Deployments
Argo CD allows teams to perform more rapid deployments into Kubernetes clusters across multi-cloud. Quicker releases of application changes mean shorter time to market and more flexibility in responding to customer demand.

---

## Prerequisites

- A running Kubernetes cluster (e.g., Minikube)

---

## How to Install ArgoCD

### Step 1: Create the namespace for ArgoCD

```bash
kubectl create namespace argocd
```

### Step 2: Install ArgoCD

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Step 3: Verify the installation

```bash
kubectl get all -n argocd
```

Expected output:

```
NAME                                                    READY   STATUS    RESTARTS   AGE
pod/argocd-application-controller-0                     1/1     Running   0          106m
pod/argocd-applicationset-controller-787bfd9669-4mxq6   1/1     Running   0          106m
pod/argocd-dex-server-bb76f899c-slg7k                   1/1     Running   0          106m
pod/argocd-notifications-controller-5557f7bb5b-84cjr    1/1     Running   0          106m
pod/argocd-redis-b5d6bf5f5-482qq                        1/1     Running   0          106m
pod/argocd-repo-server-56998dcf9c-c75wk                 1/1     Running   0          106m
pod/argocd-server-5985b6cf6f-zzgx8                      1/1     Running   0          106m

NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/argocd-applicationset-controller          ClusterIP   10.102.163.101   <none>        7000/TCP,8080/TCP            106m
service/argocd-dex-server                         ClusterIP   10.101.227.215   <none>        5556/TCP,5557/TCP,5558/TCP   106m
service/argocd-metrics                            ClusterIP   10.111.59.189    <none>        8082/TCP                     106m
service/argocd-notifications-controller-metrics   ClusterIP   10.96.102.185    <none>        9001/TCP                     106m
service/argocd-redis                              ClusterIP   10.97.229.117    <none>        6379/TCP                     106m
service/argocd-repo-server                        ClusterIP   10.102.16.58     <none>        8081/TCP,8084/TCP            106m
service/argocd-server                             ClusterIP   10.98.71.135     <none>        80/TCP,443/TCP               106m
service/argocd-server-metrics                     ClusterIP   10.109.248.207   <none>        8083/TCP                     106m

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/argocd-applicationset-controller   1/1     1            1           106m
deployment.apps/argocd-dex-server                  1/1     1            1           106m
deployment.apps/argocd-notifications-controller    1/1     1            1           106m
deployment.apps/argocd-redis                       1/1     1            1           106m
deployment.apps/argocd-repo-server                 1/1     1            1           106m
deployment.apps/argocd-server                      1/1     1            1           106m

NAME                                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/argocd-applicationset-controller-787bfd9669   1         1         1       106m
replicaset.apps/argocd-dex-server-bb76f899c                   1         1         1       106m
replicaset.apps/argocd-notifications-controller-5557f7bb5b    1         1         1       106m
replicaset.apps/argocd-redis-b5d6bf5f5                        1         1         1       106m
replicaset.apps/argocd-repo-server-56998dcf9c                 1         1         1       106m
replicaset.apps/argocd-server-5985b6cf6f                      1         1         1       106m

NAME                                             READY   AGE
statefulset.apps/argocd-application-controller   1/1     106m
```

### Step 4: Expose the ArgoCD UI

To access the ArgoCD UI, edit the service to enable NodePort:

```bash
kubectl edit svc argocd-server -n argocd
```

### Step 5: Retrieve the Admin Password

The default username is `admin`. The password is stored in a Kubernetes secret:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
```

The secret is base64 encoded. Decode it with:

```bash
echo "secret value" | base64 --decode
```

Use the decoded value as your password to log into the ArgoCD UI.

---

## Application Manifest Files

### 1. Deployment (`deployment.yml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: swiggy-app
  labels:
    app: swiggy-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: swiggy-app
  template:
    metadata:
      labels:
        app: swiggy-app
    spec:
      terminationGracePeriodSeconds: 30
      containers:
      - name: swiggy-app
        image: veeranarni/hotstar:latest
        imagePullPolicy: "Always"
        ports:
        - containerPort: 3000
```

### 2. Service (`service.yml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: swiggy-app
  labels:
    app: swiggy-app
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 3000
  selector:
    app: swiggy-app
```

---

## ArgoCD Application Configuration

To connect ArgoCD to a Git repository, create an `Application` manifest (`application.yaml`):

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-argo-application
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/CloudTechDevOps/Kubernetes.git
    targetRevision: HEAD
    path: day-14-argocd
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp

  syncPolicy:
    syncOptions:
    - CreateNamespace=true

    automated:
      selfHeal: true
      prune: true
```

### Field Explanations

| Field | Description |
|---|---|
| `apiVersion: argoproj.io/v1alpha1` | API version of ArgoCD — check docs for latest version |
| `repoURL` | URL of the Git repository containing your K8s manifests |
| `targetRevision: HEAD` | Always fetches the latest commit |
| `path` | Subdirectory in the repo where the manifests live |
| `destination.server` | Set to `https://kubernetes.default.svc` — the internal Kubernetes API server |
| `destination.namespace` | Namespace to deploy the app into (e.g., `myapp`) |
| `CreateNamespace=true` | Tells ArgoCD to automatically create the namespace if it doesn't exist |
| `selfHeal: true` | Overrides any manual `kubectl` changes to match the Git state |
| `prune: true` | Deletes cluster resources if they are removed from Git |

> **Note:** ArgoCD checks the Git repository for changes every **3 minutes** by default. To trigger syncs immediately on push, configure a **webhook**.

---

## Applying the Configuration

Apply the ArgoCD Application manifest:

```bash
kubectl apply -f application.yaml
```

Once applied, the application will appear in the ArgoCD UI where you can view its sync status, resource tree, manifests, and pod events.

---

## Making Changes via GitOps

To scale your application, simply update the `replicas` field in `deployment.yml` and commit the change to Git. ArgoCD will detect the update and automatically apply it to the cluster.

Verify the new pods:

```bash
kubectl get pods -n myapp
```

Example output after scaling to 4 replicas:

```
NAME                                READY   STATUS    RESTARTS   AGE
myapp-deployment-544dd58bc4-4sntz   1/1     Running   0          13h
myapp-deployment-544dd58bc4-wkf5j   1/1     Running   0          13h
myapp-deployment-544dd58bc4-xt7hb   1/1     Running   0          13h
myapp-deployment-544dd58bc4-zjmn8   1/1     Running   0          13h
```

You can now perform any changes in your Git repository and ArgoCD will take care of the rest.

---

> Refer article: [A Complete Overview of ArgoCD with a Practical Example](https://medium.com/@veerababu.narni232/a-complete-overview-of-argocd-with-a-practical-example-f4a9a8488cf9) 
