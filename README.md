🚀 Install Ansible AWX on Windows 11 Using WSL2, Ubuntu & Minikube
A step-by-step guide to deploying Ansible AWX locally on Windows 11 using WSL2, Ubuntu 22.04 LTS, Docker, Minikube, Kubernetes, and the AWX Operator.

This repository provides the installation commands, Kubernetes manifests, verification steps, and troubleshooting guidance needed to build a local AWX automation lab.

Important: This repository is intended for learning, development, and lab environments. It is not a production-ready deployment guide.
📚 Table of Contents
Overview
Architecture
Video Tutorials
System Requirements
Prerequisites
Installation Guide
1. Prepare Ubuntu
2. Verify Docker
3. Create the Lab Directory
4. Install kubectl
5. Install Minikube
6. Start the Kubernetes Cluster
7. Deploy the AWX Operator
8. Verify the Operator
9. Create the AWX Instance
10. Monitor Installation
11. Retrieve the Admin Password
12. Access the AWX Web Interface
Troubleshooting
Useful Commands
Project Structure
Security Notes
Learning Resources
Contributing
License
🎯 Overview
Ansible AWX is an open-source web-based platform for managing, scheduling, and running Ansible automation jobs.

It provides a centralized interface for:

Managing inventories and hosts.
Creating and executing job templates.
Managing credentials and access controls.
Scheduling automation jobs.
Tracking job execution and logs.
Integrating automation into operational workflows.
This lab deploys AWX on a local Kubernetes cluster powered by Minikube.

🏗️ Architecture
The lab uses the following components:


Component	Purpose
Windows 11	Host operating system
WSL2	Linux environment on Windows
Ubuntu 22.04 LTS	Command-line and development environment
Docker	Container runtime used by Minikube
Minikube	Local Kubernetes cluster
kubectl	Kubernetes command-line tool
AWX Operator	Manages the AWX application lifecycle
AWX Web	Provides the AWX web interface and API
AWX Task	Executes automation jobs
PostgreSQL	Stores AWX application data
Deployment flow

Windows 11
    |
    v
WSL2 - Ubuntu 22.04
    |
    v
Docker Engine
    |
    v
Minikube - Kubernetes
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
🎬 Video Tutorials
Use these videos alongside the written instructions.

Video 1: What Is Ansible AWX / Ansible Tower?
Understand AWX architecture, automation management, inventories, credentials, and job templates.
Video link: Add your YouTube URL here
Video 2: Install Ansible AWX on Windows 11
Follow the hands-on installation using WSL2, Ubuntu, Docker, Minikube, and the AWX Operator.
Video link: Add your YouTube URL here
Replace the placeholder links with your actual video URLs before publishing.
💻 System Requirements

Component	Recommended configuration
Operating system	Windows 11 64-bit
Linux environment	Ubuntu 22.04 LTS on WSL2
Virtualization	Enabled in BIOS/UEFI
CPU	4 or more logical processors
RAM	16 GB recommended; 8 GB minimum for a constrained lab
Free disk space	40 GB or more
Container runtime	Docker Engine accessible from WSL2
Internet connection	Required to download images and packages
Actual resource requirements depend on the workload, image sizes, and other applications running on the host.

🔧 Prerequisites
Before beginning, ensure that:

WSL2 is installed and working.
Ubuntu 22.04 LTS is installed.
Docker is running and accessible from Ubuntu.
Windows virtualization features are enabled.
You have a working internet connection.
If Docker is installed on Windows through Docker Desktop, enable WSL integration for your Ubuntu distribution.

Official references:

Install WSL
Docker Desktop with WSL
Minikube documentation
AWX Operator repository
🛠️ Installation Guide
Run the following commands inside your Ubuntu WSL2 terminal, unless otherwise stated.

1. Prepare Ubuntu
Update Ubuntu and install the required utilities.


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
Verify the tools:


git --version
curl --version
python3 --version
2. Verify Docker
Minikube will use Docker as its driver.


docker --version
docker version
docker info
docker run --rm hello-world
All commands should complete successfully before proceeding.

If Docker is unavailable, configure Docker Engine or Docker Desktop WSL integration first.

3. Create the Lab Directory
Keep the lab files inside your Linux home directory for better WSL filesystem performance.


mkdir -p ~/awx-lab
cd ~/awx-lab
4. Install kubectl
Configure the official Kubernetes package repository.

The commands below use the Kubernetes v1.35 package repository. This selects the repository channel; it does not guarantee that every installed component will be version v1.35.


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
Verify the installation:


