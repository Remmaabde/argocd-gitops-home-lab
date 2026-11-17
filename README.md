# ArgoCD GitOps Home Lab

![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-326CE5?logo=kubernetes&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-EF7B4D?logo=argo&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Containerization-2496ED?logo=docker&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-Version_Control-181717?logo=github&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-Environment-FCC624?logo=linux&logoColor=black)

A hands-on **GitOps and Continuous Delivery (CD) engineering lab** built with Kubernetes, ArgoCD, Docker, and GitHub.

This project demonstrates how to implement a declarative deployment workflow where Git serves as the single source of truth for Kubernetes application configurations.

Instead of manually deploying and updating workloads using `kubectl`, ArgoCD continuously monitors the Git repository and reconciles the Kubernetes cluster to match the desired state defined in Git.

The objective is to understand and implement real-world GitOps workflows, automated synchronization, self-healing deployments, and Kubernetes application lifecycle management.

---

## Architecture

```text
                  DEVELOPER
                      |
                      |
                 Git Commit
                      |
                      v
              +---------------+
              |    GitHub     |
              |  Main Branch  |
              +---------------+
                      |
                      | Repository Monitoring
                      v
              +---------------+
              |    ArgoCD     |
              |               |
              | Repo Server   |
              | Application   |
              | Controller    |
              | API Server    |
              +---------------+
                      |
                      | Reconciliation
                      v
              +---------------+
              |  Kubernetes   |
              |   Kind Lab    |
              |               |
              | Deployment    |
              | Service       |
              | Pods          |
              +---------------+
                      |
                      v
               RUNNING APP
```

### How It Works

1. Application manifests are maintained in a GitHub repository.
2. ArgoCD monitors the configured Git repository and branch.
3. Changes in Git are detected by ArgoCD.
4. ArgoCD compares the desired state in Git with the actual cluster state.
5. Kubernetes resources are synchronized when automated sync is enabled.
6. ArgoCD continuously reconciles configuration drift when self-healing is enabled.

---

## Technology Stack

| Technology | Purpose |
|---|---|
| Docker | Container runtime |
| Kubernetes | Container orchestration |
| Kind | Local Kubernetes cluster |
| ArgoCD | GitOps continuous delivery |
| Git & GitHub | Version control and configuration source of truth |
| kubectl | Kubernetes cluster management |
| YAML | Declarative infrastructure configuration |
| Linux | Development environment |

---

## Repository Structure

```text
argocd-gitops-home-lab/
│
├── argocd/
│   └── application.yaml
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
└── README.md
```

### Directory Overview

- `argocd/` - Contains ArgoCD Application configuration.
- `k8s/` - Contains Kubernetes application deployment and service manifests.
- `application.yaml` - Defines the Git repository, target revision, deployment path, and synchronization policies.
- `deployment.yaml` - Defines the Kubernetes application workload.
- `service.yaml` - Defines how the application is exposed within the cluster.

---

## Prerequisites

Install the following tools before starting:

