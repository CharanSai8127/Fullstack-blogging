# 🚀 GitOps-Driven CI/CD Platform on AWS EKS  
## Secure, Controlled, and Production-Ready Delivery with Argo CD, Vault, and Jenkins  

---

## 🧩 Overview  

This project demonstrates the design and implementation of a **production-grade GitOps-driven CI/CD platform** on AWS EKS that enables secure, automated, and controlled application delivery.

The system extends a basic GitOps setup by introducing **security enforcement, workload isolation, secret management, and cloud-native integrations**, transforming deployments into a **reliable and self-healing system**.

---

## 🎯 Objectives  

- Enforce **secure and validated artifact delivery**  
- Eliminate manual deployment and configuration drift  
- Introduce **Git as the single source of truth**  
- Separate **platform and application workloads**  
- Implement **production-grade secret management**  
- Enable **controlled autoscaling and system stability**  
- Integrate cloud-native services for **observability and auditability**  

---

## 🏗️ Architecture  

The system is structured into layered components to separate **build, control, and runtime behavior**:

---

### 🔹 CI Layer (Build & Validation)

Handles application validation and artifact creation:

- Jenkins (CI pipeline)  
- SonarQube (code quality)  
- OWASP Dependency Check (vulnerability scanning)  
- Trivy (filesystem & image scanning)  
- Docker (image build & push)  

👉 Ensures **only secure and validated artifacts are produced**

---

### 🔹 CD Layer (GitOps Control Plane)

Controls deployment using GitOps:

- Argo CD (continuous reconciliation)  
- App-of-Apps pattern (platform + workloads separation)  

👉 Ensures **cluster state always matches Git**

---

### 🔹 Platform Layer (Hub Nodes)

Provides operational capabilities:

- Argo CD  
- Vault (secret management)  
- cert-manager (TLS)  
- kube-prometheus-stack (monitoring)  
- NGINX Gateway (traffic entry)  
- AWS EBS CSI Driver (storage)  

👉 Defines **how the cluster behaves**

---

### 🔹 Application Layer (Spoke Nodes)

Runs application workloads:

- Backend application  
- MySQL (StatefulSet)  
- Migration Jobs  

👉 Ensures **clean separation from platform components**

---

### 🔹 Cloud Integration Layer (AWS)

Provides infrastructure-level capabilities:

- AWS KMS (encryption)  
- S3 (log storage)  
- CloudWatch (metrics & logs)  
- CloudTrail (audit logs)  

👉 Enables **secure and observable operations**

---

## 🧭 Architecture Diagrams  

### 🔹 CI Pipeline (Jenkins)

![CI Architecture](./docs/ci-architecture.png)

The CI pipeline validates, secures, and builds application artifacts.

**Flow:**
- Code is pushed to GitHub  
- Jenkins triggers pipeline  
- SonarQube enforces code quality  
- OWASP scans dependencies  
- Trivy scans filesystem and image  
- Docker image is built and pushed  

👉 Ensures **secure, high-quality, and deployable artifacts**

---

### 🔹 CD / GitOps Architecture (Argo CD + Hub & Spoke Model)

![CD Architecture](./docs/cd-architecture.png)

The CD layer ensures automated deployment and system consistency.

**Flow:**
- Git acts as the source of truth  
- Argo CD detects changes and syncs cluster state  
- App-of-apps deploys platform and workloads  
- Hub nodes run platform components  
- Spoke nodes run application workloads  
- Vault injects secrets dynamically  
- AWS services provide logging and audit  

👉 Ensures **secure, declarative, and self-healing deployments**

---

> This architecture separates **build (CI), deployment control (CD), and runtime execution**, enabling independent scaling and reliability.  

> It introduces **control and security layers** on top of GitOps, making the system production-ready.

---

## 📁 Repository Structure  

The repository is organized to separate **deployment control, platform components, and workloads**.

---

### 🔹 Argo CD (GitOps Control Layer)

`argocd/`

Defines deployment orchestration:

