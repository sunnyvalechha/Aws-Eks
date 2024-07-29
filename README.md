**GitHub Repo:**

https://github.com/stacksimplify/aws-eks-kubernetes-masterclass

* Launch 1 Ec2 Instance (t2.micro)
* Create one Iam User
* Assign policy of Adminstrator Access
* Generate Access Keys & Secret Access Keys
* Install AWS CLI

Practical:-
      
     apt install unzip -y
	    
     curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
     unzip awscliv2.zip
     sudo ./aws/install
     
To Validate the Aws CLI Installation

     aws --version

To Connect User with Command Line Interface

    aws configure

    aws s3 ls		# To Validate the access

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

Associate IAM OIDC provider to EKS cluster, to use IAM roles with Service account this step is necessary when going forward.

    eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster sunny-eks-cluster --approve

If having multiple cluster

    aws eks update-kubeconfig --name <cluster-name> --region us-east-1

Add nodes in the cluster

# Create Public Node Group   
	eksctl create nodegroup --cluster=sunny-eks-cluster \
                       			--region=us-east-1 \
                       			--name=sunny-eks-cluster-ng-public1 \
                       			--node-type=t2.medium \
                       			--nodes=2 \
                       			--nodes-min=1 \
                       			--nodes-max=2 \
                       			--node-volume-size=10 \
                       			--ssh-access \
                       			--ssh-public-key=sunny-aws-key \
                       			--managed \
                       			--asg-access \
                       			--external-dns-access \
                       			--full-ecr-access \
                       			--appmesh-access \
                       			--alb-ingress-access 

Verify cluster creattion

	eksctl get nodegroup --cluster sunny-eks-cluster

 	kubectl get nodes -o wide

  	kubectl config view --minify

Allow all traffic to remote security group