- [Docker](https://docs.docker.com/get-started/get-docker/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [Git](https://git-scm.com/)

Verify your installations:

```bash
docker --version
kubectl version --client
kind version
git --version
```

---

## Deployment Guide

### 1. Clone the Repository

```bash
git clone https://github.com/Remmaabde/argocd-gitops-home-lab.git

cd argocd-gitops-home-lab
```

### 2. Create a Local Kubernetes Cluster

Create a Kind cluster named `gitops-lab`:

```bash
kind create cluster --name gitops-lab
```

Verify the cluster:

```bash
kubectl cluster-info --context kind-gitops-lab
```

Check the nodes:

```bash
kubectl get nodes
```

### 3. Install ArgoCD

Create the ArgoCD namespace:

```bash
kubectl create namespace argocd
```

Install ArgoCD using its official installation manifest:

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Wait for the ArgoCD components to become ready:

```bash
kubectl get pods -n argocd -w
```

Check the deployments:

```bash
kubectl get deployments -n argocd
```

### 4. Configure the ArgoCD Application

Apply the Application manifest:

```bash
kubectl apply -f argocd/application.yaml
```

Verify that ArgoCD has registered the application:

```bash
kubectl get applications -n argocd
```

**Important:** Ensure that `argocd/application.yaml` contains the correct GitHub repository URL, branch, and Kubernetes manifest path.

### 5. Access the ArgoCD Dashboard

Forward the ArgoCD server service to your local machine:

```bash
kubectl port-forward svc/argocd-server -n argocd 8081:443
```

Open:

```text
https://localhost:8081
```

Retrieve the initial administrator password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

echo
```

Login credentials:

```text
Username: admin
Password: Retrieved from the command above
```

The dashboard allows you to monitor application synchronization, deployment health, Git revisions, and Kubernetes resources.

---

## GitOps Workflows & Practical Experiments

This lab is designed to demonstrate the core principles of GitOps through practical experiments.

### 1. Automated Synchronization

Modify the application configuration in:

```text
k8s/deployment.yaml
```

For example, change the replica count:

```yaml
spec:
  replicas: 3
```

Commit and push the changes:

```bash
git add .
git commit -m "Update application replicas"
git push origin main
```

If automated synchronization is enabled, ArgoCD detects the Git change and applies the desired configuration to Kubernetes.

Verify:

```bash
kubectl get deployments
kubectl get pods
```

### 2. Self-Healing & Drift Detection

GitOps ensures that the actual cluster configuration stays aligned with the desired configuration stored in Git.

Simulate configuration drift by manually deleting a managed deployment:

```bash
kubectl delete deployment <deployment-name> -n <namespace>
```

Observe the ArgoCD dashboard.

When `selfHeal: true` is configured and automated synchronization is enabled, ArgoCD detects the deviation and reconciles the cluster.

Verify:

```bash
kubectl get pods -n <namespace>
```

### 3. Pruning Resources

When `prune: true` is enabled, resources removed from the tracked Git configuration can also be removed from the cluster during synchronization.

This keeps the cluster aligned with the desired state in Git.

### 4. Application Rollbacks

ArgoCD maintains deployment history based on Git revisions.

A rollback can be performed by reverting the relevant Git commit and pushing the change.

```bash
git revert <commit-hash>
git push origin main
```

ArgoCD then reconciles the cluster against the reverted configuration.

---

## Verification & Troubleshooting

### Check Kubernetes Resources

```bash
kubectl get nodes

kubectl get pods -A

kubectl get deployments -A

kubectl get services -A
```

### Check ArgoCD Application Status

```bash
kubectl get applications -n argocd
```

### Inspect ArgoCD Logs

```bash
kubectl logs deployment/argocd-application-controller \
  -n argocd --all-containers=true --tail=100
```

### Common Issues

**1. ArgoCD Cannot Connect to GitHub**

Check:
- Repository URL.
- Branch name.
- Repository visibility.
- Repository credentials if the repository is private.
- Configured manifest path.

**2. Application Stuck in OutOfSync**

Check:

```bash
kubectl get applications -n argocd
kubectl describe application <application-name> -n argocd
```

Compare the desired Git configuration with the actual cluster resources.

**3. Kubernetes Pods Not Starting**

Check:

```bash
kubectl get pods -A

kubectl describe pod <my-pod-name> -n <namespace>

kubectl logs <my -pod-name> -n <namespace>
```

**4. ArgoCD Components Not Ready**

Check:

```bash
kubectl get pods -n argocd

kubectl get events -n argocd --sort-by=.metadata.creationTimestamp
```

---

## Engineering Concepts Demonstrated

Through this project, I explore and implement:

- GitOps architecture and declarative infrastructure management.
- Kubernetes application lifecycle management.
- Continuous Delivery using ArgoCD.
- Git-based deployment automation.
- Desired state versus actual state reconciliation.
- Automated synchronization and pruning.
- Configuration drift detection.
- Self-healing Kubernetes workloads.
- Version-controlled infrastructure.
- Deployment rollback strategies.
- Kubernetes troubleshooting and resource inspection.

---

## Future Improvements

Planned enhancements to evolve this lab into a more production-oriented GitOps environment:

- [ ] Integrate GitHub Actions for Continuous Integration.
- [ ] Implement automated Docker image builds.
- [ ] Integrate a container registry.
- [ ] Implement Helm-based application deployments.
- [ ] Introduce Kustomize for environment-specific configurations.
- [ ] Configure separate development, staging, and production environments.
- [ ] Implement ArgoCD App of Apps or ApplicationSet patterns.
- [ ] Integrate Prometheus and Grafana for monitoring.
- [ ] Implement secure secrets management.
- [ ] Explore Argo Rollouts for progressive delivery.

---

## Cleanup

Delete the local Kind cluster:

```bash
kind delete cluster --name gitops-lab
```

---

## Learning Objective

This project is part of my practical journey as a **DevOps / Site Reliability Engineer (SRE)**, focusing on Kubernetes, GitOps, Continuous Delivery, infrastructure automation, and production-oriented deployment practices.

**The goal is to move beyond manually deploying applications and build systems that continuously maintain their desired state through automation.**
