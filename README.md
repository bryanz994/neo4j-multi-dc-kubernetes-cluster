# Neo4j Multi Data Centre Kubernetes Cluster Deployment (AWS)

This repo provides [Helm](https://helm.sh/) charts and [Kubernetes] values files as a reference to deploy Multi-DC Neo4j Graph Data Platform across Kubernetes Clusters in Amazon Web Services (AWS).

## **Folder structure**

All the templates in this repo follow a similar folder structure.

```
./
./create_eks_cluster              <-- Folder contains Kubernetes values files to create a primary EKS cluster on AWS and to create a secondary EKS cluster with the existing VPC (primary)
./create_network_load_balancer    <-- Folder contains Kubernetes values files to create network load balancers on AWS
./deploy_neo4j                    <-- Folder contains Helm charts to deploy Neo4j on EKS
```

<br>

## **Prerequisites**

### eksctl

You will need to have AWS API credentials configured. What works for AWS CLI or any other tools (kops, Terraform, etc.) should be sufficient. 
You can use ~/.aws/credentials file or environment variables. For more information read [AWS documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-environment.html).

You will also need [AWS IAM Authenticator](https://github.com/kubernetes-sigs/aws-iam-authenticator) for Kubernetes command (either aws-iam-authenticator or aws eks get-token (available in version 1.16.156 or greater of AWS CLI) in your PATH.

The IAM account used for EKS cluster creation should have these minimal access levels.

AWS Service	Access Level
CloudFormation	Full Access
EC2	Full: Tagging Limited: List, Read, Write
EC2 Auto Scaling	Limited: List, Write
EKS	Full Access
IAM	Limited: List, Read, Write, Permissions Management
Systems Manager	Limited: List, Read

<br>

## **Steps**

1. Create primary EKS cluster
2. Create secondary EKS cluster
   1. Populate the VPC & Subnet IDs based on the VPC created from the first Kubernetes cluster
   2. Note: This will deploy the new Kubernetes cluster in the existing VPC
3. Create neo4j namespace
4. Deploy Network Load Balancers
5. Retrieve ingress IP address from ingress DNS
6. Neo4j Deployment - primary EKS cluster
7. Neo4j Deployment - secondary EKS cluster

<br>

### Deployment steps

1. Create primary and secondary EKS clusters

```
eksctl create cluster -f create-cluster.yaml
eksctl create cluster -f create-cluster-2.yaml
```

2. Create neo4j namespace in both EKS clusters and switch to the neo4j namespace

```
kubectl create namespace neo4j
kubectl config set-context --current --namespace=neo4j
```

3. To switch between different cluster's environment

```
aws eks update-kubeconfig --name cluster-name
aws eks update-kubeconfig --name cluster-name-2
```

4. Create Network Load Balancer in primary EKS cluster

```
kubectl apply -f lb-1-cluster-name.yaml
kubectl apply -f lb-2-cluster-name.yaml
kubectl apply -f lb-3-cluster-name.yaml
```

5. Create Network Load Balancer in secondary EKS cluster

```
kubectl apply -f lb-1-cluster-name-2.yaml
```

6. Deploy neo4j in primary EKS cluster

```
aws eks update-kubeconfig --name cluster-name
kubectl config set-context --current --namespace=neo4j
helm install server-1 neo4j/neo4j -f server-1.yaml
helm install server-2 neo4j/neo4j -f server-2.yaml 
helm install server-3 neo4j/neo4j -f server-3.yaml 
```

7. Deploy neo4j in secondary EKS cluster

```
aws eks update-kubeconfig --name cluster-name-2
kubectl config set-context --current --namespace=neo4j
helm install server-4 neo4j/neo4j -f server-4.yaml
```
<br>
