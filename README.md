# LRC Backend Deployment Documentation

## Overview

This document outlines the process for deploying a Node.js application on AWS Elastic Kubernetes Service (EKS) using Helm. The deployment includes setting up a LoadBalancer service to expose the application externally.

## Prerequisites

- AWS CLI installed and configured
- kubectl configured to interact with the Kubernetes cluster
- Helm installed
- Docker image pushed to Amazon Elastic Container Registry (ECR)

## Steps

### 1. Create an EKS Cluster

1.1. **Create the Cluster:**
```bash
aws eks create-cluster --name my-cluster --role-arn ARN_OF_EKS_ROLE --resources-vpc-config subnetIds=SUBNET_ID_LIST,securityGroupIds=SECURITY_GROUP_ID_LIST
```

1.2. **Update the kubeconfig:**
```bash
aws eks --region REGION update-kubeconfig --name my-cluster
```

### 2. Prepare the Docker Image

2.1. **Build the Docker Image:**
```bash
docker build -t lrc-backend:latest .
```

2.2. **Tag the Image:**
```bash
docker tag lrc-backend:latest AWS_ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/lrc-backend:latest
```

2.3. **Push the Image to ECR:**
```bash
docker push AWS_ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/lrc-backend:latest
```

### 3. Create a Helm Chart for the Application

3.1. **Initialize Helm Chart:**
```bash
helm create lrc-backend
```

3.2. **Edit `values.yaml` to set the image and service configuration:**

```yaml
replicaCount: 2

image:
  repository: AWS_ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/lrc-backend
  pullPolicy: IfNotPresent
  tag: latest

service:
  type: LoadBalancer
  port: 80
  targetPort: 3000
```

### 4. Deploy the Application using Helm

4.1. **Install the Helm Chart:**
```bash
helm install lrc-backend ./lrc-backend
```

4.2. **Verify the Deployment:**
```bash
kubectl get deployments
```

4.3. **Check the Service and Load Balancer:**
```bash
kubectl get svc
```

### 5. Access the Application

5.1. **Retrieve the Load Balancer URL:**
```bash
kubectl get service lrc-backend -o wide
```

The external IP address or hostname listed under `EXTERNAL-IP` is the URL through which the application can be accessed.

### 6. Monitoring and Management

6.1. **View Pod Logs:**
```bash
kubectl logs -f pod-name
```

6.2. **Scaling the Application:**
```bash
kubectl scale deployment lrc-backend --replicas=NUMBER_OF_REPLICAS
```

6.3. **Updating the Application:**
To update the application, update the Docker image in ECR and then roll out a new deployment using Helm.

## Troubleshooting

- **Load Balancer IP Stuck in Pending:** Check the AWS console for Load Balancer health and ensure the EKS worker nodes have the correct security group settings.
- **Deployment Failures:** Use `kubectl describe pod POD_NAME` to get more information about pod failures.
- **Service Unreachable:** Verify network ACLs, security groups, and that the Load Balancer is correctly configured and pointing to the healthy pods.

## Conclusion

This document covers the deployment of a Node.js application on AWS EKS using Helm, including setting up a LoadBalancer to expose the application. For more detailed monitoring, scaling, or updating the application, refer to Kubernetes and AWS documentation.