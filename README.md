EKS is a managed Kubernetes offering or a enterprise supported Kubernetes from AWS.

* AWS EKS - EKS Cluster has four major parts:

1. EKS Control Plane: Contains Kubernetes master components like etcd, kube-apiserver, kube-controller. It is managed service by AWS.
2. Worker Nodes & Node Groups: Group of EC2 instance where we run our application workloads.
3. Fargate Profiles (Serverless): Instead of Ec2 instance, we run our application workloads on Serverless Fargate profiles.
4. VPC: With Vpc we follow secure networking standards which will allow us to run production workloads on EKS.

There are 3 ways to use the Kubernetes on AWS:

1. Self Managed - Creating Control and Data plane all managed by ourself, any patching or any issue faced in any of the components are managed only by us.
2. Eks 		- Control plane managed by Aws and Data managed by us.
3. Fargate	- Everything managed by Aws. It is a serverless architecture.

**Practical:-**

* Launch 1 Ubuntu Ec2 Instance (t2.micro)
* Create a Iam User
* Assign policy of Adminstrator Access to the user
* Generate Access Keys & Secret Access Keys
* Install AWS CLI on the instance (usually pre-installed on the instance)
      
Install Aws cli if not installed:

	apt install unzip -y
	curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
	unzip awscliv2.zip
	sudo ./aws/install
	aws --version

Connect User with Aws CLI:

	aws configure
    aws s3 ls							# To Validate the access

Install or Update Kubectl (Linux amd64): https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

	vi kubectl.sh

	#!/bin/bash
	set -e
	uname -m  # Should print x86_64
	mkdir -p kubectlbinary && cd kubectlbinary
	curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.5/2025-09-19/bin/linux/amd64/kubectl
	chmod +x ./kubectl
	mkdir -p $HOME/bin
	cp ./kubectl $HOME/bin/kubectl
	export PATH=$HOME/bin:$PATH
	grep -qxF 'export PATH=$HOME/bin:$PATH' ~/.bashrc || echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
	kubectl version --client

Note: Run the below source command separately after running the above script

	source ~/.bashrc

Install eksctl cli: https://eksctl.io/installation/ | https://docs.aws.amazon.com/eks/latest/eksctl/what-is-eksctl.html

	vi eks.sh

	# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
	ARCH=amd64
	PLATFORM=$(uname -s)_$ARCH
	curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
	# (Optional) Verify checksum
	curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
	tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
	sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
	
    eksctl version

Setup EKS cluster:

* It will take 15-20 minutes to create the Cluster Control Plane

		eksctl create cluster --name=sunny-eks-cluster \
		                      --region=ap-south-1 \
		                      --zones=ap-south-1a,ap-south-1b \
		                      --without-nodegroup

		eksctl get cluster

  		eksctl delete cluster sunny-eks-cluster

* Installation with specifying nodes:

		eksctl create cluster --name sunny-eks-cluster --region=ap-south-1 --node-type t2.medium --nodes-min 1 --nodes-max 2

Note: To validate the EKS cluster installation > go to CloudFormation and check for Events

IAM OIDC provider:

* To enable and use IAM roles with Kubernetes service account on EKS cluster we must create and associate OIDC identity provider.
* An IAM OpenID Connect (OIDC) provider in AWS (or other cloud providers) is an authentication protocol built on top of OAuth 2.0 that allows users to authenticate with one service and use those credentials to access other services. An OIDC provider (like Google, Facebook, or a custom IdP) is responsible for verifying user identities and issuing tokens.
* IAM OIDC providers in AWS are used to create a trust relationship. This means you're essentially telling AWS, "I trust this particular OIDC provider to authenticate users, and when they present a valid token from that provider, I'll grant them access to certain AWS resources".

		eksctl utils associate-iam-oidc-provider --region=ap-south-1 --cluster sunny-eks-cluster --approve

If having multiple cluster

    aws eks update-kubeconfig --name <cluster-name> --region us-east-1

