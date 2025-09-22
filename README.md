# K8s Kind Voting App

Kubernetes on AWS EC2 with Prometheus & Grafana monitoring, plus Argo CD for deployment.

## Overview

This guide covers the steps to:
- Launching an AWS EC2 instance.
- Installing Docker and Kind to create a Kubernetes cluster.
- Setting up kubectl for cluster management.
- Deploying the Kubernetes Dashboard for cluster insights.
- Installing and configuring Prometheus and Grafana for monitoring and visualization.
- Integrating Argo CD to streamline application deployment and management.

## Architecture

![Architecture diagram](k8s-kind-voting-app.png)

## Observability

![Grafana diagram](grafana.png)
![Prometheus diagram](prometheus.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time


## Hands-On Guide: Kubernetes Monitoring with Prometheus & Grafana

## Project Setup & Implementation: Kubernetes with Prometheus and Grafana

### 1. Launch an EC2 Instance
- Region: **us-east-2**  
- AMI: **Ubuntu**  
- Instance type: **t2.medium**  
- Key-pair: **devops-live-project-k8s**  
- Network settings: **SSH (22), HTTP (80), HTTPS (443)**  
- Storage: **15GB**  

Connect via SSH:  
```bash
chmod 400 devops-live-project-k8s.pem
ssh -i "devops-live-project-k8s.pem" ubuntu@<ec2-public-dns>
2. Install Docker
bash
Copy code
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER
docker ps
3. Install Kind
bash
Copy code
mkdir k8s-install && cd k8s-install
vim install_kind.sh
chmod +x install_kind.sh
./install_kind.sh
kind --version
4. Create Kind Cluster
bash
Copy code
cd k8s-install
vim config.yml
kind create cluster --config=config.yml --name=my-cluster
Check nodes:

bash
Copy code
kubectl get nodes
docker ps
5. Install kubectl
bash
Copy code
cd k8s-install
vim install_kubectl.sh
chmod +x install_kubectl.sh
./install_kubectl.sh
6. Set Up Argo CD
bash
Copy code
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pods -n argocd
kubectl get svc -n argocd
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
kubectl port-forward -n argocd service/argocd-server 8443:443 --address=0.0.0.0 &
Retrieve initial password:

bash
Copy code
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
7. Deploy the Voting App via Argo CD
Login to Argo CD (username: admin, password: retrieved above)

Add cluster (default is already present)

Add application → Deploy Voting App

Check pods via Argo CD or kubectl:

bash
Copy code
kubectl get pods
Forward ports for app access:

bash
Copy code
kubectl port-forward svc/vote 5000:5000 --address=0.0.0.0 &
kubectl port-forward svc/result 5001:5001 --address=0.0.0.0 &
8. Access the App
Open port 5000 → Voting Page

Open port 5001 → Results Page

9. Kubernetes Dashboard
bash
Copy code
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
vim dashboard.yml
kubectl apply -f dashboard.yml
kubectl port-forward svc/kubernetes-dashboard -n kubernetes-dashboard 8080:443 --address=0.0.0.0 &
Generate access token:

bash
Copy code
kubectl -n kubernetes-dashboard create token admin-user
Access via: https://<ec2-public-dns>:8080

10. Install Prometheus & Grafana
Install Helm
bash
Copy code
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod +x get_helm.sh
./get_helm.sh
Install Kube Prometheus Stack
bash
Copy code
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
kubectl create namespace monitoring

helm install kind-prometheus prometheus-community/kube-prometheus-stack \
--namespace monitoring \
--set prometheus.service.nodePort=30000 \
--set prometheus.service.type=NodePort \
--set grafana.service.nodePort=31000 \
--set grafana.service.type=NodePort \
--set alertmanager.service.nodePort=32000 \
--set alertmanager.service.type=NodePort \
--set prometheus-node-exporter.service.nodePort=32001 \
--set prometheus-node-exporter.service.type=NodePort
Port-Forwarding
bash
Copy code
kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &
kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 --address=0.0.0.0 &
Prometheus: http://<ec2-public-dns>:9090

Grafana: http://<ec2-public-dns>:31000

Username: admin

Password: prom-operator

11. Sample Monitoring Dashboards
CPU Usage (PromQL):

promql
Copy code
sum(rate(container_cpu_usage_seconds_total{namespace="default"}[1m])) / sum(machine_cpu_cores) * 100
Incoming Traffic (PromQL):

promql
Copy code
sum(rate(container_network_receive_bytes_total{namespace="default"}[5m])) by (pod)
12. Grafana User Management
Create new user → Assign role as Viewer

Users can now view dashboards with limited permissions

13. Cleanup
bash
Copy code
kind delete cluster --name=my-cluster
markdown
Copy code



