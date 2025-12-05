# Flask + MongoDB Kubernetes Deployment
Assignment Submission – Arpit Thakur

## 1. Project Overview

This project demonstrates how to containerize and deploy a Python Flask application connected to a MongoDB database on a Kubernetes cluster using Minikube.

The Flask application exposes two endpoints:

| Endpoint | Purpose |
|---------|---------|
| `/` | Returns a welcome message with current server time |
| `/data` | POST inserts JSON data, GET retrieves stored data |

MongoDB is deployed with authentication enabled and persistent volume storage. The Flask application is deployed as a scalable application using Kubernetes Horizontal Pod Autoscaler (HPA). DNS-based service discovery ensures inter-pod communication without depending on IP addresses.

---

## 2. Repository Structure

.
├── app.py
├── requirements.txt
├── Dockerfile
├── README.md
└── k8s/
├── mongo-secret.yaml
├── mongo-pv-pvc.yaml
├── mongo-statefulset.yaml
├── mongo-service.yaml
├── flask-deployment.yaml
├── flask-service.yaml
└── flask-hpa.yaml

yaml
Copy code

---

## 3. Docker Image Build and Push Instructions

Build Docker image:
```sh
docker build -t kapil0321/flask-mongodb-app:latest .
Login to Docker Hub:

sh
Copy code
docker login
Push image:

sh
Copy code
docker push kapil0321/flask-mongodb-app:latest
4. Kubernetes Deployment Guide
Start Minikube:

sh
Copy code
minikube start --driver=docker
Enable metrics server:

sh
Copy code
minikube addons enable metrics-server
Deploy MongoDB resources:

sh
Copy code
kubectl apply -f k8s/mongo-secret.yaml
kubectl apply -f k8s/mongo-pv-pvc.yaml
kubectl apply -f k8s/mongo-statefulset.yaml
kubectl apply -f k8s/mongo-service.yaml
Deploy Flask application:

sh
Copy code
kubectl apply -f k8s/flask-deployment.yaml
kubectl apply -f k8s/flask-service.yaml
kubectl apply -f k8s/flask-hpa.yaml
Verify deployments:

sh
Copy code
kubectl get pods
kubectl get svc
kubectl get deploy
kubectl get hpa
5. Accessing the Application
Expose the Flask service:

sh
Copy code
minikube service flask-service
You will receive a URL like:

cpp
Copy code
http://127.0.0.1:<port>/
Open this in a browser or call using curl.

6. Testing API
Test root:

sh
Copy code
curl http://127.0.0.1:<port>/
Insert JSON data:

sh
Copy code
curl -X POST -H "Content-Type: application/json" \
-d '{"name":"test"}' http://127.0.0.1:<port>/data
Retrieve stored data:

sh
Copy code
curl http://127.0.0.1:<port>/data
7. DNS Resolution in Kubernetes
Kubernetes provides internal DNS through CoreDNS.

Service DNS format:

pgsql
Copy code
<service-name>.<namespace>.svc.cluster.local
Example in this project:

pgsql
Copy code
mongo.default.svc.cluster.local
The Flask application connects to MongoDB using only:

nginx
Copy code
mongo
This ensures communication remains valid even if MongoDB Pod IP changes.

8. Resource Requests and Limits
Component	CPU Request	CPU Limit	Memory Request	Memory Limit
Flask App	0.2 CPU	0.5 CPU	250Mi	500Mi
MongoDB	0.2 CPU	0.5 CPU	250Mi	500Mi

Requests guarantee resources for stable performance.
Limits prevent containers from consuming excessive cluster resources.

9. Design Choices
Flask is stateless, so it uses a Deployment for scaling.

MongoDB requires persistent identity, so it uses a StatefulSet.

Data persistence is ensured using PV and PVC.

MongoDB Service is ClusterIP for internal-only access.

Flask Service is NodePort to enable access from Minikube.

Secrets are used to securely manage database credentials.

HPA is configured to automatically scale Flask based on CPU usage.

10. Testing Scenarios
Database functionality was verified using POST and GET requests on /data.

Autoscaling load test was performed with continuous curls:

sh
Copy code
while true; do curl http://127.0.0.1:<port>/ > /dev/null; done
Observed behavior:

CPU usage exceeded 70%

Replicas increased from 2 to higher count

After load stopped, replicas returned to minimum of 2

This confirms autoscaling is working correctly.
