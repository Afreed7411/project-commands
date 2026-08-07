# project-commands
k8s











PREREQUISITES (Windows Side)

•	Windows 10/11

•	WSL 2 enabled
•	Ubuntu installed
•	Docker Desktop installed
o	Enable WSL Integration
 	


To deploy an application on a Kubernetes cluster and monitor its performance using Prometheus and Grafana.
 Ubuntu (WSL)

 
sudo apt update -y
_______________________________________________________________________________________________________________________________________________
 
 
 Install Docker (inside WSL)

 
 
sudo apt install docker.io -y

Start Docker:

sudo service docker start

Give permission:

sudo usermod -aG docker $USER


Close & reopen Ubuntu terminal

Verify:

docker ps



Install kubectl

 
______________________________________________________________________________________________________________________________________________


curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl



chmod +x kubectl


sudo mv kubectl /usr/local/bin/


kubectl is the primary command-line tool for interacting with a Kubernetes cluster


Check:


kubectl version --client
_______________________________________________________________________________________________________________________________________________
 
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube


Start Kubernetes:


minikube start --driver=docker


Verify:


kubectl get nodes



Explain:

Minikube creates a local Kubernetes cluster. Minikube is a lightweight, single-node Kubernetes cluster tool for local development





Helm is the package manager for Kubernetes, simplifying the definition, installation, and upgrading of even complex applications by bundling all necessary Kubernetes resources into single, reusable packages called Helm Charts.



 It acts like a software installer for Kubernetes, letting you manage applications with commands like install, upgrade, and rollback, reducing manual configuration and ensuring consistency across different environments


 
Install Helm (Package Manager)

_______________________________________________________________________________________________________________________________________________
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash


Check:


helm version
_______________________________________________________________________________________________________________________________________________
Deploy Sample Application (nginx)



kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx --type=NodePort --port=80



Check:

kubectl get pods

kubectl get svc
Access app:


minikube service nginx
 
Kubernetes is now running our application inside a Pod.
_______________________________________________________________________________________________________________________________________________
Install Prometheus (Monitoring)


Add Helm repo:


helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

Install Prometheus:


helm install prometheus prometheus-community/prometheus



Check:

kubectl get pods

Port forward Prometheus:

kubectl port-forward svc/prometheus-server 9090:80

Open browser:

http://localhost:9090


Test query:


up
 Explain:

 
Prometheus is scraping Kubernetes metrics automatically.
______________________________________________________________________________________________________________________________________________
Install Grafana (Visualization)

helm repo add grafana https://grafana.github.io/helm-charts


helm repo update

helm install grafana grafana/grafana

Get password:

kubectl get secret grafana -o jsonpath="{.data.admin-password}" | base64 --decode && echo

Port forward:


kubectl port-forward svc/grafana 3000:80
Open:

http://localhost:3000
Login:
•	Username: admin
•	Password: (from above)

_______________________________________________________________________________________________________________________________________________
  Connect Prometheus to Grafana

  
Grafana UI:
1.	 Settings → Data Sources
2.	Add Prometheus
3.	URL:
http://prometheus-server.monitoring.svc.cluster.local


http://prometheus-server:80

4.	Save & Test


______________________________________________________________________________________________________________________________________________
Import Kubernetes Dashboard 


Grafana → Dashboards → Import

Use Dashboard ID:
6417
(or)
3662
 You will see:
•	CPU usage
•	Memory usage
•	Pod count
•	Node health
________________________________________
  Show Live Load (Best Demo)
  
kubectl exec -it deploy/nginx -- /bin/bash

Inside pod:
while true; do wget -q -O- http://localhost; done
________________________________________
Developer Push Code
        ↓
GitHub Repository
        ↓
GitHub Webhook
        ↓
ngrok Public URL
        ↓
Jenkins Pipeline
        ↓
Docker Build
        ↓
Docker Container Deploy
        ↓
Application running on port 8081
_______________________________________________________________________________________________________________________________________________
Here's a simple step-by-step Kubernetes deployment guide with the commands you used, plus the most useful debugging commands.
3-Tier MERN Application Deployment on Kubernetes (Minikube)


Step 1: Start Minikube

minikube start --driver=docker
Check status:
minikube status
________________________________________



Step 2: Build Docker Images

Frontend

cd frontend

docker build -t affu9164/frontend:v1 .

Backend

cd ../backend

docker build -t affu9164/backend:v1 .
_______________________________________________________________________________________________________________________________________________
Step 3: Push Images to Docker Hub

docker push affu9164/frontend:v1



docker push affu9164/backend:v1

MongoDB uses the official mongo image, so no need to build it.
_______________________________________________________________________________________________________________________________________________


Step 4: Go to Kubernetes Folder
cd ../k8s
______________________________________________________________________________________________________________________________________________
Step 5: Create Namespace

kubectl apply -f namespace.yaml