* Create Node Group with additional Add-Ons in Public Subnets. These add-ons will create the respective IAM policies for us automatically within our Node Group role.

		eksctl create nodegroup --cluster=sunny-eks-cluster \
		                       			--region=ap-south-1 \
		                       			--name=sunny-eks-cluster-ng-public1 \
		                       			--node-type=t2.medium \
		                       			--nodes=1 \
		                       			--nodes-min=1 \
		                       			--nodes-max=1 \
		                       			--node-volume-size=20 \
		                       			--ssh-access \
		                       			--ssh-public-key=awskey-1 \
		                       			--asg-access \
		                       			--external-dns-access \
		                       			--full-ecr-access \
		                       			--appmesh-access \
		                       			--alb-ingress-access \
		  								--verbose=3

Delete Cluster & Nodegroups

		aws cloudformation describe-stacks   --region ap-south-1   --query "Stacks[?contains(StackName, 'sunny-eks-cluster-ng-public1')].StackName"

		aws cloudformation delete-stack --stack-name eksctl-sunny-eks-cluster-nodegroup-sunny-eks-cluster-ng-public1 --region ap-south-1		# if any issues creation of nodes & deletion will take time

			aws cloudformation describe-stack-events \
		  --stack-name eksctl-sunny-eks-cluster-nodegroup-sunny-eks-cluster-ng-public1 \
		  --region ap-south-1 \
		  --query 'StackEvents[?ResourceStatus==`CREATE_FAILED`].{Resource:LogicalResourceId, Reason:ResourceStatusReason}' \
		  --output table		# this will the logs from cloudformation

	eksctl get nodegroup --cluster sunny-eks-cluster  			# Get nodegroup name

	eksctl delete nodegroup --cluster=sunny-eks-cluster --name=sunny-eks-cluster-ng-public1

 	eksctl delete cluster sunny-eks-cluster

Verify cluster:

	eksctl get nodegroup --cluster sunny-eks-cluster

 	kubectl get nodes -o wide

  	kubectl config view --minify

# EKS Storage with Elastic block storage (EBS)

1. EBS CSI Driver: The Amazon Elastic Block Store (Amazon EBS) Container Storage Interface (CSI) driver manages the lifecycle of Amazon EBS volumes as storage for the Kubernetes Volumes that you create. The Amazon EBS CSI driver makes Amazon EBS volumes for these types of Kubernetes volumes: generic ephemeral volumes and persistent volumes.
   
2. EFS CSI Driver: Amazon Elastic File System (Amazon EFS) provides serverless, fully elastic file storage so that you can share file data without provisioning or managing storage capacity and performance. The Amazon EFS Container Storage Interface (CSI) driver provides a CSI interface that allows Kubernetes clusters running on AWS to manage the lifecycle of Amazon EFS file systems.
  
3. Amazon FSx for Lustre CSI driver: The FSx for Lustre Container Storage Interface (CSI) driver provides a CSI interface that allows Amazon EKS clusters to manage the lifecycle of FSx for Lustre file systems.

* Go to IAM & create new policy named Amazon_EKS_sunny_CBI_Driver


		{
		  "Version": "2012-10-17",
		  "Statement": [
		    {
		      "Effect": "Allow",
		      "Action": [
		        "ec2:AttachVolume",
		        "ec2:CreateSnapshot",
		        "ec2:CreateTags",
		        "ec2:CreateVolume",
		        "ec2:DeleteSnapshot",
		        "ec2:DeleteTags",
		        "ec2:DeleteVolume",
		        "ec2:DescribeInstances",
		        "ec2:DescribeSnapshots",
		        "ec2:DescribeTags",
		        "ec2:DescribeVolumes",
		        "ec2:DetachVolume"
		      ],
		      "Resource": "*"
		    }
		  ]
		}

		kubectl describe configmap aws-auth -n kube-system

* Get a role, copy rolearn (**eksctl-sunny-eks-cluster-nodegroup-NodeInstanceRole-Nj0ZJ32vJcpJ**) search <-- this in a IAM **Role**
* In the selected role, go to add permissions, attach policy (Amazon_EKS_sunny_CBI_Driver)

- Other way to finding a role attached with the worker node is from the EC2 dashboard under IAM section

