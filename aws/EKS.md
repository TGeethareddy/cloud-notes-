 ## EKS
### step1
*Launch new Ubuntu VM using AWS Ec2 ( t2.micro ) AZ is virigina *
create with keypair and default security group .

*Connect to machine using below commands*
open mobaexterm with new ssh
`sudo apt update`

### step2
*instal cubectl*

paste in moba
`curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl`
`chmod +x ./kubectl`
`sudo mv ./kubectl /usr/local/bin`
`kubectl version --short --client`

*Install AWS CLI latest version*
`sudo apt install unzip`
`curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"`
`unzip awscliv2.zip`
`sudo ./aws/install`
`aws --version`

*Install eksctl*
`curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp`
`sudo mv /tmp/eksctl /usr/local/bin`
`eksctl version`

### step 3  Create IAM role & attach to EKS Management Host

*1* create IAM role - create new role - AWS service - use case (EC2) - permission (administor access ) - add any role name 
*2* attach role to EKS -(Select EC2 => actions -Click on Security => Modify IAM Role => attach IAM role we have created)

### step 4  Create EKS Cluster using eksctl

open moba 
N. Virgina: (depend on ec2 instance region )
`eksctl create cluster --name ashokit-cluster4 --region us-east-1 --node-type t2.medium  --zones us-east-1a,us-east-1b`

cluster creation started it will take 10 min

` kubectl get nodes`
we can see how many nodes are created .

### step5 Deploy a Sample App (Test EKS) in moba

 create deployment- 
 `kubectl create deployment nginx --image=nginx` 
 Expose service   -
 `kubectl expose deployment nginx --type=LoadBalancer --port=80`
 check service    -
 `kubectl get svc`
 
 You will get an EXTERNAL-IP

 ### step6 open in browser 

 in moba `kubectl get svc`
 output will get with external ip address like "10.100.45.23   a1b2c3d4e5f6g7.ap-south-1.elb.amazonaws.com"
 open browser and add Http://a1b2c3d4e5f6g7.ap-south-1.elb.amazonaws.com
 completed 
 





