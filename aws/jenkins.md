## jenkins

* create ec2 instance :
allow inbound with 8080 

* open mobaexterm with ssh

### step1: Update the server
```
sudo apt update -y
sudo apt upgrade -y
```

### Step2: Install Java
`sudo apt install -y fontconfig openjdk-17-jre`

### step3: Install Jenkins
Add Jenkins key:
```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```
Add Jenkins repository:
```
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```
Install Jenkins:
```
sudo apt update -y
sudo apt install -y jenkins
```

### step4:Start Jenkins service
```
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

check :
`sudo systemctl status jenkins --no-pager`
✅ If you see active (running) Jenkins is running.

Open Jenkins in browser 

### step5:Unlock Jenkins
`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

# to run full stack
Install Node.js + NPM (for backend)
```
sudo apt install -y nodejs npm
node -v
npm -v
```
Install Nginx
```
sudo apt install -y nginx
sudo systemctl enable --now nginx
```
Test: 
http://<EC2-Public-IP>
You should see “Welcome to nginx”.

### step2: Allow Jenkins to run deploy commands cleanly


