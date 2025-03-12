# Kind Cluster Setup and Application Deployment in Ec2 machine in AWS

## Overview
This document provides step-by-step instructions for setting up a Kubernetes cluster using Kind, deploying MySQL and a web application, and ensuring proper connectivity and scaling using Kubernetes resources such as ReplicaSets, Deployments, and Services.

## Prerequisites
Before proceeding, ensure the following tools are installed and properly configured:
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [AWS CLI](https://aws.amazon.com/cli/)

## Step 1: Setting Up the Kind Cluster
Initialize the Kind cluster using the provided configuration file.

```sh
chmod +x ./init_kind.sh
./init_kind.sh

echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc
```

Verify the Kind and `kubectl` versions:
```sh
kind version
kubectl version --client
```

Confirm that the cluster is running:
```sh
kubectl cluster-info
kubectl get endpoints -n default kubernetes
kubectl get nodes
```

## Step 2: Creating Namespaces and Secrets
### Create a Namespace for the Database
```sh
kubectl create namespace db-nmspc
```

### Configure AWS Credentials
Set up AWS credentials for accessing the Elastic Container Registry (ECR):
```sh
export AWS_ACCESS_KEY_ID=<YOUR_AWS_ACCESS_KEY_ID>
export AWS_SECRET_ACCESS_KEY=<YOUR_AWS_SECRET_ACCESS_KEY>
export AWS_SESSION_TOKEN=<YOUR_AWS_SESSION_TOKEN>
```

### Create Docker Registry Secret for ECR
```sh
kubectl create secret docker-registry ecr-registry-key \
  --docker-server=<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region us-east-1) \
  --docker-email=<YOUR_EMAIL> \
  -n db-nmspc
```

## Step 3: Deploying the Database Pod and Service
Apply the database pod manifest and verify its status:
```sh
kubectl apply -f db-pod.yaml
kubectl get pods -n db-nmspc
kubectl logs -n db-nmspc db-pod
```

Deploy the database service:
```sh
kubectl apply -f db-service.yaml
kubectl get svc -n db-nmspc
```

## Step 4: Setting Up the Web Application Namespace
```sh
kubectl create namespace app-nmspc
```

### Create Docker Registry Secret for the Web Application
```sh
kubectl create secret docker-registry ecr-registry-key \
  --docker-server=<AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region us-east-1) \
  --docker-email=<YOUR_EMAIL> \
  -n app-nmspc
```

## Step 5: Deploying the Web Application Pod and Service
Deploy the web application pod:
```sh
kubectl apply -f app-pod.yaml
kubectl get pods -n app-nmspc
```

Access the pod and test connectivity:
```sh
kubectl exec -it app-pod -n app-nmspc -- bash
apt-get install curl
curl http://localhost:8080/
exit
```

Check logs for verification:
```sh
kubectl logs app-pod -n app-nmspc
```

Deploy the web application service:
```sh
kubectl apply -f app-service.yaml
kubectl get svc -n app-nmspc
```

## Step 6: Verifying Web Application Accessibility
Retrieve service endpoints:
```sh
kubectl get endpoints app-service -n app-nmspc
```

Verify accessibility in a browser by navigating to:
```
http://<PUBLIC_IP>:30000/
```

## Step 7: Deploying ReplicaSets for MySQL and Web Application
Deploy MySQL ReplicaSet:
```sh
kubectl apply -f db-replicaset.yaml
kubectl get rs -n db-nmspc
kubectl get pods -l app=db -n db-nmspc
```

Deploy Web Application ReplicaSet:
```sh
kubectl apply -f app-replicaset.yaml
kubectl get rs -n app-nmspc
kubectl get pods -l app=employees -n app-nmspc
```

To delete an existing ReplicaSet (if necessary):
```sh
kubectl delete rs webapp-replicaset -n app-nmspc
```

## Step 8: Deploying Applications Using Deployments
Deploy MySQL using a Deployment:
```sh
kubectl apply -f db-deployment.yaml
kubectl get deployment -n db-nmspc
```

Deploy the web application using a Deployment:
```sh
kubectl apply -f app-deployment.yaml
kubectl get deployment -n app-nmspc
```

## Step 9: Updating Web Application Deployment
Monitor the rollout status of the updated deployment:
```sh
kubectl rollout status deployment/app-deployment -n app-nmspc
```

Once updated, refresh the browser to see changes.

---
### Notes:
- Replace placeholders such as `<AWS_ACCOUNT_ID>`, `<YOUR_EMAIL>`, and `<PUBLIC_IP>` with actual values before execution.
- Use `kubectl logs <pod_name> -n <namespace>` for debugging any deployment issues.
- This guide ensures a structured, reproducible, and scalable Kubernetes deployment process.
