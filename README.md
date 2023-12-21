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

Note: To validate the EKS cluster installation > go to CloudFormation and check for Events

Check how many cluster available?

    eksctl get cluster

If having multiple cluster

    aws eks update-kubeconfig --name <cluster-name> --region us-east-1

    



