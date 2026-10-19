🚀 How to Install Ansible AWX on Windows 11 with WSL2, Ubuntu 22.04 & MinikubeA step-by-step production-grade lab setup guide for deploying Ansible AWX locally on Windows 11 using WSL2 (Ubuntu 22.04 LTS), Docker, Minikube, and the AWX Operator.📽️ Video Tutorials & Architecture OverviewLearn the underlying architecture and follow the step-by-step setup visually with these tutorials:🎬 Video 1: What is Ansible AWX / Ansible Tower? (Architecture Overview) — Learn why AWX is essential for enterprise automation management.🎬 Video 2: Hands-on Installation Walkthrough on Windows 11 — Step-by-step video guide covering WSL2, Minikube, and AWX deployment.📊 System RequirementsComponentRecommended SpecsHost OSWindows 11 64-bitLinux DistroUbuntu 22.04 LTS on WSL2VirtualizationEnabled in BIOS / UEFICPU Cores4 or more logical processorsRAM16 GB recommended (8 GB minimum)Free Storage40 GB+ SSD space🛠️ Step-by-Step Installation GuideStep 1: Update Ubuntu and Install PrerequisitesRun these commands inside your WSL2 Ubuntu terminal:sudo apt update
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
Step 2: Verify Prerequisites & Docker EngineEnsure git, curl, python, and Docker are operating properly:git --version
curl --version
python3 --version

df -h

docker --version
docker version
docker info
docker run --rm hello-world
Step 3: Create the AWX Lab DirectorySet up a dedicated working directory on the Linux filesystem:mkdir -p ~/awx-lab
cd ~/awx-lab
Step 4: Install kubectl (v1.35)Configure the official Kubernetes apt repository and install kubectl:sudo apt-get update

sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key \
  | sudo gpg --dearmor --yes \
    -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubectl

kubectl version --client
Step 5: Install MinikubeDownload and install the official Linux AMD64 Minikube binary:curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube

rm minikube-linux-amd64

minikube version
Step 6: Start the Minikube ClusterLaunch Minikube with the Docker driver using optimal system resources:minikube start \
  --driver=docker \
  --cpus=4 \
  --memory=6144 \
  --disk-size=30g

minikube kubectl -- get pods -A

minikube status

kubectl get nodes

kubectl get pods -A
Step 7: Configure the AWX OperatorCreate the operator deployment directory and kustomization.yaml manifest:export AWX_OPERATOR_VERSION=2.7.2

echo "$AWX_OPERATOR_VERSION"

mkdir -p ~/awx-lab/operator
cd ~/awx-lab/operator
Create kustomization.yaml:apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - github.com/ansible/awx-operator/config/default?ref=2.7.2

images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.7.2

namespace: awx
Apply the deployment:cat kustomization.yaml

kubectl apply -k .
Step 8: Check and Troubleshoot the AWX OperatorVerify the operator pod status and follow logs:kubectl get pods -n awx

kubectl get deployments -n awx

kubectl logs \
  -n awx \
  deployment/awx-operator-controller-manager \
  -c awx-manager \
  -f
Inspect operator details if needed:kubectl describe pod \
  awx-operator-controller-manager-6f4b9d58ff-bz6z9 \
  -n awx
(Optional) If you hit image registry timeouts, pull and replace the proxy image:minikube image pull \
  gcr.io/kubebuilder/kube-rbac-proxy:v0.15.0

kubectl -n awx set image \
  deployment/awx-operator-controller-manager \
  kube-rbac-proxy=quay.io/brancz/kube-rbac-proxy:v0.15.0

kubectl rollout status \
  deployment/awx-operator-controller-manager \
  -n awx

kubectl get pods -n awx
Step 9: Create the AWX Instance ConfigurationSet up the custom resource directory and create awx-demo.yml:mkdir -p ~/awx-lab/instance
cd ~/awx-lab/instance
Create awx-demo.yml:---
apiVersion: awx.ansible.com/v1beta1
kind: AWX

metadata:
  name: awx-demo

spec:
  service_type: NodePort
  nodeport_port: 30080
Inspect and apply the instance manifest:cat awx-demo.yml

kubectl apply -f awx-demo.yml -n awx
Step 10: Monitor AWX InstallationWatch application pods spin up:kubectl get pods -n awx -w
In another terminal tab, inspect the AWX custom resource and logs:kubectl get awx -n awx

kubectl describe awx awx-demo -n awx

kubectl logs \
  -n awx \
  deployment/awx-operator-controller-manager \
  -c awx-manager \
  --tail=100
Step 11: Inspect Services and Persistent VolumesVerify that storage PVCs, services, and endpoints are bound:kubectl get services -n awx

kubectl get pvc -n awx

kubectl get endpoints awx-demo-service -n awx

kubectl get pods -n awx -o wide

kubectl get svc -n awx -o wide
Step 12: Retrieve the AWX Admin PasswordExtract and base64-decode the administrator password:kubectl get secret awx-demo-admin-password \
  -n awx \
  -o jsonpath='{.data.password}' \
  | base64 --decode

echo
🔐 Note: Keep this secret safe and do not expose it publicly.⏳ MANDATORY BACKEND CONFIGURATION WAIT (5–7 MINUTES)⚠️ CRITICAL STEP — PLEASE READ CAREFULLY:Do not launch Step 13 immediately! Please wait 5 to 7 minutes after retrieving your password.Why? The AWX Operator requires this time to complete backend migrations in PostgreSQL, create database tables, run initial automation seeds, and compile static web UI assets. Attempting to open the browser early will result in connection timeouts or HTTP 502 Bad Gateway errors.Step 13: Access the AWX Web InterfaceOnce all pods show Running and 5–7 minutes have passed, launch the Minikube service tunnel:minikube service -n awx awx-demo-service --url
Copy the printed URL into your Windows browser (e.g., Chrome, Edge, or Firefox).Log in using:Username: adminPassword: (Retrieved in Step 12)🔗 Learning Resources & Links📺 Video Tutorial 1: What is AWX / Ansible Tower?📺 Video Tutorial 2: Ansible AWX Windows 11 Installation Walkthrough📖 Official Documentation: Ansible AWX Operator Docs
