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


### launch 1 EC2 : master (kubernetes-cluster)
- launch : us-east-2
- ami: ubuntu
- instance type : t2.medium
- key-pair : devops-live-project-k8s
- network settings: ssh(22), http(80), https(443)
- storage: 15gb

  <img width="1917" height="950" alt="masterMachine(kubernetes-cluster)" src="https://github.com/user-attachments/assets/38c0c7db-b003-4acb-916c-0bbc82a41156" />


### Connect to master (kubernetes-cluster) : via SSH

- set .pem file permission to 400 :
```bash
  chmod 400 devops-live-project-k8s.pem
```

- connect:
```bash
  ssh -i "devops-live-project-k8s.pem" ubuntu@ec2-3-148-144-183.us-east-2.compute.amazonaws.com
```

### Install on master
- Install Docker:
```bash
sudo apt-get update
sudo apt-get install docker.io
sudo usermod -aG docker $USER
docker ps
```
<img width="1472" height="162" alt="docker_installed" src="https://github.com/user-attachments/assets/d6703ede-9994-4ae2-ab64-da668299fe2d" />



- Install Kind:
```bash
mkdir k8s-install
cd k8s-install
vim install_kind.sh
chmod +x install_kind.sh
./install_kind.sh
kind --version
```
<img width="1466" height="328" alt="kind_installed" src="https://github.com/user-attachments/assets/b9146b0c-7d04-4782-9c56-021e488e2dd7" />


### Creating Kind cluster (my-cluster) : via master

- Create a 3-node cluster using Kind: Control-plane, node1, node2
```bash
cd k8s-install
vim config.yml
kind create cluster --config=config.yml --name=my-cluster
```

<img width="1476" height="499" alt="cluster_created_(ControlPlane-Node1-Node2)" src="https://github.com/user-attachments/assets/907ab68f-22a5-4d5b-aa1d-bacbffc1e19a" />


### Access the cluster : via installing kubectl on master
```bash
cd k8s-install
vim install_kubectl.sh
chmod +x install_kubectl.sh
./install_kubectl.sh
```
<img width="1479" height="383" alt="kubectl_installed" src="https://github.com/user-attachments/assets/7a67b31e-4ec2-42db-9ee9-3135fa855d6d" />


### Check nodes : via kubectl
```bash
cd k8s-install
kubectl get nodes
```
<img width="1477" height="184" alt="check_nodes_via_kubectl" src="https://github.com/user-attachments/assets/4cef099f-3386-43d7-96c1-eec56d536cdd" />


### Check docker containers of the cluster nodes:
```bash
cd k8s-install
docker ps
```

<img width="1478" height="331" alt="docker_containers_of_cluster_nodes" src="https://github.com/user-attachments/assets/48e19764-5b00-430f-8d59-86d8f87ad81a" />


### Set up ArgoCD in Cluster:
```bash
cd k8s-install
```
- Create a namespace for Argo CD:                 
```bash
kubectl create namespace argocd
```
- Apply the Argo CD manifest:
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
- check pods inside namespace argocd:
```bash
kubectl get pods -n argocd
```
<img width="1481" height="312" alt="pods_in_namespace_argocd" src="https://github.com/user-attachments/assets/2007aa19-65d5-4367-a4b2-90f11d49f571" />