- `app-of-apps.yaml` → Root application  
- `hub-apps.yaml` → Platform components  
- `app-apps.yaml` → Workloads  

👉 Acts as the **control plane for deployment**

---

### 🔹 Application Manifests  

`applications/`

Defines workloads:

- Backend  
- Database  
- Jobs  

👉 Represents the **application layer**

---

### 🔹 Platform Configuration  

`projects/`

Defines:

- RBAC policies  
- Namespace controls  
- Deployment boundaries  

👉 Enforces **security and isolation**

---

## 🔁 System Flow  

### Flow Explanation:

1. Developer pushes code to GitHub  
2. Jenkins pipeline builds and scans application  
3. Docker image is created and pushed  
4. Deployment manifests updated in Git  
5. Argo CD detects changes  
6. Cluster state is reconciled  
7. Pods are deployed on Spoke nodes  
8. Vault injects secrets dynamically  
9. Application becomes accessible via Gateway  
10. Metrics and logs are collected  

---

## 🎛️ Control Model  

System behavior is enforced through a layered control model:

| Layer | Responsibility |
|------|---------------|
| **Git** | Source of truth for configurations |
| **Jenkins CI** | Code quality and security validation |
| **Argo CD** | Continuous reconciliation and drift correction |
| **Kubernetes** | Runtime enforcement (scheduling, scaling, isolation) |
| **Vault** | Secure secret access |
| **AWS Services** | Encryption, logging, audit |
| **HPA** | Controlled autoscaling |

---

## ⚙️ Runtime Behavior  

---

### 🔹 Pod Failure  
- Kubernetes restarts failed pods  
- Argo CD ensures desired state is maintained  
- Vault re-injects secrets on restart  

👉 System remains **self-healing**

---

### 🔹 Configuration Changes  
- Managed via Git and applied through Argo CD  
- Triggers rolling updates of pods  
- Invalid configurations surface via health checks  

👉 Ensures **controlled and traceable changes**

---

### 🔹 Autoscaling Behavior  

- Scaling is based on sustained metrics  
- Short spikes do not trigger scaling  
- Scale-out occurs gradually  
- Scale-in is delayed to allow in-flight requests to complete  

👉 Ensures **stability and avoids flapping**

---

### 🔹 Deployment Behavior  
- Git changes trigger Argo CD sync  
- Rolling updates ensure gradual deployment  
- Pods are scheduled on Spoke nodes  
- Platform components remain isolated on Hub nodes  

👉 Ensures **safe and isolated deployments**

---

### 🔹 Secret Management Behavior  
- Pods authenticate using ServiceAccount  
- Vault validates identity  
- Secrets injected dynamically at runtime  

👉 No secrets stored in Git or cluster  

---

### 🔹 Drift Handling  
- Manual changes detected by Argo CD  
- System reverts to Git-defined state  

👉 Ensures **Git remains the single source of truth**

---

## 📊 Observability  

The platform integrates monitoring and logging:

- Prometheus → metrics collection  
- Grafana → visualization dashboards  
- CloudWatch → logs  
- CloudTrail → audit logs  

This enables:

- Real-time system visibility  
- Performance monitoring  
- Failure detection and alerting  

---

## ⚖️ Design Trade-offs & Future Enhancements  

- Jenkins introduces operational overhead → GitHub Actions can reduce cost  
- Autoscaling is metric-based → request-based scaling can improve responsiveness  
- Deployment strategy uses rolling updates → canary or blue-green can reduce risk  
- Vault adds complexity → requires operational management  
- Single cluster design → multi-cluster can improve resilience  

---

## 💬 Summary  

This project demonstrates a **production-grade GitOps CI/CD platform** that integrates security, automation, and cloud-native capabilities to deliver applications reliably.

It highlights how **declarative control, security enforcement, workload isolation, and runtime intelligence** can be combined to build a scalable and resilient system.

> The system is designed using a **Solution → Control → Behavior model**, ensuring stability under change and resilience under failure.
