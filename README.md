
# 🚀 Install Ansible AWX on Windows 11 Using WSL2, Ubuntu 22.04 & Minikube

A step-by-step guide to installing **Ansible AWX locally on Windows 11** using WSL2, Ubuntu 22.04 LTS, Docker, Minikube, Kubernetes, and the AWX Operator.

> **Note:** This repository is intended for learning, development, and lab environments. It is not a production deployment guide.

---

## 📚 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Video Tutorials](#-video-tutorials)
- [System Requirements](#-system-requirements)
- [Installation Guide](#-installation-guide)
- [Troubleshooting](#-troubleshooting)
- [Useful Commands](#-useful-commands)
- [Project Structure](#-project-structure)
- [Security Notes](#-security-notes)
- [Learning Resources](#-learning-resources)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🎯 Overview

Ansible AWX is an open-source web-based platform for managing and running Ansible automation jobs.

Features include:

- Centralized automation management
- Inventory and credential management
- Job templates and scheduling
- Automation execution logs
- Role-based access control
- API-based integrations

This project demonstrates how to deploy AWX on a local Kubernetes cluster using Minikube.

## 🏗️ Architecture

| Component | Purpose |
|---|---|
| Windows 11 | Host operating system |
| WSL2 | Linux environment |
| Ubuntu 22.04 LTS | Installation environment |
| Docker | Container runtime |
| Minikube | Local Kubernetes cluster |
| kubectl | Kubernetes command-line tool |
| AWX Operator | Manages the AWX deployment |
| AWX Web | Web interface and API |
| AWX Task | Executes automation jobs |
| PostgreSQL | Application database |

### Architecture Diagram

```text
Windows 11
    |
    v
WSL2 - Ubuntu 22.04
    |
    v
Docker
    |
    v
Minikube / Kubernetes
    |
    +-- AWX Operator
    |
    +-- AWX Web
    |
    +-- AWX Task
    |
    +-- PostgreSQL
    |
    v
AWX Web Interface
```

---

## 🎬 Video Tutorials

### Video 1: Introduction to Ansible AWX

Learn about AWX, Ansible Tower, inventories, credentials, and automation job templates.

## 📺 Watch the Installation Tutorial

<p align="center">
  <a href="https://www.youtube.com/watch?v=AU14jrXl6Oc">
    <img
      src="https://img.youtube.com/vi/AU14jrXl6Oc/maxresdefault.jpg"
      alt="Watch the Ansible AWX installation tutorial"
      width="800"
    />
  </a>
</p>

<p align="center">
  <strong>▶ Click the thumbnail to watch the full tutorial on YouTube.</strong>
</p>

### Video 2: What is AWX?

<p align="center">
  <a href="https://www.youtube.com/watch?v=jr5qLyWAQJg">
    <img
      src="https://img.youtube.com/vi/jr5qLyWAQJg/maxresdefault.jpg"
      alt="Watch the Ansible AWX installation tutorial"
      width="800"
    />
  </a>
</p>

<p align="center">
  <strong>▶ Click the thumbnail to watch the full tutorial on YouTube.</strong>
</p>
---

## 💻 System Requirements

| Component | Recommended Configuration |
|---|---|
| Operating System | Windows 11 64-bit |
| Linux Distribution | Ubuntu 22.04 LTS on WSL2 |
| Virtualization | Enabled in BIOS/UEFI |
| CPU | 4 or more logical processors |
| RAM | 16 GB recommended; 8 GB minimum for a constrained lab |
| Free Disk Space | 40 GB or more |
| Container Runtime | Docker |
| Internet | Required for downloading packages and images |

Actual requirements depend on your workload and host configuration.

---

# 🛠️ Installation Guide

Run the following commands inside your Ubuntu WSL2 terminal unless otherwise specified.

## Step 1: Update Ubuntu and Install Prerequisites

```bash
sudo apt update
sudo apt upgrade -y

sudo apt install -y \
  curl \
  wget \
  git \
  nano \
  ca-certificates \
  gnupg \
  apt-transport-https \
  conntrack \
  socat \
  iptables \
  python3 \
  python3-pip
```

Verify the installed tools:

```bash
git --version
curl --version
python3 --version
```

## Step 2: Verify Docker

Ensure Docker is running and accessible from Ubuntu.

```bash
docker --version
docker version
docker info
docker run --rm hello-world
```

If Docker commands fail, verify your Docker Engine installation or enable Docker Desktop WSL integration.

## Step 3: Create the Lab Directory

```bash
mkdir -p ~/awx-lab
cd ~/awx-lab
```

## Step 4: Install kubectl

Configure the official Kubernetes package repository.

The following example uses the Kubernetes v1.35 repository channel.

```bash
sudo mkdir -p /etc/apt/keyrings

curl -fsSL \
  https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key \
  | sudo gpg --dearmor --yes \
      -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubectl

kubectl version --client
```

Documentation: [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

## Step 5: Install Minikube

These commands assume an AMD64/x86-64 Linux environment.

```bash
curl -LO \
  https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube

rm minikube-linux-amd64

minikube version
```

Documentation: [Install Minikube](https://minikube.sigs.k8s.io/docs/start/)

## Step 6: Start the Minikube Cluster

```bash
minikube start \
  --driver=docker \
  --cpus=4 \
  --memory=6144 \
  --disk-size=30g
```

Verify the cluster:

```bash
minikube status
kubectl get nodes
kubectl get pods -A
```

The Minikube node should eventually report `Ready`.

## Step 7: Deploy the AWX Operator

Set the example operator version:

```bash
export AWX_OPERATOR_VERSION=2.7.2

mkdir -p ~/awx-lab/operator
cd ~/awx-lab/operator
```

Create the configuration file:

```bash
nano kustomization.yaml
```

Add the following content to `kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - github.com/ansible/awx-operator/config/default?ref=2.7.2

images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.7.2

namespace: awx
```

Apply the operator configuration:

```bash
kubectl apply -k .
```

Verify the deployment:

```bash
kubectl get deployments -n awx
kubectl get pods -n awx
```

Documentation: [AWX Operator GitHub Repository](https://github.com/ansible/awx-operator)

## Step 8: Verify the AWX Operator

Wait for the operator deployment to become ready:

```bash
kubectl rollout status \
  deployment/awx-operator-controller-manager \
  -n awx \
  --timeout=5m
```

View the operator logs:

```bash
kubectl logs \
  -n awx \
  deployment/awx-operator-controller-manager \
  -c awx-manager \
  -f
```

Press `Ctrl+C` to stop following the logs.

Inspect pod status and events if necessary:

```bash
kubectl get pods -n awx -o wide
kubectl describe pods -n awx
```

### Optional: Troubleshoot the Proxy Image

If the `kube-rbac-proxy` image cannot be pulled, inspect the pod events first.

```bash
kubectl describe pod \
  -n awx \
  -l control-plane=controller-manager
```

If the configured image is unavailable, verify an alternative image and tag before updating the deployment.

The following alternative was used in this lab:

```bash
kubectl -n awx set image \
  deployment/awx-operator-controller-manager \
  kube-rbac-proxy=quay.io/brancz/kube-rbac-proxy:v0.15.0
```

Verify the rollout:

```bash
kubectl rollout status \
  deployment/awx-operator-controller-manager \
  -n awx

kubectl get pods -n awx
```

Only use an alternative image after confirming its availability and compatibility.

## Step 9: Create the AWX Instance

Create the instance directory:

```bash
mkdir -p ~/awx-lab/instance
cd ~/awx-lab/instance
```

Create the manifest:

```bash
nano awx-demo.yml
```

Add the following YAML:

```yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX

metadata:
  name: awx-demo
  namespace: awx

spec:
  service_type: NodePort
  nodeport_port: 30080
```

Apply the manifest:

```bash
kubectl apply -f awx-demo.yml
```

The AWX Operator will create and manage the application resources.

## Step 10: Monitor AWX Installation

Watch the pods:

```bash
kubectl get pods -n awx -w
```

Check the AWX resource:

```bash
kubectl get awx -n awx
kubectl describe awx awx-demo -n awx
```

Check the operator logs:

```bash
kubectl logs \
  -n awx \
  deployment/awx-operator-controller-manager \
  -c awx-manager \
  --tail=100
```

Inspect services, storage, and endpoints:

```bash
kubectl get services -n awx
kubectl get pvc -n awx
kubectl get endpoints awx-demo-service -n awx
kubectl get pods -n awx -o wide
```

### Expected Status

| Resource | Expected State |
|---|---|
| PostgreSQL pod | `Running` |
| AWX task pod | All containers ready |
| AWX web pod | All containers ready |
| AWX operator pod | All containers ready |

The number of containers may vary by version.

> The first deployment can take several minutes. Wait for the AWX resource and application pods to become ready. If the interface remains on “AWX is currently upgrading,” inspect the AWX resource and application logs.

## Step 11: Retrieve the AWX Administrator Password

```bash
kubectl get secret awx-demo-admin-password \
  -n awx \
  -o jsonpath='{.data.password}' \
  | base64 --decode

echo
```

The default administrator username is generally:

```text
admin
```

**Security:** Do not commit the password to GitHub or expose it in screenshots, logs, or public issues.

## Step 12: Access the AWX Web Interface

Check that the service has ready endpoints:

```bash
kubectl get pods -n awx
kubectl get svc awx-demo-service -n awx
kubectl get endpoints awx-demo-service -n awx
```

Get the service URL:

```bash
minikube service -n awx awx-demo-service --url
```

Open the URL printed by Minikube in your Windows browser.

Log in with:

| Field | Value |
|---|---|
| Username | `admin` |
| Password | The password retrieved in Step 11 |

Keep the terminal running if Minikube uses a temporary forwarding process.

---

## 🔍 Troubleshooting

### ImagePullBackOff or ErrImagePull

```bash
kubectl get pods -n awx
kubectl describe pod <pod-name> -n awx
```

Inspect the `Events` section for the exact image name and error.

Common causes include invalid tags, registry connectivity, missing credentials, and incompatible image architecture.

### Minikube Reports SVC_UNREACHABLE

```bash
kubectl get endpoints awx-demo-service -n awx
kubectl get pods -n awx -o wide
```

If there are no endpoints, the service has no matching ready pods.

### AWX Remains on the Upgrade Page

```bash
kubectl get awx awx-demo -n awx

kubectl logs deployment/awx-demo-web \
  -n awx \
  --all-containers=true \
  --tail=100
```

Check the task pod logs if necessary:

```bash
kubectl logs deployment/awx-demo-task \
  -n awx \
  --all-containers=true \
  --tail=100
```

### Pods Remain in ContainerCreating

```bash
kubectl describe pod <pod-name> -n awx
```

Look for image download problems, volume mounts, storage issues, and resource constraints.

### Minikube Is Not Running

```bash
minikube status
kubectl get nodes
```

If the cluster is stopped:

```bash
minikube start --driver=docker
```

---

## 📋 Useful Commands

| Purpose | Command |
|---|---|
| Check Minikube | `minikube status` |
| List Kubernetes nodes | `kubectl get nodes` |
| List AWX pods | `kubectl get pods -n awx` |
| List AWX services | `kubectl get svc -n awx` |
| Check AWX status | `kubectl get awx awx-demo -n awx` |
| Check persistent volumes | `kubectl get pvc -n awx` |
| View operator logs | `kubectl logs -n awx deployment/awx-operator-controller-manager -c awx-manager --tail=100` |
| Open AWX | `minikube service -n awx awx-demo-service --url` |

---

## 📁 Project Structure

```text
awx-lab/
├── operator/
│   └── kustomization.yaml
└── instance/
    └── awx-demo.yml
```

| File | Description |
|---|---|
| `operator/kustomization.yaml` | Defines the AWX Operator resources and version |
| `instance/awx-demo.yml` | Defines the AWX instance and service configuration |

---

## 🔐 Security Notes

- Never commit passwords, API tokens, private keys, or Kubernetes secrets.
- Store administrator credentials securely.
- Do not expose the AWX NodePort to untrusted networks.
- Use TLS, backups, authentication, and appropriate access controls for shared environments.
- Review compatibility and release notes before upgrading Kubernetes or AWX components.

A local Minikube environment is intended for learning and testing, not high availability or production workloads.

---

## 📖 Learning Resources

- [Ansible AWX](https://github.com/ansible/awx)
- [AWX Operator](https://github.com/ansible/awx-operator)
- [AWX Operator Documentation](https://ansible.readthedocs.io/projects/awx-operator/en/latest/)
- [Ansible Documentation](https://docs.ansible.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/)
- [Install WSL](https://learn.microsoft.com/windows/wsl/install)
- [Docker Documentation](https://docs.docker.com/)

---

## 🤝 Contributing

Contributions, corrections, and suggestions are welcome.

1. Fork this repository.
2. Create a feature branch.
3. Make your changes and test the commands.
4. Submit a pull request describing your improvements.

If you need GitHub links or additional commands, please leave a comment below.

---

## 📄 License

Add a `LICENSE` file to this repository before specifying a license.

---

⭐ If this guide helps you deploy AWX successfully, consider starring the repository and sharing it with other automation learners.
