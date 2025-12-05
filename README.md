# Flask + MongoDB Kubernetes Deployment  
Assignment Submission – Arpit Thakur

## 1. Project Overview

This project demonstrates how to containerize and deploy a Python Flask application connected to a MongoDB database on a Kubernetes cluster using Minikube.

The Flask application exposes two endpoints:

| Endpoint | Purpose |
|---------|---------|
| `/` | Returns a welcome message with current server time |
| `/data` | POST inserts JSON data, GET retrieves stored data |

MongoDB is deployed with authentication enabled and persistent volume storage. Flask is deployed as a scalable application using Kubernetes Horizontal Pod Autoscaler (HPA). DNS-based service discovery ensures inter-pod communication without IP dependency.

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
```bash
docker build -t kapil0321/flask-mongodb-app:latest .
Login to Docker Hub:

bash
Copy code
docker login
Push image:

bash
Copy code
docker push kapil0321/flask-mongodb-app:latest
4. Kubernetes Deployment Guide
Start Minikube:

bash
Copy code
minikube start --driver=docker
Enable metrics server:

bash
Copy code
minikube addons enable metrics-server
Deploy MongoDB:

bash
Copy code
kubectl apply -f k8s/mongo-secret.yaml
kubectl apply -f k8s/mongo-pv-pvc.yaml
kubectl apply -f k8s/mongo-statefulset.yaml
kubectl apply -f k8s/mongo-service.yaml
Deploy Flask Application:

bash
Copy code
kubectl apply -f k8s/flask-deployment.yaml
kubectl apply -f k8s/flask-service.yaml
kubectl apply -f k8s/flask-hpa.yaml
Verify:

bash
Copy code
kubectl get pods
kubectl get svc
kubectl get deploy
kubectl get hpa
5. Accessing the Application
Expose the Flask service:

bash
Copy code
minikube service flask-service
A URL such as the following will appear:

cpp
Copy code
http://127.0.0.1:<port>/
Access in a browser or using curl.

6. Testing API
Root endpoint:

bash
Copy code
curl http://127.0.0.1:<port>/
Insert sample data:

bash
Copy code
curl -X POST -H "Content-Type: application/json" \
-d '{"name":"test"}' http://127.0.0.1:<port>/data
Retrieve data:

bash
Copy code
curl http://127.0.0.1:<port>/data
7. DNS Resolution in Kubernetes
Kubernetes automatically assigns DNS names to services using CoreDNS.

DNS format:

pgsql
Copy code
<service-name>.<namespace>.svc.cluster.local
In this project:

pgsql
Copy code
MongoDB Service DNS: mongo.default.svc.cluster.local
Flask connects to MongoDB using only:

nginx
Copy code
mongo
This ensures reliable communication even when pod IPs change.

8. Resource Requests and Limits
Component	CPU Request	CPU Limit	Memory Request	Memory Limit
Flask App	0.2	0.5	250Mi	500Mi
MongoDB	0.2	0.5	250Mi	500Mi

Requests ensure minimum guaranteed resources.
Limits prevent a container from consuming excessive CPU or memory.

9. Design Choices
Deployment used for Flask as it is stateless and scalable.

StatefulSet used for MongoDB because data persistence and stable pod identity are required.

Persistent Volume Claim ensures MongoDB data is retained across restarts.

ClusterIP Service restricts MongoDB access within cluster only.

NodePort Service exposes Flask externally through Minikube.

Secrets store database credentials securely.

HPA used for automatic scaling based on CPU usage.

10. Testing Scenarios
Database Communication:

Verified by sending POST requests to /data and retrieving data with GET requests.

Autoscaling Test:

Continuous curl requests were executed to increase CPU load.

HPA scaled replicas when CPU exceeded 70 percent threshold.

After stopping load, HPA scaled replicas back down to two.

Observed behavior confirms proper autoscaling and database integration.
