# EKS backend 

* What We Will Do (Overview)*

✅ Create a simple backend inside EC2 using MobaXterm
✅ Build Docker image on EC2
✅ Push image to Amazon ECR
✅ Create EKS cluster
✅ Deploy backend using kubectl
✅ Access backend using public IP

### STEP 1: Create an EC2 Instance
Create EC2→Amazon Linux 2→Instance type: t3.medium *(important!)*→Storage: 20 GB
Security Group:
SSH (22) →  *My IP*
HTTP (80) → Anywhere

### IAM Role:
AWS Console → IAM → Roles 
Attach Permission Policy → AdministratorAccess

 ### Attach IAM Role to EC2
 IAM → Roles → Trusted entity type: AWS service → Use case: EC2
 Attach Permission →AdministratorAccess

role name :EKS-Admin-Role

### Attach IAM Role to EC2
ec2 instance → action → security → modify iam role → EKS-Admin-Role

### connect moba :
Open MobaXterm SSH → Username: ec2-user

 to verify IAM role ARN, it worked `aws sts get-caller-identity`

 ### step2 : install 
 update `sudo yum update -y`

 Install Docker(for containers)
 ```
sudo yum install docker -y
sudo systemctl start docker
sudo usermod -aG docker ec2-user
newgrp docker
```

Install kubectl (Kubernetes CLI)
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.29.0/2024-01-04/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

Install eksctl (EKS creator tool)
```
curl --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" | tar xz
sudo mv eksctl /usr/local/bin
eksctl version
```
### STEP 3: Create EKS Cluster (Theory)
change region

```
eksctl create cluster \
--name backend-cluster \
--region ap-south-1 \
--nodegroup-name backend-nodes \
--node-type t3.medium \
--nodes 2
```

verify cluster `kubectl get nodes`

### STEP 5: Create Backend App
Create folder
```
mkdir backend
cd backend
```
Create Node.js app `nano app.js`
```
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('EKS Backend is running 🚀');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```
CTRL + O → Enter → CTRL + X

### Create package.json
`nano package.json`

```
{
  "name": "eks-backend",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

 ### STEP 6: Create Dockerfile

 `nano Dockerfile`

 ```
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "app.js"]
```

Build Docker image `docker build -t eks-backend .`

### STEP 7: Push Image to Amazon ECR
Create ECR repo:
`aws ecr create-repository --repository-name eks-backend --region ap-south-1`
### Copy repository URI

no need to change region because ECR repository was created in ap-south-1

```
aws ecr get-login-password --region ap-south-1 \
| docker login --username AWS --password-stdin <ECR_URI>
```

expected output:login sceessed 

tag & push :
change ECR uri
```
docker tag eks-backend:latest <ECR_URI>:latest
docker push <ECR_URI>:latest
```
### STEP 8: Deploy App to EKS
Create deployment file:
```
nano deployment.yaml
```
change uri from below code
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: nginx:latest   
        ports:
        - containerPort: 80

```

Apply deployment
```
kubectl apply -f deployment.yaml
kubectl get pods
```

### STEP 9: Expose Backend Publicly
Create service:
`nano service.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80        
      targetPort: 3000 

```

Apply service:
```
kubectl apply -f service.yaml
kubectl get svc
```

check and get ip address :
```
kubectl get svc
```
expected output
EXTERNAL-IP   a1b2c3d4.eu-north-1.elb.amazonaws.com

Open in browser and seach the ip 
