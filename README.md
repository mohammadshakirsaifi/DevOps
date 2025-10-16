# DevOps

# Microservices Platform Local Setup

## 1) Overview — Running the Project Locally

**Goal:**  
Reproduce a microservices platform similar to a cloud environment for practicing architecture, DevOps, security operations, and team leadership.

### Components to Run Locally:

- **Kubernetes multi-node cluster:** k3d running inside WSL/Docker  
- **Service mesh / ingress:** Optional (Traefik or NGINX)  
- **Datastores:**  
  - Postgres (Aurora dev equivalent)  
  - Redis  
  - OpenSearch (or Elasticsearch)  
  - Kafka (Strimzi or Bitnami)  
- **Services:**  
  - Backend microservice (Node.js or Python)  
  - Frontend (React)  
  - Worker service (consuming Kafka)  
- **CI/CD:** GitHub Actions or local Jenkins pipeline deploying Helm charts  
- **Infrastructure as Code (IaC):** Terraform to manage cluster resources locally and model AWS infra (using local state or LocalStack)  
- **Observability:** Prometheus + Grafana, EFK/OL stack for logs (Fluentd/Fluent Bit + OpenSearch)  
- **Security hardening:**  
  - RBAC, NetworkPolicies, Pod Security Standards  
  - Image scanning (Trivy)  
  - Secrets management (Sealed Secrets or HashiCorp Vault)  

**Why k3d?**  
Lightweight, fast multi-node Kubernetes clusters using Docker. Enables easy multi-server/agent topology for HA testing.  
[https://k3d.io](https://k3d.io)

---

## 2) High-level Architecture (Textual)

- **Host OS:** Windows 11/10 with WSL2  
- **WSL:** Multiple Ubuntu distros (e.g., UbuntuDev1, UbuntuDev2) for isolation and role separation  
- **Docker:** Docker Desktop (with WSL2 backend) or Docker Engine inside WSL (choose one; Docker Desktop integrates smoothly)  
  [https://docs.docker.com](https://docs.docker.com)  

- **k3d:** Creates a multi-node Kubernetes cluster (1–3 server nodes + 2 agent nodes)  
- **Apps:** Deployed via Helm charts into namespaces with RBAC and NetworkPolicy enabled  
- **CI/CD:** GitHub Actions or Jenkins builds container images, pushes to local k3d registry, and runs Helm upgrades  
- **Observability stack:** Deployed via Helm (prometheus-operator, Grafana, Fluent Bit, OpenSearch)  

---

## 3) Step-by-Step: Prepare Windows & WSL

### A. Enable WSL2 and Install Ubuntu

Open **PowerShell as Administrator** and run:

```powershell
wsl --install
```

Install Ubuntu from Microsoft Store, or via command line:
```powershell
wsl --install -d ubuntu-22.04
```

(For multiple Ubuntu instances, see next section.)
https://learn.microsoft.com/en-us/windows/wsl/install

B. Create Multiple WSL Ubuntu Instances (Optional)

Create isolated distro copies for different roles or teams:

Export your initialized Ubuntu distro (with user setup and essential packages):
```powershell
wsl --export Ubuntu ubuntu-base.tar
```

Import new distro instances:
```powershell
wsl --import UbuntuDev1 C:\wsl\UbuntuDev1 ubuntu-base.tar --version 2
wsl --import UbuntuDev2 C:\wsl\UbuntuDev2 ubuntu-base.tar --version 2
```

List installed distros:
```powershell
wsl -l -v
```

More info:
https://endjin.com/blog/creating-multiple-wsl-instances/

C. Choose Docker Strategy

Two main options:

Docker Desktop with WSL2 integration (Recommended for Windows Desktop):
Follow Docker documentation to enable WSL2 backend.
https://docs.docker.com/docker-for-windows/wsl/

Docker Engine inside WSL (No Docker Desktop):
Install Docker Engine directly on Ubuntu inside WSL.
https://docs.docker.com/engine/install/ubuntu/

If using Docker Desktop:

Enable “Use the WSL 2 based engine”

Enable integration with your Ubuntu distros (Docker Desktop Settings → Resources → WSL Integration)

## 4) Install Tools Inside WSL Ubuntu (e.g., UbuntuDev1)

Open WSL shell:
```powershell
wsl -d UbuntuDev1
```

Run these commands step-by-step:

```bash
# 1. Update package lists and upgrade
sudo apt update && sudo apt upgrade -y

# 2. Install essential tools
sudo apt install -y curl git jq vim apt-transport-https ca-certificates gnupg lsb-release

# 3. Install kubectl
curl -fsSL https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl -o /usr/local/bin/kubectl
sudo chmod +x /usr/local/bin/kubectl
kubectl version --client --output=yaml

# 4. Install Helm
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | sudo bash
helm version

# 5. Install k3d

# Check if k3d exists
ls /usr/local/bin/k3d

# If missing, install via script
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
k3d version

# If still missing, download latest binary manually
curl -s https://api.github.com/repos/k3d-io/k3d/releases/latest \
| grep "browser_download_url.*linux-amd64" \
| cut -d '"' -f 4 \
| xargs curl -L -o k3d
chmod +x k3d
sudo mv k3d /usr/local/bin/
k3d version

# 6. Install k9s (optional, for Kubernetes navigation)
curl -sS https://webi.sh/k9s | sh
source ~/.config/envman/PATH.env
k9s

# 7. Install Docker client (if needed)

# If using Docker Desktop, docker CLI is already available in WSL.
# Otherwise, install Docker Engine inside WSL:

sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Add current user to docker group
sudo usermod -aG docker $USER
newgrp docker

docker --version

```
Additional Resources

k3d documentation

Kubernetes documentation

Helm documentation

Docker documentation

Terraform documentation# Or, if WSL is already installed:
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
