# ECS Backend Deployment

* EC2 (Docker)
* ECR (Docker Image)
* ECS Cluster → Service → Task 
* Application Load Balancer
* Public URL

  ### step1:create Ec2 instance and connect with Moba

create instance with ubuntu → Security Group:
SSH (22) → My IP
HTTP (80) → Anywhere
Custom TCP (3000) → Anywhere

### step2: Update system & install Docker

Update packages, Install Docker , Start & enable Docker , Give Docker permission
``` sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
exit
```

After *exit*, close the terminal and reconnect using MobaXterm.

verify docker installation `docker --version`

### step3: Create a Backend App (INSIDE EC2)
Create backend folder:
```
mkdir backend
cd backend
```

Install Node.js 18:
```
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install npm -y
```

verify
```
node -v
npm -v
```

Initialize Node project
```
npm init -y
npm install express
```
### STEP 4: Create Backend Entry File
Create index.js:
`nano index.js`

```
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello from Ubuntu EC2 Backend');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```
CTRL + O → Enter → CTRL + x

### STEP 5: Run Backend without docker
`node index.js`
Expected output: Server running on port 3000

### STEP 6: Access Backend from Browser
Open a browser and go to: http://13.xxx.xxx.xxx:3000
 
 stops the server CTRL + C

### step 7 :Dockerize the app
Create Dockerfile : `nano Dockerfile`

```
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

Build Docker image : `docker build -t backend-app .`

### STEP 8: Run Backend Inside Docker (Test Locally)
Run container
`docker run -d -p 3000:3000 --name backend-test backend-app`
Check container status
`docker ps`

Expected output: Container name: backend-test
Port mapping: 0.0.0.0:3000->3000

open browser and check again for same results

### STEP 9: Create Amazon ECR Repository (Console)
Go to AWS Console → Search ECR → Create repository → Repository name: backend-app → create

### step10 : Configure AWS CLI (One Time)
Download AWS CLI v2 :
`curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip`

Install unzip
```
sudo apt install unzip -y
unzip awscliv2.zip
```

Install AWS CLI `sudo ./aws/install`
expected o/t : You can now run: /usr/local/bin/aws --version

 verify : `aws --version`
 expected o/t : aws-cli/2.x.x ...

Configure AWS CLI : `aws configure`

GO to IAM → create user → name → attach policies →create → open user → Security credentials→ Create access key→CLI
*AmazonEC2ContainerRegistryFullAccess
*AmazonECSFullAccess
*CloudWatchLogsFullAccess

in moba output form json

### STEP 11: Create an ECS Cluster
`aws ecs create-cluster --cluster-name backend-cluster`

`nano task-def.json`

```
{
  "family": "backend-task",
  "networkMode": "awsvpc",
  "containerDefinitions": [
    {
      "name": "backend-container",
      "image": "public.ecr.aws/docker/library/node:18",
      "memory": 512,
      "cpu": 256,
      "essential": true,
      "portMappings": [
        {
          "containerPort": 3000,
          "hostPort": 3000
        }
      ]
    }
  ],
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512"
}
```

Register the task definition with AWS CLI - `aws ecs register-task-definition --cli-input-json file://task-def.json`

press *q*

### step12 :Run the ECS Task 
Create a Fargate ECS cluster `aws ecs create-cluster --cluster-name backend-cluster`

find default vpc , sub group ,security group

```
#!/bin/bash

# Set AWS region
REGION="eu-north-1"

# 1️⃣ Get default VPC
DEFAULT_VPC=$(aws ec2 describe-vpcs \
  --filters "Name=isDefault,Values=true" \
  --query "Vpcs[0].VpcId" \
  --output text --region $REGION)

echo "Default VPC: $DEFAULT_VPC"

# 2️⃣ Get first subnet in default VPC
DEFAULT_SUBNET=$(aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=$DEFAULT_VPC" \
  --query "Subnets[0].SubnetId" \
  --output text --region $REGION)

echo "Default Subnet: $DEFAULT_SUBNET"

# 3️⃣ Get first security group in default VPC
DEFAULT_SG=$(aws ec2 describe-security-groups \
  --filters "Name=vpc-id,Values=$DEFAULT_VPC" \
  --query "SecurityGroups[0].GroupId" \
  --output text --region $REGION)

echo "Default Security Group: $DEFAULT_SG"

# 4️⃣ Run ECS task on Fargate
aws ecs run-task \
  --cluster backend-cluster \
  --launch-type FARGATE \
  --task-definition backend-task \
  --network-configuration "awsvpcConfiguration={subnets=['$DEFAULT_SUBNET'],securityGroups=['$DEFAULT_SG'],assignPublicIp=ENABLED}" \
  --region $REGION
```

 verify task is running
 `aws ecs list-tasks --cluster backend-cluster --region eu-north-1`

 copy task id from abouve code edit below and run 
 `aws ecs describe-tasks --cluster backend-cluster --tasks 6ffc0b20418144bda639423c8c5c5166 --region eu-north-1`

 

 

