kubectl version --client
Official documentation: Install kubectl on Linux

5. Install Minikube
Download the Linux AMD64 binary.

These commands assume an AMD64/x86-64 Ubuntu environment. Check your architecture with uname -m if needed.

curl -LO \
  https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube

rm minikube-linux-amd64

minikube version
Official documentation: Install Minikube

6. Start the Kubernetes Cluster
Start Minikube with the Docker driver and allocate resources for the lab.


minikube start \
  --driver=docker \
  --cpus=4 \
  --memory=6144 \
  --disk-size=30g
This configuration allocates four CPUs, approximately 6 GiB of memory, and a 30 GiB disk to Minikube. It does not reserve all of your Windows host resources exclusively for Kubernetes.

Verify the cluster:


minikube status
kubectl get nodes
kubectl get pods -A
The Minikube node should eventually show Ready.

7. Deploy the AWX Operator
For this example, the AWX Operator version is pinned to 2.7.2 so that the manifest is reproducible.

Create the operator directory:


export AWX_OPERATOR_VERSION=2.7.2

mkdir -p ~/awx-lab/operator
cd ~/awx-lab/operator
Create the kustomization.yaml file:


nano kustomization.yaml
Add the following configuration:


apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - github.com/ansible/awx-operator/config/default?ref=2.7.2

images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.7.2

namespace: awx
Save the file, then deploy the operator:


kubectl apply -k .
Check the resources:


kubectl get deployments -n awx
kubectl get pods -n awx
The operator deployment creates the awx namespace as part of this configuration.

Official repository: Ansible AWX Operator

8. Verify the Operator
Wait for the operator deployment to become ready:


kubectl rollout status \
  deployment/awx-operator-controller-manager \
  -n awx \
  --timeout=5m
Follow its logs:


kubectl logs \
  -n awx \
  deployment/awx-operator-controller-manager \
  -c awx-manager \
  -f
Press Ctrl+C to stop following the logs.

If the operator pod fails, inspect its details:


kubectl get pods -n awx -o wide
kubectl describe pods -n awx
Troubleshooting an unavailable proxy image
Some operator installations may encounter problems pulling the kube-rbac-proxy image.

Inspect the pod events first:


kubectl describe pod \
  -n awx \
  -l control-plane=controller-manager
If the configured image is unavailable, verify an alternative image and tag before updating the deployment. Do not assume that every image hosted at an alternative registry is interchangeable.

For example, the following override was used in this lab:


kubectl -n awx set image \
  deployment/awx-operator-controller-manager \
  kube-rbac-proxy=quay.io/brancz/kube-rbac-proxy:v0.15.0
Verify the rollout:


kubectl rollout status \
  deployment/awx-operator-controller-manager \
  -n awx

kubectl get pods -n awx
Use this override only if the alternative image is available and compatible with your operator configuration.

9. Create the AWX Instance
Create a directory for the AWX custom resource:


mkdir -p ~/awx-lab/instance
cd ~/awx-lab/instance
Create the manifest:


nano awx-demo.yml
Add:


---
apiVersion: awx.ansible.com/v1beta1
kind: AWX

metadata:
  name: awx-demo
  namespace: awx

spec:
  service_type: NodePort
  nodeport_port: 30080
Apply the configuration:


kubectl apply -f awx-demo.yml
The AWX Operator will create and manage the associated application resources, including the web, task, database, and service resources.

10. Monitor the AWX Installation
Watch the pods:


kubectl get pods -n awx -w
Wait for the AWX web and task pods to finish initializing.

Check the custom resource:


kubectl get awx -n awx
kubectl describe awx awx-demo -n awx
Review the operator logs:


kubectl logs \
  -n awx \
  deployment/awx-operator-controller-manager \
  -c awx-manager \
  --tail=100
Check services and persistent volumes:


kubectl get services -n awx
kubectl get pvc -n awx
kubectl get endpoints awx-demo-service -n awx
kubectl get pods -n awx -o wide
Expected pod status
Depending on the version and configuration, the main application pods should eventually become ready.


Resource	Expected state
PostgreSQL pod	Running
AWX task pod	All containers ready
AWX web pod	All containers ready
AWX operator pod	All containers ready
The exact number of containers may differ between versions.

Initialization note: The first AWX deployment can take several minutes. Wait for the AWX custom resource and application pods to become ready rather than relying on a fixed waiting period. If the web interface remains on “AWX is currently upgrading,” inspect the resource status and logs.
11. Retrieve the AWX Admin Password
The AWX Operator creates an admin password secret for the instance.

Retrieve it with:


