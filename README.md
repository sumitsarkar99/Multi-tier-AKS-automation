# Multi-tier-AKS-automation
# 🛠️ Multi-Tier AKS Application Deployment Using Azure CI/CD

This repository contains a step-by-step Azure DevOps CI/CD pipeline for deploying a containerized, multi-tier application (Storefront App) to Azure Kubernetes Service (AKS). The application consists of:

- 🛒 **Store Front** – User-facing app
- 📦 **Product Service** – Displays product data
- 🧾 **Order Service** – Processes orders
- 📬 **RabbitMQ** – Message broker

---

##  Part 1: Local Development Reference – Docker Compose (Prep Phase)

Although this repo uses Azure DevOps for CI/CD, the application was originally built and tested using Docker Compose.

### Reference Local Steps (If Needed)

```bash
# Clone the sample repo
git clone https://github.com/Azure-Samples/aks-store-demo.git
cd aks-store-demo

# Start all services locally using Docker Compose
docker compose -f docker-compose-quickstart.yml up -d

# Verify images are created
docker images

# View running containers
docker ps

# Open the app in your browser
http://localhost:8080

## **Part 2: Build & Push Images to Azure Container Registry (ACR)**
-------------------------------------------------------------------
To support automated Kubernetes deployments, we publish our multi-tier app images to [Azure Container Registry (ACR)](https://learn.microsoft.com/en-us/azure/container-registry/). This enables secure and scalable access from Azure Kubernetes Service (AKS).

---

### What We Automate in Azure DevOps

| Manual Step (Azure CLI) | Azure DevOps Pipeline Equivalent |
|--------------------------|----------------------------------|
| `az acr create` | A pipeline stage with Azure CLI task to create ACR (optional for infra-as-code repos) |
| `az acr import` | Azure CLI task in pipeline to import product-service image from GHCR |
| `az acr build` | Azure DevOps `Docker@2` task or `az acr build` via AzureCLI@2 task |
| `az acr repository list` | Azure CLI task with `az acr repository list` (for validation or logs) |

---

### Project Structure

src/
├── order-service/
├── product-service/ (imported from GHCR)
├── store-front/
infra/
└── acr-setup/ (optional Bicep or Terraform ACR creation)
.azure-pipelines/
└── pipeline.yml

### Key Pipeline Tasks - (see yaml code -   )

-- At this stage, all required images are securely available in ACR and ready to be pulled by your AKS cluster during deployment.

## **Part 3: Create and Connect to Azure Kubernetes Service (AKS)**
-----------------------------------------------------------------
This section focuses on provisioning a production-grade AKS cluster using Azure DevOps and configuring it to pull container images from Azure Container Registry (ACR).

---

### ✅ What We Automate in Azure DevOps

| Manual Step (Azure CLI) | Azure DevOps Equivalent |
|--------------------------|--------------------------|
| `az aks create` | Use `AzureCLI@2` task with scripted AKS creation (infra-as-code alternatives: Bicep, Terraform) |
| `az aks install-cli` | Pre-installed on hosted agents, skip |
| `az aks get-credentials` | Not required inside pipeline; use `kubectl` context directly via Azure DevOps |
| `kubectl get nodes` | Validation task in pipeline using `kubectl` |

---

### 📦 Azure DevOps Pipeline Snippet – Create AKS Cluster - (see yaml code -   )

At this point, your cluster is ready, and your ACR-hosted images can be pulled into your AKS workloads using Kubernetes manifests or Helm charts.

## 🗄️ Part 4: (Optional) Enable Azure Container Storage for Stateful Workloads

To support **stateful workloads** or simulate high-performance storage needs (e.g., databases, cache layers, benchmark pods), you can extend your AKS setup with [Azure Container Storage (ACS)](https://learn.microsoft.com/en-us/azure/container-storage/).

This section demonstrates how to:

- Enable Azure Container Storage on AKS
- Create a storage pool using **ephemeral NVMe disk**
- Deploy a pod using a **generic ephemeral volume** with a test workload (FIO)

---

### ⚙️ What We Automate in Azure DevOps

| Manual Step | Azure DevOps Equivalent |
|-------------|--------------------------|
| `az aks update ... --enable-azure-container-storage` | AzureCLI@2 task inside `StorageSetup` stage |
| `kubectl apply -f acstor-pod.yaml` | Kubernetes@1 task for pod deployment |
| `kubectl describe ...` | Optional debug/validate step post-deployment |

---

### 📦 Optional Azure DevOps Stage for Storage Setup  (see yaml code -   )



This is an advanced add-on for users managing stateful apps (e.g., MongoDB, PostgreSQL) or simulating storage-intensive workloads on AKS.

## Part 5: Deploy the Application to AKS

Now that your AKS cluster is running and your container images are available in Azure Container Registry (ACR), it's time to deploy the multi-tier app to the Kubernetes cluster using updated manifest files.

---

### What we can do in Azure DevOps

| Manual Step | Azure DevOps Equivalent |
|-------------|--------------------------|
| Update image paths in manifest file | Pipeline script/replace task or templated YAML |
| `kubectl apply -f aks-store-quickstart.yaml` | `Kubernetes@1` task |
| `kubectl get pods` / `get service` | Post-deployment validation step |
| Watching LoadBalancer IP assignment | Pipeline waits & fetches IP (optional script or status check) |

---

### Kubernetes Manifest File Updates

Update the image names in your `aks-store-quickstart.yaml` file to point to your ACR images:

```yaml
containers:
- name: order-service
  image: <yourACR>.azurecr.io/aks-store-demo/order-service:latest

- name: product-service
  image: <yourACR>.azurecr.io/aks-store-demo/product-service:latest

- name: store-front
  image: <yourACR>.azurecr.io/aks-store-demo/store-front:latest