- Check services in Argo CD namespace:
```bash
kubectl get svc -n argocd
```
- Expose Argo CD server using NodePort:
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```
<img width="1457" height="585" alt="argocd_server_service_type_patching" src="https://github.com/user-attachments/assets/34bf0fc1-00ad-4e61-b6f7-0a545a89cfe8" />


- Forward ports to access Argo CD server:
```bash
kubectl port-forward -n argocd service/argocd-server 8443:443 --address=0.0.0.0 &
```
<img width="1467" height="95" alt="port-forwarding" src="https://github.com/user-attachments/assets/813dfaeb-433c-4b4d-894c-78c11e191054" />


- Open port 8443 in security group of master (kubernetes-cluster):
<img width="1916" height="966" alt="App_port_range_opened" src="https://github.com/user-attachments/assets/6c7b3f00-b359-44f6-bfad-5cd29ff157d1" />


- Open ArgoCd in Browser on port 8443 and log in using below credentials:
Note: username: admin
initial password: 
```bash
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```
<img width="1911" height="960" alt="argocd_logged_in" src="https://github.com/user-attachments/assets/b8c3649e-793a-4b40-8293-10a00ca23b49" />


- After Logged in Argocd:

1. ADD Cluster:
settings -> cluster
default cluster in already added in argocd
<img width="1901" height="978" alt="default_cluster_in_argocd" src="https://github.com/user-attachments/assets/06d3c517-17e2-4a0e-b8f2-22b8ccf4e831" />


3. ADD Application:
Applications -> New App
<img width="1910" height="965" alt="adding_app_in_argocd_1" src="https://github.com/user-attachments/assets/e10f1482-67e9-474b-b158-5800e268fb29" />
<img width="1910" height="965" alt="adding_app_in_argocd_2" src="https://github.com/user-attachments/assets/b48e6219-1fbb-4a35-8325-b34c8784f972" />

<img width="1917" height="969" alt="app_added_in_argocd" src="https://github.com/user-attachments/assets/9e48346a-224b-4161-a84e-9d139827ec3d" />
<img width="1911" height="912" alt="check_pods_running_via_argocd" src="https://github.com/user-attachments/assets/e4b48e80-1bde-4a8b-bc2e-0b10b1bfc635" />


### To see runing pods via kubectl run:
```bash
kubectl get pods
```
(pic) check pods running via kubectl


### Access the App in Browser:
```bash
cd k8s-install
kubectl get svc
```
(pic)

```bash
kubectl port-forward svc/vote 5000:5000 --address=0.0.0.0 &
kubectl port-forward svc/result 5001:5001 --address=0.0.0.0 &
```

- Open 5000-5001 port range in master
(pic)



- Access Vote
(pic)
- Access result
(pic)


- Vote and view Result in App:
1 vote for cat
(pic)
View result
(pic)

- open vote page in incognito
1 vote for dog
(pic)
view result
(pic)


### Creating kubernetes dashboard:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

cd k8s-install

vim dashboard.yml

kubectl apply -f dashboard.yml



Expose Port For the Kubernetes-Dashboard:
kubectl get svc -n kubernetes-dashboard

kubectl port-forward svc/kubernetes-dashboard -n kubernetes-dashboard 8080:443 --address=0.0.0.0 &

Open DashBoard on port 8080 (https)
(pic)

Gnerate Token to sign-in :
Create a token for dashboard access:      kubectl -n kubernetes-dashboard create token admin-user
 
Sign in using the generated token.

Accessing Kubernetes-dashboard on Browser:

(pic)dashboard-signed in



Prometheus and Grafana:

Install Helm package manager:
cd ..
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod +x get_helm.sh
./get_helm.sh

(pic)

Install Kube Prometheus Stack:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
kubectl create namespace monitoring
helm install kind-prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --set prometheus.service.nodePort=30000 --set prometheus.service.type=NodePort --set grafana.service.nodePort=31000 --set grafana.service.type=NodePort --set alertmanager.service.nodePort=32000 --set alertmanager.service.type=NodePort --set prometheus-node-exporter.service.nodePort=32001 --set prometheus-node-exporter.service.type=NodePort

Port-forwarding for Prometheus and grafana:
kubectl get svc -n monitoring
kubectl get namespace

kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &
kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 --address=0.0.0.0 &

Open Ports 9090 for prometheus and 31000 for grafana in master

Note: Grafana username: admin password: prom-operator

Access in Browser
Prometheus
(pic)

Grafana
(pic)


Prometheus Dashboard target health status:
(pic)

CPU usage Monitoring of App : via  Prometheus
promQL : sum (rate (container_cpu_usage_seconds_total{namespace="default"}[1m])) / sum (machine_cpu_cores) * 100
(pic)



InComing Traffic of App: via  Prometheus
sum(rate(container_network_receive_bytes_total{namespace="default"}[5m])) by (pod)
(pic)



User-Management in Grafana:
Giving View role to a new user to Grafana Dashboards:
Creating new user in Grafana:
Administration -> user and access -> users -> new user
(pic)
user created
(pic)

Keep change role of the demo user as Viewer
(pic)

Now any user can view the Grafana dashboard with the credentials of the user created above


Connection establishment in Grafana:

Adding Datasources to Grafana:
Connections -> Datasources
Data Sources**(Prometheus and Alert Manager)** was by default added when we deployed the Prometheus and Grafana Stack using Helm

Build a Dashboard:
Click on build a dashboard for Prometheus DataSource
Then Add Visualization
Select Prometheus DataSource

View CPU Usage in the DashBoard:
choose metrics as container-cpu-cfs-periods-total and label filters as namespace and kube-system
(pic)



Optional: Importing Dashboard Templates of your choice
Importing Kubernetes dashboard Template in Grafana:



Delete kind cluster:
 kind delete cluster --name=my-cluster