![image](https://github.com/user-attachments/assets/220a3a13-7d66-43dc-ad07-18df1b0c7801)

Delete Cluster

	eksctl get nodegroup --cluster sunny-eks-cluster  # Get nodegroup name

	eksctl delete nodegroup --cluster=sunny-eks-cluster --name=sunny-eks-cluster-ng-public1

 	eksctl delete cluster sunny-eks-cluster

 	
# Aws EKS Cluster - CLI's

3 Types of CLI's

1. AWS CLI - We can control multiple AWS services from the command line and automate them through scripts.
2. kubectl - We can control kubernetes clusters and objects using kubectl
3. eksctl - Used for creating & deleting clusters on Aws EKS.
            We can even create, autoscale and delete node groups.
            We can even create fargate profiles using eksctl

# AWS EKS - Core Objects

EKS Cluster has four major parts:

1. EKS Control Plane: Contains Kubernetes master components like etcd, kube-apiserver, kube-controller. It is managed service by AWS.
2. Worker Nodes & Node Groups: Group of EC2 instance where we run our application workloads.
3. Fargate Profiles (Serverless): Instead of Ec2 instance, we run our application workloads on Serverless Fargate profiles.
4. VPC: With Vpc we follow secure networking standards which will allow us to run production workloads on EKS.

# EKS Storage

1. EBS CSI Driver: The Amazon Elastic Block Store (Amazon EBS) Container Storage Interface (CSI) driver manages the lifecycle of Amazon EBS volumes as storage for the Kubernetes Volumes that you create. The Amazon EBS CSI driver makes Amazon EBS volumes for these types of Kubernetes volumes: generic ephemeral volumes and persistent volumes.
   
2. EFS CSI Driver: Amazon Elastic File System (Amazon EFS) provides serverless, fully elastic file storage so that you can share file data without provisioning or managing storage capacity and performance. The Amazon EFS Container Storage Interface (CSI) driver provides a CSI interface that allows Kubernetes clusters running on AWS to manage the lifecycle of Amazon EFS file systems.
  
3. Amazon FSx for Lustre CSI driver: The FSx for Lustre Container Storage Interface (CSI) driver provides a CSI interface that allows Amazon EKS clusters to manage the lifecycle of FSx for Lustre file systems.


# Kubernetes Fundamentals

Pod - A Pod is a single instance of an Application. A Pod is the smallest object that you can create in Kubernetes.

Replicasets- A ReplicaSet will maintain a stable set of replica Pods running at any given time. In short, it is often used to gaurantee the availability of a specified number of identical Pods.

Deployments - A Deployment runs multiple replicas of your application and automatically replaces any instances that fail or become un-responsibe. Rollout & rollback changes to applications. Deployments are well suited for stateless applications.

Service - A service is an abstraction for Pods, providing a stable so called virtual IP address. In simple terms service sits Infront of a Pod and acts as a load balancer.

# Service

We can expose an application running on a Pod using 3 types:
1. Cluster IP - Default service, can access the Pod IP only within the cluster.
2. NodePort - Expose the Port of the application and access the application outside the cluster via Internet (Port Range - 30,000 to 32,767)
3. Load Balancer - Specially for Cloud Providers like AWs, GCP

       kubectl run my-nginx --image=nginx
       kubectl get pods -o wide
       kubectl describe pod my-nginx
       kubectl get services

       kubectl expose pod nginx --type=ClusterIP --port=80 --name=my-nginx-svc		# Exposed with Cluster IP, cannot access outside the cluster

       curl 10.109.215.88 
   	
       kubectl expose pod my-nginx --type=NodePort --port=80 --name=np-service		# Expose with NodePort, can access on Internet with <PublicIP>:<exposed-port>

Troubleshooting Pod

    kubectl exec -it nginx -- /bin/bash			# Go Inside the Pod

    kubectl logs nginx					# Get Pod logs
    kubectl logs -f  nginx				# Running logs
    kubectl exec -ti nginx ls				# Run command outside the pod
    kubectl exec -ti nginx env
    kubectl exec -ti nginx cat /usr/share/nginx/html/index.html
    kubectl get pods nginx -o yaml			# Get yaml file of Pod 
    kubectl get pods nginx -o yaml > nginx.yaml		# Re-direct content of yaml to another file
    kubectl get svc my-nginx-svc -o yaml		# Get yaml file for service 
    

**Replicasets**

* Replicaset is helps us to maintain a stable set of replica pods running at any given time.
* If application crashes (any pod dies) replicaset will re-create the pod immediately to ensure the configured number of pods running at any given time.

We can achive following things with the help of replica-sets:

1. High Availability or Reliability
2. Scaling
3. Load Balancing
4. Labels & Selectors

Note: We don't have impative command (ad-hoc) to create a replica set. Must have to write yaml

vim replica-1.yaml

	apiVersion: apps/v1
	kind: ReplicaSet
	metadata:
	name: frontend
	labels:
		app: guestbook
		tier: frontend
	spec:
	# modify replicas according to your case
	replicas: 3
	selector:
		matchLabels:
		tier: frontend
	template:
		metadata:
		labels:
			tier: frontend
		spec:
		containers:
		- name: php-redis
			image: us-docker.pkg.dev/google-samples/containers/gke/gb-frontend:v5


	kubectl apply -f replica-1.yaml

	kubectl get replicaset / rs

**Expose replicaset as Service**

Expose rs with Service NodePort to access the application externally (from Internet)

	kubectl expose rs frontend --type=NodePort --port=80 --target-port=8080 --name=frontend


# Deployments

1. Create a Deployment to rollout a Replicaset.
2. Updating the Deployment.
3. Rolling Back a Deployment.
4. Scaling a Deployment.
5. Pausing and Resuming a Deployment.
6. Deployment Status
7. Clean up Policy
8. Canary Deployments

Practical:

kubectl create deployment <Deployment-name> --image=<Container-name>

	kubectl create deployment nginx --image=nginx:1.14.2		# Imparative Way

 	kubectl scale --replicas=20 deployment nginx			# Imparative Way

	apiVersion: apps/v1						# Declarative Way
	kind: Deployment
	metadata:
	name: nginx-deployment
	labels:
		app: nginx
	spec:
	replicas: 3
	selector:
		matchLabels:
		app: nginx
	template:
		metadata:
		labels:
			app: nginx
		spec:
		containers:
		- name: nginx
			image: nginx:1.14.2
			ports:
			- containerPort: 80

**Expose Deployment as a Service**

kubectl expose deployment <Deployment-Name> --type=NodePort --port=80 --target-port=80 --name=<Service-Name-Created>

	kubectl expose deployment nginx --type=NodePort --port=80 --target-port=80 --name=my-dep-svc

	kubectl get svc

 **Update Deployments**

 Deployments can be updated by 2 methods:

 1. Set Image
 2. Edit Deployments

Updating Application version V1 to V2 using "Set Image" Option

Note: Check the container name in spec > container > name and replace in kubectl set image command.

	kubectl get deployments.apps nginx -o yaml

![image](https://github.com/user-attachments/assets/0b2691de-b7c6-4609-9b28-c48c8308af62)

	kubectl set image deployment nginx nginx=nginx:1.14.3		# Validate with describe, check in Image section

 	kubectl rollout status deployment nginx				# Rolling out all pods, Validate with 'get pods' command.

  	kubectl get deployments.apps

**Update application using "Edit Deployment Version"**

	kubectl edit deployments.apps nginx

Version change from '1.14.2' to '1.14.3'

	spec:
      	  containers:
      	    - image: nginx:1.14.3

	kubectl rollout status deployment/nginx

 	kubectl rollout history deployment

  	kubectl rollout history deployment nginx --revision=2		# Check by changing the revision numbers

   	kubectl rollout undo deployment nginx				# Undo the rollback, It will also increase the rollout history

**Rollback to Specific version**

	kubectl rollout undo deployment nginx --to-revision=3

**Pause and Resume Deployment**

* If we want to make multiple changes to our deployment, we can pause the deployment make changes and resume it again.

      kubectl rollout pause deployment nginx

* Change the version using edit deployment command.	

      kubectl edit deployments.apps nginx	# We notice that not changes found after running the command, check with history command

      kubectl set resources deployments/nginx -c=nginx --limits=cpu=20m, memory=30Mi

* Resume deployment

      kubectl resume rollout deployment nginx