kubectl get secret awx-demo-admin-password \
  -n awx \
  -o jsonpath='{.data.password}' \
  | base64 --decode

echo
Keep the password private. Do not commit it to GitHub, include it in screenshots, or paste it into public issue reports.

The default AWX administrator username is typically:


admin
12. Access the AWX Web Interface
First, verify that the web pod and service are ready:


kubectl get pods -n awx
kubectl get svc awx-demo-service -n awx
kubectl get endpoints awx-demo-service -n awx
Start the Minikube service helper:


minikube service -n awx awx-demo-service --url
Open the URL printed by Minikube in your Windows browser.

Log in using:


Field	Value
Username	admin
Password	The password retrieved in Step 11
Keep the terminal running if Minikube provides a temporary forwarding process. If the connection fails, check the pod readiness and service endpoints before retrying.

🧰 Troubleshooting
1. ImagePullBackOff or ErrImagePull
Inspect the affected pod:


kubectl get pods -n awx
kubectl describe pod <pod-name> -n awx
Check the Events section for the exact image name and pull error.

Common causes include:

Incorrect image name or tag.
Registry connectivity problems.
Authentication failures.
Unsupported image architecture.
2. SVC_UNREACHABLE
If Minikube reports that no running pod is available for the AWX service, inspect the endpoints:


kubectl get endpoints awx-demo-service -n awx
An empty endpoint list generally indicates that no ready pods match the service selector.

Check pod status and events:


kubectl get pods -n awx -o wide
kubectl describe awx awx-demo -n awx
3. AWX remains on “AWX is currently upgrading”
Inspect the AWX resource and web logs:


kubectl get awx awx-demo -n awx

kubectl logs deployment/awx-demo-web \
  -n awx \
  --all-containers=true \
  --tail=100
Also check the task pod if required:


kubectl logs deployment/awx-demo-task \
  -n awx \
  --all-containers=true \
  --tail=100
Look for database migration errors, failed initialization tasks, and application exceptions.

4. AWX pods remain in ContainerCreating
Inspect the affected pod:


kubectl describe pod <pod-name> -n awx
Check for image downloads, volume mounts, storage issues, or resource constraints.

5. Minikube is not running
Check the cluster:


minikube status
kubectl get nodes
If the cluster is stopped, start it again:


minikube start --driver=docker
6. Docker is unavailable in WSL2
Verify Docker connectivity:


docker version
docker info
If these commands fail, verify that Docker Engine or Docker Desktop is running and that WSL integration is enabled where applicable.

📋 Useful Commands

Purpose	Command
Check Minikube	minikube status
List Kubernetes nodes	kubectl get nodes
List AWX pods	kubectl get pods -n awx
List AWX services	kubectl get svc -n awx
Inspect AWX status	kubectl get awx awx-demo -n awx
Inspect persistent volumes	kubectl get pvc -n awx
Follow operator logs	kubectl logs -n awx deployment/awx-operator-controller-manager -c awx-manager -f
Open the AWX service	minikube service -n awx awx-demo-service --url
📁 Project Structure
The lab directory should look like this:


awx-lab/
├── operator/
│   └── kustomization.yaml
└── instance/
    └── awx-demo.yml
File descriptions
operator/kustomization.yaml — Defines the AWX Operator version and Kubernetes resources.
instance/awx-demo.yml — Defines the AWX custom resource and service configuration.
🔐 Security Notes
Do not commit passwords, tokens, private keys, or Kubernetes secrets.
Keep the AWX administrator password in a secure password manager.
Do not expose the AWX NodePort to untrusted networks.
Use appropriate authentication, TLS, backups, and access controls for shared environments.
Keep Kubernetes, Minikube, the AWX Operator, and AWX versions compatible.
Review upstream release notes before upgrading pinned versions.
Production deployments require additional design and security controls. A local Minikube cluster is intended for learning and testing, not high availability or production workloads.

📖 Learning Resources
Ansible AWX
AWX Operator
AWX Operator Documentation
Ansible Documentation
Kubernetes Documentation
Minikube Documentation
Install WSL
Docker Documentation
🤝 Contributing
Contributions, corrections, and suggestions are welcome.

Fork this repository.
Create a feature branch.
Make your changes and test the commands.
Submit a pull request describing your improvements.
If you need GitHub links or additional commands, please leave a comment below.

📄 License
Add a LICENSE file to this repository before specifying a license here. If you intend to publish the project as open source, choose a license appropriate for your intended use.

⭐ If this guide helps you deploy AWX successfully, consider starring the repository and sharing it with other automation learners.
