# 🔧 DevSecOps CI-CD Pipeline: High-Level Overview

Here’s a **practical and secure pipeline**, with all the tools you mentioned, structured into the following stages:

### 📌 Pipeline Stages: `Plan → Build → Test → Deploy → Monitor`

---

### ✅ **1. Plan (Infrastructure Setup & Config Management)**

> 🛠 Tools: **Terraform**, **Ansible**, **Git**, **GitHub**

```
[GitHub Repo]
     ⬇️ Code
[Terraform] ─ (Provision EC2 + EBS) ─► [AWS EC2 Instance]
     ⬇️
[Ansible] ─ (Install Jenkins, Docker, K8s, etc.)
```

---

### ✅ **2. Build (CI/CD Code & Containers)**

> 🧱 Tools: **GitHub**, **Jenkins**, **Maven**, **Docker**

```
[Developer Push Code to GitHub]
     ⬇️ Webhook
[Jenkins] ─► [Maven] ─ (Build App)
          ⬇️
      [SonarQube] (Code Quality Scan)
          ⬇️
        [Trivy] (Image Vulnerability Scan)
          ⬇️
      [Docker Build + Push to DockerHub]
```

---

### ✅ **3. Test (Static, Dynamic, Security)**

> 🔍 Tools: **SonarQube**, **Trivy**, **Jenkins (Unit Tests)**

```
[Jenkins] ─► Run Unit Tests
         ├► [SonarQube] ─ (Code Quality)
         └► [Trivy] ─ (Image Scan)
```

---

### ✅ **4. Deploy (App on Kubernetes)**

> 🚀 Tools: **Jenkins**, **Kubernetes**, **Docker**

```
[Jenkins] ─► kubectl apply -f deployment.yaml
              ⬇️
         [Kubernetes (on same EC2)] ─ Deploy App Container
```

---

### ✅ **5. Monitor (Infra & App Health)**

> 📈 Tools: **Prometheus**, **Grafana**

```
[Prometheus] ─► Scrapes Metrics from App & EC2
     ⬇️
[Grafana] ─ (Visual Dashboards)
```

---

## 🌐 EC2 Instance Role Overview

You'll host **all components on a single t2.medium EC2**:

| Component  | Deployed On EC2     | Notes                       |
| ---------- | ------------------- | --------------------------- |
| Jenkins    | ✅ Yes               | Master node for CI/CD       |
| Docker     | ✅ Yes               | Container runtime           |
| Kubernetes | ✅ Yes (Single-node) | K3s or Minikube recommended |
| SonarQube  | ✅ Yes               | DevSecOps static scan       |
| Trivy      | ✅ Yes               | Image scan                  |
| Prometheus | ✅ Yes               | Monitoring                  |
| Grafana    | ✅ Yes               | Dashboards                  |
| GitHub     | ❌ Cloud             | Source repo (GitHub.com)    |

---

## ✅ Does This Work For You?

Please confirm:

1. ✅ Proceed with **Terraform for one-time EC2 provisioning**, or skip it and manually create the instance.
2. ✅ High-level pipeline overview is OK, so we can **dive into step-by-step setup**, starting with **Terraform + Ansible**.
3. Any preference between **Minikube** or **K3s** for single-node Kubernetes?

---

Once you confirm, we’ll start with:

### ➡️ Step 1: **Terraform Script for One EC2 Instance + EBS (optional)**