Verify:
kubectl get ns
______________________________________________________________________________________________________________________________________________
Step 6: Create Persistent Volume Claim

kubectl apply -f mongo-pvc.yaml

Check PVC:

kubectl get pvc -n chat-app
_______________________________________________________________________________________________________________________________________________
Step 7: Create Secrets

kubectl apply -f backend-secrets.yaml

Check:

kubectl get secrets -n chat-app
______________________________________________________________________________________________________________________________________________
Step 8: Create MongoDB Deployment

kubectl apply -f mongodb-deployment.yaml

Create MongoDB Service:

kubectl apply -f mongodb-service.yaml

Check:


kubectl get pods -n chat-app
kubectl get svc -n chat-app
_______________________________________________________________________________________________________________________________________________
Step 9: Create Backend

kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml

Check:

kubectl get pods -n chat-app
kubectl get svc -n chat-app
_______________________________________________________________________________________________________________________________________________
Step 10: Create Frontend ConfigMap

kubectl apply -f frontend-configmap.yaml

Verify:

kubectl get configmap -n chat-app
_______________________________________________________________________________________________________________________________________________
Step 11: Create Frontend

kubectl apply -f frontend-deployment.yaml

kubectl apply -f frontend-service.yaml
_______________________________________________________________________________________________________________________________________________

Step 12: Verify Everything

Pods
kubectl get pods -n chat-app

Services

kubectl get svc -n chat-app
Deployments

kubectl get deployment -n chat-app
_______________________________________________________________________________________________________________________________________________
Step 13: Access Application

Port Forward

kubectl port-forward svc/frontend 3000:80 -n chat-app

or
kubectl port-forward svc/frontend-service 3000:80 -n chat-app
Open
http://localhost:3000

Backend

kubectl port-forward svc/backend 5001:5001 -n chat-app

Open

http://localhost:5001
_______________________________________________________________________________________________________________________________________________
Useful Debugging Commands

Check Pods

kubectl get pods -n chat-app


Watch pods live

watch kubectl get pods -n chat-app
_______________________________________________________________________________________________________________________________________________
Check Services
kubectl get svc -n chat-app
_____________________________________________________________________________________________________________________________________________
Check Deployments
kubectl get deployment -n chat-app
___________________________________________________________________________________________________________________________________________
View Logs
Backend
kubectl logs <backend-pod-name> -n chat-app
Previous logs
kubectl logs <backend-pod-name> -n chat-app --previous
Frontend
kubectl logs <frontend-pod-name> -n chat-app
MongoDB
kubectl logs <mongodb-pod-name> -n chat-app
________________________________________
Describe a Pod

kubectl describe pod <pod-name> -n chat-app

Example

_______________________________________________________________________________________________________________________________________________
kubectl describe pod backend-xxxx -n chat-app



Look at the Events section for errors.
_______________________________________________________________________________________________________________________________________________
Check ConfigMaps



kubectl get configmap -n chat-app


Describe ConfigMap



kubectl describe configmap nginx-config -n chat-app
_______________________________________________________________________________________________________________________________________________
Check Secrets


kubectl get secrets -n chat-app
_______________________________________________________________________________________________________________________________________________
Check PVC


kubectl get pvc -n chat-app
_______________________________________________________________________________________________________________________________________________
Check Endpoints


kubectl get endpoints -n chat-app
_______________________________________________________________________________________________________________________________________________
Restart Deployment



Backend


kubectl rollout restart deployment backend -n chat-app


Frontend
kubectl rollout restart deployment frontend -n chat-app
MongoDB
kubectl rollout restart deployment mongodb -n chat-app
_______________________________________________________________________________________________________________________________________________
Delete a Pod
Backend
kubectl delete pod -l app=backend -n chat-app
Frontend
kubectl delete pod -l app=frontend -n chat-app
MongoDB
kubectl delete pod -l app=mongodb -n chat-app
________________________________________
Delete Everything
kubectl delete -f .
Deploy everything again
kubectl apply -f .
________________________________________
Common Errors & Fixes

Backend: CrashLoopBackOff

kubectl logs <backend-pod> -n chat-app --previous


Usually caused by:

•	Wrong MongoDB URI
•	Missing Secret
•	MongoDB Service not created
_______________________________________________________________________________________________________________________________________________
Frontend: ContainerCreating

kubectl describe pod <frontend-pod> -n chat-app

Check the Events section.

Common causes:

•	Missing ConfigMap (nginx-config)
•	Image pull issues
•	Volume mount errors
_______________________________________________________________________________________________________________________________________________

MongoDB Connection Error

getaddrinfo EAI_AGAIN mongodb
Check:

kubectl get svc -n chat-app

If mongodb service is missing:

kubectl apply -f mongodb-service.yaml
________________________________________
Port Forward

kubectl port-forward svc/frontend 3000:80 -n chat-app

Backend

kubectl port-forward svc/backend 5001:5001 -n chat-app

