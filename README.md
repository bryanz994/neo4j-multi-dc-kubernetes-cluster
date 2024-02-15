# Neo4j Multi Data Centre Kubernetes Cluster Deployment (AWS)

This repo provides [Helm](https://helm.sh/) charts and [Kubernetes] values files as a reference to deploy Multi-DC Neo4j Graph Data Platform across Kubernetes Clusters in Amazon Web Services (AWS).

## Multi-DC Architecture (FOR TEST ONLY, not for PROD)
<img width="1368" alt="image" src="https://github.com/bryanz994/neo4j-multi-dc-kubernetes-cluster/assets/19281954/ac0d2065-b83f-4387-a4f9-dc7ed6752607">


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

### awscli
To access AWS services with the AWS CLI, you need an AWS account and IAM credentials. When running AWS CLI commands, the AWS CLI needs to have access to those AWS credentials. 
Refer to this guide to install [awscli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

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
2. Create secondary (second and third) EKS cluster
   1. Populate the VPC & Subnet IDs based on the VPC created from the first Kubernetes cluster
   2. Note: This will deploy the new Kubernetes cluster in the existing VPC
3. Create neo4j namespace
4. Deploy Network Load Balancers
5. Retrieve External-IP 
<img width="927" alt="image" src="https://github.com/bryanz994/neo4j-multi-dc-kubernetes-cluster/assets/19281954/4d4e14e2-ae2e-4425-8cd5-d737a6812b5c">

6. Retrieve ingress IP address from ingress DNS
<img width="927" alt="image" src="https://github.com/bryanz994/neo4j-multi-dc-kubernetes-cluster/assets/19281954/234211ef-135d-4fff-9f52-c2b3e13ae3e9">

7. Neo4j Deployment - primary EKS cluster
8. Neo4j Deployment - secondary (second and third) EKS cluster
9. Verify Cluster Formation
<img width="1407" alt="image" src="https://github.com/bryanz994/neo4j-multi-dc-kubernetes-cluster/assets/19281954/bffbde52-a745-4b21-a726-347a048e25ca">


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
aws eks update-kubeconfig --name cluster-name
kubectl config set-context --current --namespace=neo4j
kubectl apply -f lb-1-cluster-name.yaml
kubectl apply -f lb-2-cluster-name.yaml
kubectl apply -f lb-3-cluster-name.yaml
```

5. Create Network Load Balancer in secondary EKS cluster

```
aws eks update-kubeconfig --name cluster-name-2
kubectl config set-context --current --namespace=neo4j
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
