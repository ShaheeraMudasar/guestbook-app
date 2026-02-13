# 🚀 Cloud-Native Guestbook: Multi-Repo GitOps Pipeline

A full-stack Guestbook application (Go & React) deployed to **OpenShift** using a modern **GitOps** architecture. This project demonstrates automated CI/CD patterns, container orchestration, and infrastructure-as-code principles.

## 🏗️ The "Handshake" Workflow

This project utilizes a **separated repository strategy** to decouple application code from environment configuration.

1. **Trigger:** A code push to the `backend/` or `frontend/` directories in the **App Repo** triggers the specific GitHub Action pipeline.
2. **Build & Registry:** GitHub Actions logs into **Quay.io** (using stored Repository Secrets) to build and push an OCI-compliant image tagged with the unique Git SHA.
3. **Manifest Update:** The pipeline clones the **Infra Repo** and uses `sed` to update the image tag in the deployment YAMLs. It then commits and pushes these changes back to the Infra Repo.
4. **GitOps Sync:** **Argo CD** monitors the Infra Repo. When it detects the new image tag, it automatically synchronizes the OpenShift cluster state to match Git.
5. **Deployment:** OpenShift pulls the new image from Quay.io using an `ImagePullSecret` and performs a rolling update of the pods.

---
## 🚀 How to Run the System

### 1. Security & Configuration (Manual Bootstrap)

To keep sensitive credentials secure, **Secrets** and **ConfigMaps** are stored locally and injected directly into the cluster.

* **Inject Local Configuration:**
```bash
oc apply -f ./local-secrets.yaml
oc apply -f ./local-configmaps.yaml

```



> **Note:** These files are git-ignored to ensure passwords and tokens are never exposed in the public repository.

### 2. Deploy Infrastructure & Applications

Once the secrets are in place, you can deploy the entire stack using the manifests in the Infra Repo:

```bash
oc apply -f k8s/

```

*This command applies all Deployments and Services (Postgres, Redis, Backend, and Frontend) simultaneously.*

### 3. Expose the Application

After the pods are running, create the external "door" to the frontend:

```bash
oc expose svc guestbook-frontend-service --name=guestbook-frontend

```

**Access the app at:** `oc get route guestbook-frontend`

---

## 🌐 Networking & Access

In this architecture, the application is exposed to the internet via the **OpenShift Route** layer.

### How the Route is Created

While Deployments and Services are managed via YAML in the Infra Repo, the **Route** (the external URL) can be generated from the Service.

* **Command:** `oc expose svc guestbook-frontend-service --name=guestbook-frontend`
* **Logic:** This command creates a "door" from the outside world to the internal Service. The Route automatically maps to the Service's port (e.g., `8080`) and provides a DNS hostname for the application.

> **Note:** If the Route is ever deleted (e.g., during a "Prune" sync in Argo CD), it can be recreated using the `oc expose` command above to restore public access.

---

## 🛠️ Tech Stack & Security

* **Frontend/Backend:** React.js & Golang (Gin)
* **Data:** PostgreSQL & Redis
* **Registry:** Quay.io
* **Orchestration:** Red Hat OpenShift
* **CD Tool:** Argo CD
* **Security:** ConfigMaps and Secrets are managed **manually** within the cluster namespace. This "Bootstrap" method ensures sensitive credentials (like DB passwords) are never stored in plain text in public repositories. For production environments, this would be upgraded to a solution like **HashiCorp Vault** or **Sealed Secrets**.

---

## Architecture

```txt
Internet
    ↓
[Route] → [Frontend Service] → [Frontend Pod]
                                      ↓
                            [Backend Service] → [Backend Pod(s)]
                                                  ↓         ↓
                                            [Redis]   [PostgreSQL]
```
---

## 🚀 Future Improvements

* Implement **Helm Charts** for better manifest management.
* Add **Liveness and Readiness probes** to ensure zero-downtime deployments.
* Integrate **SonarQube** for automated code quality gates in the CI pipeline.






