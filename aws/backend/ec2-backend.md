# EC2 backend deployment

**workflow**
* Create EC2 instance
* Connect to EC2 using MobaXterm
* Install Node.js
* Create backend project directly on EC2
* Run backend
* Keep backend running (PM2)
* Open port in EC2 Security Group
* Access backend from browser

*Step1*
create EC2 instance with Ubuntu 
open instance and security group link and edit inbout rule with custome tcp - 3000 - anywhere ipv4

*step2*
open mobaexterm with session
update - `sudo apt update && sudo apt upgrade -y`

Install Node.js - `curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -`
`sudo apt install -y nodejs`

*step3*
create backend folder - `mkdir backend`
`cd backend`
create project - `npm init -y`
Install Express: - `npm install express`

*step4* Create Backend File (No Uploading)
Create file: `nano index.js`

`const express = require('express');
const app = express();

const PORT = 3000;

app.get('/', (req, res) => {
  res.send('Backend is running on EC2 🚀');
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});`

clt+o , enter , ctrl+x

*step 5* Run Backend (Testing)
`node index.js`
output :Server running on port 3000

test in browser as "http://EC2-PUBLIC-IP:3000:






