## FUll stack deployment using ec2

User Browser
     ↓
Apache (Port 80)
     ↓
Reverse Proxy (/api)
     ↓
Node.js + Express (Port 3000)
     ↓
JSON Response


### STEP 1 prerequisites 
create ec2 instace → linux → allow http 
open instance → secuirty → edit inbound rule → custom tcp , 3000 , anywhere 

open mobaexter → public ip → ec2-user → pem.key 

### STEP 2 — Update server packages
```
sudo dnf update -y || sudo yum update -y
```
### STEP 3 — Install and start Apache (Frontend server)

*3.1 Install Amazon Linux 2023:*
`sudo dnf install -y httpd`

Amazon Linux 2:
`sudo yum install -y httpd`

*3.2 Start + enable*
```
sudo systemctl start httpd
sudo systemctl enable httpd
```

*3.3 Test Apache*
Open browser:
http://YOUR_PUBLIC_IP

### STEP 4 — Install Node.js + npm (Backend runtime)

*4.1 Install Node (simple way)*
```
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo dnf install -y nodejs || sudo yum install -y nodejs
```

*4.2 Verify*
```
node -v
npm -v
```

### STEP 5 — Create Backend (Express API) on port 3000

*5.1 Create folder*
```
mkdir -p ~/backend
cd ~/backend
```

*5.2 Initialize project*
```
npm init -y
npm install express cors
```

*5.3 Create server.js*
```
cat > server.js <<'EOF'
const express = require("express");
const cors = require("cors");

const app = express();
app.use(cors());
app.use(express.json());

app.get("/", (req, res) => res.send("Backend is running ✅"));

app.get("/api/hello", (req, res) => {
  res.json({ message: "Hello from Express API ✅" });
});

const PORT = 3000;
app.listen(PORT, "0.0.0.0", () => {
  console.log(`API running on port ${PORT}`);
});
EOF
```

*5.4 Run once (test)*
`node server.js`

## STEP 6 — Keep backend alive with PM2
*6.1 Install PM2 globally*
`sudo npm install -g pm2`

*6.2 Start backend with PM2*
```
cd ~/backend
pm2 start server.js --name backend
pm2 status
```

*6.3 Auto-start on reboot*
`pm2 startup`
It will print a command. Copy-paste that command exactly, then run:

`pm2 save`

## STEP 7 — Create Frontend (served by Apache)
*7.1 Create index.html*
edit public ip
```
sudo tee /var/www/html/index.html > /dev/null <<'EOF'
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Full Stack on EC2</title>
</head>
<body style="font-family: Arial; padding: 24px;">
  <h1>Frontend (Apache) ✅</h1>
  <p>Click the button to call backend API:</p>
  <button onclick="callApi()" style="padding:10px 16px;">Call API</button>
  <pre id="out" style="margin-top:16px; background:#f5f5f5; padding:12px;"></pre>

  <script>
    async function callApi() {
      const out = document.getElementById("out");
      out.textContent = "Calling API...";
      try {
        const res = await fetch("http://YOUR_PUBLIC_IP:3000/api/hello");
        const data = await res.json();
        out.textContent = JSON.stringify(data, null, 2);
      } catch (e) {
        out.textContent = "Error: " + e;
      }
    }
  </script>
</body>
</html>
EOF
```

*7.2 IMPORTANT: Replace YOUR_PUBLIC_IP*
```
PUBIP=$(curl -s http://checkip.amazonaws.com)
sudo sed -i "s/YOUR_PUBLIC_IP/$PUBIP/g" /var/www/html/index.html
```

*7.3 Restart Apache*
`sudo systemctl restart httpd`

Now open:
http://YOUR_PUBLIC_IP

Click Call API → you should see JSON message.

### STEP 8 — Clean, secure approach (Recommended): reverse proxy /api via Apache
*8.1 Enable proxy modules*
```
sudo tee /etc/httpd/conf.d/reverse-proxy.conf > /dev/null <<'EOF'
ProxyRequests Off
ProxyPreserveHost On

ProxyPass "/api/" "http://127.0.0.1:3000/api/"
ProxyPassReverse "/api/" "http://127.0.0.1:3000/api/"
EOF
```

Restart Apache:
`sudo systemctl restart httpd`

#completed




