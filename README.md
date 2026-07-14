# 🚀 GitOps with ArgoCD & Helm

[![GitHub repo size](https://img.shields.io/github/repo-size/atulupadhyay2004/agrocd)](https://github.com/atulupadhyay2004/agrocd)
[![GitHub stars](https://img.shields.io/github/stars/atulupadhyay2004/agrocd?style=social)](https://github.com/atulupadhyay2004/agrocd/stargazers)

> A complete GitOps workflow demonstrating automated Kubernetes deployments using **ArgoCD** and **Helm** — the modern way to manage cloud-native applications.

---

## 📋 Table of Contents
- [About The Project](#-about-the-project)
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [System Architecture](#-system-architecture)
- [GitOps Workflow](#-gitops-workflow)
- [Helm Chart Structure](#-helm-chart-structure)
- [ArgoCD Sync Policy](#-argocd-sync-policy)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Deploy the Helm Chart](#deploy-the-helm-chart)
  - [Set up ArgoCD](#set-up-argocd)
  - [Create the ArgoCD Application](#create-the-argocd-application)
  - [Trigger Auto-Sync](#trigger-auto-sync)
- [Contributing](#-contributing)
- [License](#-license)
- [Contact](#-contact)

---

## 📖 About The Project

This project demonstrates a **production-ready GitOps workflow** using ArgoCD and Helm. It includes:

- A **Helm chart** for a Node.js application with configurable replicas, image tags, resource limits, and probes
- **ArgoCD** configured to automatically sync the cluster state with the Git repository
- **GitOps principles** where the Git repository is the single source of truth for the desired state

The repository represents the **source of truth** for the application's desired state. Any change to the Helm chart or values triggers an automatic sync, keeping the cluster in perfect alignment with Git.

---

## ✨ Features

- ✅ **GitOps-driven** — Declarative infrastructure as code
- ✅ **Helm chart** — Parameterized and reusable Kubernetes manifests
- ✅ **ArgoCD auto-sync** — Automatic cluster reconciliation with Git
- ✅ **Health checks** — Liveness and readiness probes for application stability
- ✅ **Resource management** — CPU and memory requests/limits defined
- ✅ **Rollback capability** — Revert to any previous state via Git history
- ✅ **Audit trail** — Every change is tracked in Git commits

---

## 🛠 Tech Stack

| Category | Technology |
|----------|------------|
| **GitOps Operator** | ArgoCD |
| **Package Manager** | Helm (v3) |
| **Orchestration** | Kubernetes |
| **Application** | Node.js (atulupadhyay002/eks-helm-demo:1.0) |
| **Registry** | Docker Hub |
| **Version Control** | Git / GitHub |

---

## 🧠 System Architecture

Below is the high-level architecture of the GitOps workflow:
┌─────────────────────────────────────────────────────────────────────────────────┐
│ DEVELOPER WORKSTATION │
│ │
│ ┌─────────┐ git push ┌─────────────────────────────────────────┐ │
│ │ Code │ ─────────────▶ │ GitHub Repository (atulupadhyay2004/ │ │
│ │ Editor │ │ agrocd) │ │
│ └─────────┘ │ ┌─────────────────────────────────┐ │ │
│ │ │ helm-charts/nodejs-app/ │ │ │
│ │ │ ├── Chart.yaml │ │ │
│ │ │ ├── values.yaml │ │ │
│ │ │ └── templates/ │ │ │
│ │ │ ├── deployment.yaml │ │ │
│ │ │ └── service.yaml │ │ │
│ │ └─────────────────────────────────┘ │ │
│ └───────────────────┬─────────────────────┘ │
│ │ │
└───────────────────────────────────────────────────┼────────────────────────────┘
│
▼ (ArgoCD polls Git)
┌─────────────────────────────────────────────────────────────────────────────────┐
│ KUBERNETES CLUSTER │
│ │
│ ┌─────────────────────────────────────────────────────────────────────────┐ │
│ │ ARGOCD (Controller) │ │
│ │ │ │
│ │ 1. Polls GitHub for changes │ │
│ │ 2. Detects drift between Git and cluster │ │
│ │ 3. Applies the desired state │ │
│ └───────────────────────────┬─────────────────────────────────────────────┘ │
│ │ │
│ ▼ (Sync) │
│ ┌─────────────────────────────────────────────────────────────────────────┐ │
│ │ NAMESPACE: default (or custom) │ │
│ │ │ │
│ │ ┌─────────────────────────────────────────────────────────────────┐ │ │
│ │ │ Deployment (nodejs-app) │ │ │
│ │ │ - replicas: {{ .Values.replicaCount }} │ │ │
│ │ │ - image: {{ .Values.image.repository }}:{{ .Values.image.tag }}│ │ │
│ │ │ - resources: requests/limits │ │ │
│ │ │ - livenessProbe: /health │ │ │
│ │ │ - readinessProbe: /health │ │ │
│ │ └───────────────────────────┬─────────────────────────────────────┘ │ │
│ │ │ │ │
│ │ ▼ │ │
│ │ ┌─────────────────────────────────────────────────────────────────┐ │ │
│ │ │ Service (nodejs-app) │ │ │
│ │ │ - type: ClusterIP │ │ │
│ │ │ - port: 80 → targetPort: 3000 │ │ │
│ │ └─────────────────────────────────────────────────────────────────┘ │ │
│ │ │ │
│ └─────────────────────────────────────────────────────────────────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────────────┘

---

## 🔄 GitOps Workflow

This diagram illustrates the end-to-end GitOps flow:

```mermaid
flowchart TD
    A[Developer updates Helm chart or values] --> B[git add, commit, push]
    B --> C[GitHub receives the change]
    C --> D[ArgoCD polls GitHub every 3 minutes]
    D --> E{Detected change?}
    E -->|Yes| F[ArgoCD identifies drift]
    E -->|No| D
    F --> G[ArgoCD applies the new desired state]
    G --> H[Kubernetes reconciles the cluster]
    H --> I[New pods are created/updated]
    I --> J[Liveness & Readiness probes verify health]
    J --> K{Healthy?}
    K -->|Yes| L[Application is live]
    K -->|No| M[Rollback to previous state]
    
    N[Developer reviews ArgoCD UI] --> O[Application is Synced & Healthy]
