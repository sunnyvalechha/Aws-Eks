Launch 1 Ec2 Instance

Create 1 Iam User and assign policy of Adminstrator Access and Generate Access Keys for CLI Under Security Credentials.

Install AWS CLI

Practical:-
      
     apt install unzip -y
	    
     curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
     unzip awscliv2.zip
     sudo ./aws/install
     
To Validate the Aws CLI Installation

     aws --version

     aws s3 ls

To Connect User with Command Line Interface

    aws configure

![image](https://github.com/sunnyvalechha/Aws-Eks-Cluster-Setup/assets/59471885/a1932aac-32f8-4598-8948-560b108f2abd)

Install eksctl

    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/local/bin

    eksctl version

To get helping commands 

    eksctl

To Install kubectl

    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

To validate kubectl installation

    echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

To Download kubectl

    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

To Install EKS cluster 

    eksctl create cluster --name sunny-eks-cluster --region us-east-1 --node-type t2.medium --nodes-min 2 --nodes-max 3

Install without node group
    
    eksctl create cluster --name sunny-eks-cluster --region=us-east-1 --zones=us-east-1a,us-east-1b --without-nodegroup

Note: To validate the EKS cluster installation > go to CloudFormation and check for Events

Check how many cluster available?

    eksctl get cluster

If having multiple cluster

    aws eks update-kubeconfig --name <cluster-name> --region us-east-1



# Elastic Kubernetes Services Full Course

**GitHub Repos:**

https://github.com/stacksimplify/aws-eks-kubernetes-masterclass

https://github.com/stacksimplify/docker-fundamentals

https://github.com/stacksimplify/kubernetes-fundamentals

**Aws EKS Cluster - CLI's**

3 Types of CLI's

1. AWS CLI - We can control multiple AWS services from the command line and automate them through scripts.
2. kubectl - We can control kubernetes clusters and objects using kubectl
3. eksctl - Used for creating & deleting clusters on Aws EKS.
            We can even create, autoscale and delete node groups.
            We can even create fargate profiles using eksctl

**AWS EKS - Core Objects**

EKS Cluster has four major parts:

1. EKS Control Plane: Contains Kubernetes master components like etcd, kube-apiserver, kube-controller. It is managed service by AWS.
2. Worker Nodes & Node Groups: Group of EC2 instance where we run our application workloads.
3. Fargate Profiles (Serverless): Instead of Ec2 instance, we run our application workloads on Serverless Fargate profiles.
4. VPC: With Vpc we follow secure networking standards which will allow us to run production workloads on EKS.











# EKS Concepts 

**EKS Storage**

1. EBS CSI Driver: The Amazon Elastic Block Store (Amazon EBS) Container Storage Interface (CSI) driver manages the lifecycle of Amazon EBS volumes as storage for the Kubernetes Volumes that you create. The Amazon EBS CSI driver makes Amazon EBS volumes for these types of Kubernetes volumes: generic ephemeral volumes and persistent volumes.
   
2. EFS CSI Driver: Amazon Elastic File System (Amazon EFS) provides serverless, fully elastic file storage so that you can share file data without provisioning or managing storage capacity and performance. The Amazon EFS Container Storage Interface (CSI) driver provides a CSI interface that allows Kubernetes clusters running on AWS to manage the lifecycle of Amazon EFS file systems.
  
3. Amazon FSx for Lustre CSI driver: The FSx for Lustre Container Storage Interface (CSI) driver provides a CSI interface that allows Amazon EKS clusters to manage the lifecycle of FSx for Lustre file systems.


# Kubernetes Fundamentals

Pod - A Pod is a single instance of an Application. A Pod is the smallest object that you can create in Kubernetes.

Replicasets- A ReplicaSet will maintain a stable set of replica Pods running at any given time. In short, it is often used to gaurantee the availability of a specified number of identical Pods.

Deployments - A Deployment runs multiple replicas of your application and automatically replaces any instances that fail or become un-responsibe. Rollout & rollback changes to applications. Deployments are well suited for stateless applications.

Service - A service is an abstraction for Pods, providing a stable so called virtual IP address. In simple terms service sits Infront of a Pod and acts as a load balancer.




