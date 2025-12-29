## ECS full stack

### Step 1: Create EC2 Instance (AWS Console) and attach IAM role
Login to AWS Console → EC2 → Instances → Launch Instance
Security Group:
* Allow SSH (port 22) from your IP
* Allow HTTP (port 80) for frontend
* Allow Custom TCP 3000 for backend API

*IAM role* 
Click Instances → select your EC2 instance →Click Actions→ Security → Modify IAM Role→Create new IAM role
Use case: EC2
Permissions:
* AmazonEC2ContainerRegistryFullAccess
* AmazonECSFullAccess
role name :"ecs-ec2-role"
comeback to modify and update role

### Step 2: Connect via MobaXterm
public ip → ec2-user → pemkey

### Step 3: Install Required Software on EC2
Update packages
`sudo yum update -y`

Install Docker
```
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker $USER
```

Install Node.js and npm
```
curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
sudo yum install -y nodejs
```

Install Apache (for frontend)
```
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

Check installations:
```
docker --version
node -v
npm -v
httpd -v
```

### Step 4: Create Backend App (Node.js)
Create folder:
`mkdir backend && cd backend`

Initialize Node.js project:
`npm init -y`

Install Express:
`npm install express`

Create index.js:
`nano index.js`

```
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.send('Hello from Backend!');
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

Run the file using Node.js:
`node index.js`

ecpected output:Server running on port 3000

Test in browser:http://<EC2-PUBLIC-IP>:3000

### Step 5: Create Frontend
```
mkdir ~/frontend
cd ~/frontend
```

Create index.html:
`nano index.html`

Important: Replace <EC2-PUBLIC-IP> with your actual EC2 public IP.
```
<!DOCTYPE html>
<html>
<head>
  <title>Fullstack App</title>
</head>
<body>
  <h1>Hello from Frontend!</h1>
  <button onclick="callBackend()">Call Backend</button>
  <p id="response"></p>

  <script>
    async function callBackend() {
      const res = await fetch('http://<EC2-PUBLIC-IP>:3000'); 
      const text = await res.text();
      document.getElementById('response').innerText = text;
    }
  </script>
</body>
</html>
```

*Serve Frontend via Apache:*
Copy frontend files to Apache default folder:
`sudo cp -r ~/frontend/* /var/www/html/`

Start Apache and enable it to run on boot:
```
sudo systemctl start httpd
sudo systemctl enable httpd
```

Test Frontend:
http://<EC2-PUBLIC-IP>


### Step 7: Dockerize Backend
```
cd ~/backend
ls
```
*Create Dockerfile:*
`nano Dockerfile`

```
FROM node:20

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "index.js"]
```

*Start Docker service*
`sudo service docker start`

*Build Docker image*
`docker build -t backend-app .`

*Run Docker container (TEST):*
`docker run -p 3000:3000 backend-app`

Open browser:http://<EC2-PUBLIC-IP>:3000
expected :Hello from Backend!

### step8 :Create ECR (Elastic Container Registry) & Push Image
Check AWS CLI
`aws sts get-caller-identity`

*Create ECR Repository*
`aws ecr create-repository --repository-name backend-app`
Copy your repositoryUri

*Login Docker to ECR*
replace region and account id (aws account id)
```
aws ecr get-login-password --region <region> \
| docker login --username AWS --password-stdin <ACCOUNT-ID>.dkr.ecr.<region>.amazonaws.com
```

*Tag Docker Image*
```
replace account id and region
docker tag backend-app:latest <ACCOUNT-ID>.dkr.ecr.<region>.amazonaws.com/backend-app:latest
```

Push Image to ECR:
```
docker push <ACCOUNT-ID>.dkr.ecr.<region>.amazonaws.com/backend-app:latest
```











