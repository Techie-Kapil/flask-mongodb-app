#Flask + MongoDB Kubernetes Deployment


1. Project Overview

This project demonstrates how to containerize and deploy a Python Flask application connected to a MongoDB database on a Kubernetes cluster using Minikube. The Flask application provides two endpoints:

Endpoint	Description
/	Returns a welcome message with the current server time
/data	Supports POST to insert JSON data and GET to retrieve data stored in MongoDB

MongoDB is deployed with authentication enabled and uses persistent storage. The Flask application is deployed with autoscaling capability based on CPU metrics. Kubernetes DNS ensures inter-pod communication without depending on IP addresses.

2. Repository Structure
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

3. Docker Image Build and Push Instructions

Build the container image:

docker build -t kapil0321/flask-mongodb-app:latest .


Login to Docker Hub:

docker login


Push image:

docker push kapil0321/flask-mongodb-app:latest

4. Kubernetes Deployment Guide

Start Minikube:

minikube start --driver=docker


Enable metrics-server (required for HPA):

minikube addons enable metrics-server


Deploy MongoDB:

kubectl apply -f k8s/mongo-secret.yaml
kubectl apply -f k8s/mongo-pv-pvc.yaml
kubectl apply -f k8s/mongo-statefulset.yaml
kubectl apply -f k8s/mongo-service.yaml


Deploy Flask application:

kubectl apply -f k8s/flask-deployment.yaml
kubectl apply -f k8s/flask-service.yaml
kubectl apply -f k8s/flask-hpa.yaml


Check resources:

kubectl get pods
kubectl get svc
kubectl get deploy
kubectl get hpa

5. Accessing the Application

Expose the NodePort service:

minikube service flask-service


This will provide a URL such as:

http://127.0.0.1:<port>/

6. API Testing

Test GET root endpoint:

curl http://127.0.0.1:<port>/


Insert a record:

curl -X POST -H "Content-Type: application/json" \
-d '{"name":"test"}' http://127.0.0.1:<port>/data


Retrieve stored data:

curl http://127.0.0.1:<port>/data

7. DNS Resolution in Kubernetes

CoreDNS provides automatic service discovery. Each service receives a DNS name in the format:

<service>.<namespace>.svc.cluster.local


Flask connects to MongoDB using only the service name:

mongo


This ensures reliable communication even if pod IPs change after restarts or rescheduling.

8. Resource Requests and Limits

Both Flask and MongoDB have resource constraints applied:

Component	CPU Request	CPU Limit	Memory Request	Memory Limit
Flask App	0.2	0.5	250Mi	500Mi
MongoDB	0.2	0.5	250Mi	500Mi

Requests ensure the scheduler assigns sufficient resources. Limits protect the cluster from performance degradation due to resource overuse.

9. Design Choices

Deployment used for Flask because it is stateless and horizontally scalable.
StatefulSet used for MongoDB because persistent identity and storage are required.
ClusterIP Service used for MongoDB to restrict access within the cluster.
NodePort Service used for Flask to allow external access through Minikube.
Secrets store MongoDB authentication credentials securely.
Horizontal Pod Autoscaler adjusts replica count based on CPU usage.

10. Testing Scenarios for Autoscaling and Database

Database operations were successfully validated using POST and GET requests.
Autoscaling was tested by generating continuous load using repeated curl commands.

Observed behavior:

CPU usage increased beyond 70 percent

HPA scaled Flask deployment from 2 replicas up to additional pods

When the load was stopped, it scaled back to 2 replicas

This confirmed proper autoscaling functionality.
