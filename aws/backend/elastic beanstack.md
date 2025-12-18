# Elastic Beanstalk backend 

* Create an Elastic Beanstalk application
* Create an environment (Node.js)
* SSH into the EB EC2 instance
* Create backend files inside the server
* Run the backend
* Access it from browser

  **step1 - Create Elastic Beanstalk Application**
  apllication name , platform -Node.js ,
   Platform branch: Node.js 18 (or latest) ,
   application code - Sample application

  **step2 -SSH into Elastic Beanstalk EC2**
  Elastic Beanstalk → Environment → Configuration → services access - keypair - select/create key

  **step3 - Connect Elastic Beanstalk EC2 using MobaXterm**
   open EC2 → serach instance  → copy Public IPv4 address
    open moba → ssh → username(**ec2-user**) → ip

  **step 4 -Create Node.js Backend Files**
  `cd /var/app/current`

  Give Write Permission `sudo chown -R ec2-user:ec2-user /var/app/current`
  
  *Create package.json* - `nano package.json`

  ```
  {
  "name": "backend-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
} `
CTRL + O → ENTER → CTRL + X

*Create index.js* - `nano index.js`

```const express = require('express');
const app = express();

const PORT = process.env.PORT || 8081;

app.get('/', (req, res) => {
  res.send('Elastic Beanstalk Backend is running 🚀');
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
`
verify (optional) - `ls -l`
output :index.js , package.json

**step5 -Install Dependencies & Run Backend**
`npm install`

Start the Backend - `npm start`

ouput in moba -Server running on port 8081

Final output :http://<Elastic-Beanstalk-Environment-URL>

**if page is not openedor orginal page not opened**
`CTRL + C` - This stops npm start

`sudo systemctl restart web` - sudo systemctl restart web

`sudo tail -n 30 /var/log/web.stdout.log` - verify log

completed 






  
  
  
  
