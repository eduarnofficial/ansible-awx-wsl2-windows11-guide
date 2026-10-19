🚀 Ansible AWX Installation on Windows 11 using WSL2 & MinikubeA complete, production-grade DevOps guide for deploying and running Ansible AWX locally on Windows 11 using WSL2 (Ubuntu 22.04 LTS), Docker Engine, Minikube, and the AWX Operator.📺 Video Tutorials & Architecture OverviewBefore starting the terminal commands, watch these companion video guides to understand the architecture, core concepts, and complete visual step-by-step installation process:🎥 What is Ansible AWX & Tower?🎥 Complete AWX Installation Walkthrough🎬 Watch Architecture Overview🎬 Watch Installation Guide💻 Hardware & System RequirementsComponentMinimum RecommendedOperating SystemWindows 11 (64-bit) with WSL2Linux DistributionUbuntu 22.04 LTSCPU4 or more logical coresRAM16 GB (8 GB minimal)Free Storage Space40 GB+ free disk space🛠️ Step-by-Step AWX Lab Installation GuideStep 1: Update Ubuntu and Install PrerequisitesRun these commands inside your WSL2 Ubuntu terminal to install core dependencies and utilities:sudo apt update
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
Step 2: Verify PrerequisitesVerify system resources, installed tools, and the Docker runtime environment:git --version
curl --version
python3 --version

df -h

docker --version
docker version
docker info
docker run --rm hello-world
Step 3: Create the AWX Lab DirectoryCreate a dedicated workspace folder inside your native Linux user home directory:mkdir -p ~/awx-lab
cd ~/awx-lab
Step 4: Install kubectlConfigure the official Kubernetes repository key and install kubectl (v1.35):sudo apt-get update

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
Step 5: Install MinikubeDownload and install the official stable Minikube binary:curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube

rm minikube-linux-amd64

minikube version
Step 6: Start the Minikube ClusterInitialize Minikube using the Docker driver configured with dedicated CPU, memory, and disk space:minikube start \
  --driver=docker \
  --cpus=4 \
  --memory=6144 \
  --disk-size=30g

minikube kubectl -- get pods -A

minikube status

kubectl get nodes

kubectl get pods -A
Step 7: Configure the AWX OperatorSet up the release version environment variable and create the deployment directory:export AWX_OPERATOR_VERSION=2.7.2

echo "$AWX_OPERATOR_VERSION"

mkdir -p ~/awx-lab/operator
cd ~/awx-lab/operator
Create or edit the Kustomization file:vi kustomization.yaml
Paste the following YAML manifest:apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - github.com/ansible/awx-operator/config/default?ref=2.7.2

images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.7.2

namespace: awx
Inspect and apply the manifest to deploy the operator:cat kustomization.yaml

kubectl apply -k .
Step 8: Check and Troubleshoot the AWX OperatorMonitor operator pods and controller manager deployment status:kubectl get pods -n awx

kubectl get deployments -n awx

kubectl logs \
  -n awx \
  deployment/awx-operator-controller-manager \
  -c awx-manager \
  -f
(Optional) Inspect the operator pod or update the proxy image if image pull failures occur:kubectl describe pod \
  awx-operator-controller-manager-6f4b9d58ff-bz6z9 \
  -n awx

minikube image pull \
  gcr.io/kubebuilder/kube-rbac-proxy:v0.15.0

kubectl -n awx set image \
  deployment/awx-operator-controller-manager \
  kube-rbac-proxy=quay.io/brancz/kube-rbac-proxy:v0.15.0

kubectl rollout status \
  deployment/awx-operator-controller-manager \
  -n awx

kubectl get pods -n awx
Step 9: Create the AWX Instance ConfigurationDefine the Custom Resource definition (CRD) for the main AWX application deployment:mkdir -p ~/awx-lab/instance
cd ~/awx-lab/instance

vi awx-demo.yml
Paste the following manifest:---
apiVersion: awx.ansible.com/v1beta1
kind: AWX

metadata:
  name: awx-demo

spec:
  service_type: NodePort
  nodeport_port: 30080
Inspect and apply the instance manifest:cat awx-demo.yml

kubectl apply -f awx-demo.yml -n awx
Step 10: Monitor AWX InstallationWatch application pods initializing in real-time:kubectl get pods -n awx -w
In another terminal window, check AWX custom resources and operator logs:kubectl get awx -n awx

kubectl describe awx awx-demo -n awx

kubectl logs \
  -n awx \
  deployment/awx-operator-controller-manager \
  -c awx-manager \
  --tail=100
Step 11: Inspect Services and Persistent VolumesVerify Kubernetes endpoints, NodePort services, and dynamic persistent volume claims:kubectl get services -n awx

kubectl get pvc -n awx

kubectl get endpoints awx-demo-service -n awx

kubectl get pods -n awx -o wide

kubectl get svc -n awx -o wide
Step 12: Retrieve the AWX Admin PasswordExtract and decode the administrator password created automatically by the operator:kubectl get secret awx-demo-admin-password \
  -n awx \
  -o jsonpath='{.data.password}' \
  | base64 --decode

echo
🔒 Security Note: Keep this password secure. Never commit credentials into public code repositories.⏳ IMPORTANT: Wait 5 to 7 Minutes for Backend Setup⚠️ CRITICAL BACKEND INITIALIZATION STEP:PLEASE WAIT 5 TO 7 MINUTES before running Step 13!
During this period, the AWX Operator is actively executing backend tasks:Provisioning the PostgreSQL database pod.Running database migrations & schema setup.Generating dynamic secrets and compiling web UI static assets.Accessing the web service before this initialization finishes will result in 502 Bad Gateway or Connection Refused errors.Step 13: Access the AWX Web InterfaceExpose the NodePort service using Minikube to access the web console:minikube service -n awx awx-demo-service --url
Copy the printed service URL (e.g., http://127.0.0.1:XXXXX).Open your preferred Windows browser (Edge, Chrome, or Firefox).Log in with your credentials:Username: adminPassword: (Value retrieved in Step 12)🔗 Related Resources & Documentation🎥 Architecture Overview: What is AWX & Ansible Tower?🎥 Installation Walkthrough: AWX Step-by-Step Installation Video🌐 Official Docs: Ansible AWX Operator Documentation
