# MERN Blog Application - VM Deployment Guide

This guide details exactly how to deploy your "Proper MERN" Blog Application on a fresh Cloud Virtual Machine (such as AWS EC2, GCP, or Azure).

---

## Prerequisites

1. You have launched a Virtual Machine (Ubuntu 22.04 / 24.04 recommended).
2. You have configured the **Security Group / Firewall** to allow Traffic on **Port 80 (HTTP)** and **Port 22 (SSH)**.
3. You have your SSH Key file (`.pem`).

### Step 1: Connect to your VM via SSH

Open your local terminal and run:

```bash
ssh -i /path/to/your-key.pem ubuntu@<YOUR_VM_PUBLIC_IP>
```

_(Replace `ubuntu` with `root`, `ec2-user`, or your specific cloud provider's default username)._

---

### Step 2: Transfer your Application Files to the VM

You need to transfer the `blog_mern_app` folder to the remote VM.

Open a NEW terminal window on your local machine and run:

```bash
scp -i /path/to/your-key.pem -r "d:\insem\LP2\blog_mern_app" ubuntu@<YOUR_VM_PUBLIC_IP>:~/
```

_(Once transferred, go back to your SSH terminal)._

---

## ✅ One-Terminal Deployment (Automated)

Run the setup script. It handles Node.js, MongoDB, Nginx, PM2, and your build automatedly.

```bash
cd folder_name
sudo bash setup.sh
```

**Your app is now live!** The script automatically registers the backend with PM2 and serves the frontend on Port 80.

## 📊 Management Commands

Use these from a single terminal:

- `pm2 status`: Check if services are running.
- `pm2 logs`: View real-time activity.
- `mongosh <db_name>`: Access the database.

ssh -i LP2_key.pem azureuser@20.193.248.78
Warning: Identity file LP2_key.pem not accessible: No such file or directory.
azureuser@20.193.248.78: Permission denied (publickey).
PS C:\Users\91798> cd .\Downloads\
PS C:\Users\91798\Downloads> ssh -i LP2_key.pem azureuser@20.193.248.78
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.17.0-1011-azure x86_64)

- Documentation: https://help.ubuntu.com
- Management: https://landscape.canonical.com
- Support: https://ubuntu.com/pro

System information as of Sun Apr 19 10:43:52 UTC 2026

System load: 0.11 Processes: 135
Usage of /: 13.5% of 28.02GB Users logged in: 1
Memory usage: 49% IPv4 address for eth0: 10.0.0.4
Swap usage: 0%

Expanded Security Maintenance for Applications is not enabled.

1 update can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

Last login: Sun Apr 19 10:36:27 2026 from 103.97.166.170
azureuser@LP2:~$ mongosh blog_db
Current Mongosh Log ID: 69e4b1f097cd2ffe3644ba88
Connecting to: mongodb://127.0.0.1:27017/blog_db?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.8.2
Using MongoDB: 7.0.31
Using Mongosh: 2.8.2

For mongosh info see: https://www.mongodb.com/docs/mongodb-shell/

---

The server generated these startup warnings when booting
2026-04-19T10:39:03.104+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
2026-04-19T10:39:03.434+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
2026-04-19T10:39:03.434+00:00: For customers running MongoDB 7.0, we suggest changing the contents of the following sysfsFile

---

blog_db> db.collection.find().pretty()

blog_db> db.collection.find().pretty()

blog_db> db.collection();
TypeError: db.collection is not a function
blog_db> db.collection
blog_db.collection
blog_db> show collections
posts
blog_db> db.posts.find();
[
{
_id: ObjectId('69e4b13b71b4d6c364c29d75'),
title: 'hello',
content: 'hello',
author: 'Contributor',
createdAt: ISODate('2026-04-19T10:40:59.159Z'),
__v: 0
},
{
_id: ObjectId('69e4b20b79761e0992077a66'),
title: '123',
content: '123',
author: 'Contributor',
createdAt: ISODate('2026-04-19T10:44:27.243Z'),
__v: 0
}
]
blog_db>

## 🔎 Useful MongoDB Queries

- List all entries: `db.collection.find().pretty()`
- Count entries: `db.collection.countDocuments()`
- Delete all data: `db.collection.deleteMany({})`

## Deployment Option B: Manual Execution (Without `setup.sh`)

If the faculty wants to see raw terminal commands without automation, run these step-by-step:

### 1. System Update & Dependencies

```bash
sudo apt update
sudo apt install -y curl nginx
```

### 2. Install Node.js & MongoDB

```bash
# Node.js
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# MongoDB
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg --batch --yes -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl enable --now mongod
```

### 3. Install & Build Application

```bash
cd ~/blog_mern_app

# Backend
cd backend
npm install
sudo npm install -g pm2

# Frontend
cd ../frontend
npm install
npm run build
```

### 4. Start the Backend API

Use PM2 to start the background Express process so it survives SSH disconnects:

```bash
cd ~/blog_mern_app/backend
pm2 start server.js --name "blog-api"
```

### 5. Configure Nginx Reverse Proxy

By default, Nginx displays an Ubuntu welcome page. Replace it with your application.

```bash
sudo rm -f /etc/nginx/sites-enabled/default
sudo nano /etc/nginx/sites-available/mern_blog
```

Paste the following configuration into the NGINX editor. **Make sure to replace `/home/ubuntu/blog_mern_app` with your actual absolute path!**

```nginx
server {
    listen 80;
    server_name _;

    # Serve the optimized React build
    root /home/ubuntu/blog_mern_app/frontend/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Proxy the API requests entirely to Node
    location /api/ {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Save (Ctrl+O, Enter) and Exit (Ctrl+X). Then activate the config:

```bash
sudo ln -s /etc/nginx/sites-available/mern_blog /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

**Done!** Browse to `http://<YOUR_VM_PUBLIC_IP>` to see the app.