<img width="1040" height="313" alt="image" src="https://github.com/user-attachments/assets/a094e586-cd37-465f-9b81-c25af58c2d83" />

		kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=master"		# It might show some error at last
		kubectl get pods -n kube-system			# look for csi controller and node

<img width="807" height="65" alt="image" src="https://github.com/user-attachments/assets/7a209f84-3f7f-4eac-a926-7477bd3b7b02" />

		kubectl get deploy -n kube-system

* Creating mysql database with persistent storage using Aws Ebs volume

		sudo vim ~/.vimrc --> set ai ts=2 cursorcolumn et				# set indentation
		vim storage.yml

		apiVersion: storage.k8s.io/v1
		kind: StorageClass
		metadata:
		  name: ebs-sc
		provisioner: ebs.csi.aws.com
		volumeBindingMode: WaitForFirstConsumer

* In Amazon EKS, **ebs.csi.aws.com** is the provisioner used by the Amazon EBS CSI (Container Storage Interface) driver. This driver handles the dynamic provisioning of Amazon EBS volumes for use with Kubernetes workloads running on EKS.
* When a PVC is created with a StorageClass configured with **volumeBindingMode: WaitForFirstConsumer**, the PVC will remain in a "Pending" state, even if the underlying storage class is configured to provision volumes dynamically.

		vim pvc.yml

		apiVersion: v1
		kind: PersistentVolumeClaim
		metadata:
		  name: ebs-mysql-pv-claim
		spec:
		  accessModes:
		    - ReadWriteOnce
		  storageClassName: ebs-sc
		  resources:
		    requests:
		      storage: 4Gi


		vim configmap.yml

		apiVersion: v1
		kind: ConfigMap
		metadata:
		  name: usermanagement-dbcreation-script
		data:
		  mysql_usermgmt.sql: |-
		    DROP DATABASE IF EXIST usermgmt;
		    CREATE DATABASE usermgmt;

		mkdir kube-manifest
		mv storage.yml pvc.yml configmap.yml kube-manifest
		kubectl apply -f kube-manifest
		kubectl get pvc		# It will be in pending state


  apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:5.6
          env:
          - name: MYSQL_ROOT_PASSWORD
            values: dbpassword11
          ports:
          - containerPort: 3306
            name: mysql
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
            - name: usermanagement-dbcreation-script
              mountPath: /docker-entrypoint-initdb.d

      volumes:
        - name: mysql-persistent-storage
          persistent-volume-claim:
            claimName: ebs-mysql-pv-claim
        - name: usermanagement-dbcreation-script
          configMap:
            name: usermanagement-dbcreation-script
		




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

    kubectl exec -it nginx -- /bin/bash				# Go Inside the Pod

    kubectl logs nginx								# Get Pod logs
    kubectl logs -f  nginx							# Running logs
    kubectl exec -ti nginx ls						# Run command outside the pod
    kubectl exec -ti nginx env
    kubectl exec -ti nginx cat /usr/share/nginx/html/index.html
    kubectl get pods nginx -o yaml					# Get yaml file of Pod 
    kubectl get pods nginx -o yaml > nginx.yaml		# Re-direct content of yaml to another file
    kubectl get svc my-nginx-svc -o yaml			# Get yaml file for service 
    

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

 	kubectl scale --replicas=20 deployment nginx				# Imparative Way

	apiVersion: apps/v1											# Declarative Way
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

 	kubectl rollout status deployment nginx						# Rolling out all pods, Validate with 'get pods' command.

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

   	kubectl rollout undo deployment nginx						# Undo the rollback, It will also increase the rollout history

**Rollback to Specific version**

	kubectl rollout undo deployment nginx --to-revision=3

**Pause and Resume Deployment**

* If we want to make multiple changes to our deployment, we can pause the deployment make changes and resume it again.

      kubectl rollout pause deployment nginx

* Change the version using edit deployment command.	

      kubectl edit deployments.apps nginx											# We notice that not changes found after running the command, check with history command

      kubectl set resources deployments/nginx -c=nginx --limits=cpu=20m, memory=30Mi

* Resume deployment

      kubectl resume rollout deployment nginx

# EKS Project



