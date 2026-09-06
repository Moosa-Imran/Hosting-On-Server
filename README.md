# Hosting a Node.js App on a Linux Server (Ubuntu, Hostinger)

This guide provides step-by-step instructions for hosting a **Node.js application** on an Ubuntu-based server (such as **Hostinger**). It includes installing dependencies, setting up a **process manager**, configuring **Nginx as a reverse proxy**, enabling **SSL with Certbot**, securing with **UFW firewall**, and setting up **MongoDB**.

✅ If this guide helps you, **please give it a ⭐ on GitHub!** Your support encourages further improvements.

---

## 📂 Files Included in This Repository

| File | Description |
|------|------------|
| `README.md` | This guide on hosting a Node.js app |
| `install.sh` | A script to automate installation of Node.js, MongoDB, and dependencies |
| `mongodb-config.sample` | Sample configuration file for MongoDB (`/etc/mongod.conf`) |
| `nginx-config.sample` | Sample Nginx configuration for reverse proxy (`/etc/nginx/sites-available/yourdomain.com`) |
| `pm2-startup.sh` | PM2 setup script to start the Node.js app on reboot |
| `LICENSE` | MIT License for this open-source project |

---

## 🚀 1. Install Node.js

First, **update your system**:

```sh
sudo apt update && sudo apt upgrade -y
```

Now, install Node.js:

```sh
curl -fsSL https://deb.nodesource.com/setup_current.x | sudo -E bash -
sudo apt install -y nodejs
```

Verify installation:

```sh
node -v
npm -v
```

✅ **Automate this step:** You can use the included [`install.sh`](install.sh) script to install Node.js and other dependencies automatically.

🔴 **Common Mistake:** Ensure you install the correct version of Node.js that matches your application's requirements.

---

## 🛢️ 2. Install and Configure MongoDB

MongoDB is a NoSQL database commonly used with Node.js applications.

### Install MongoDB

```sh
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt update
sudo apt install -y mongodb-org
```

### Start and Enable MongoDB

```sh
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod
```

### Secure MongoDB Configuration

Before modifying the MongoDB configuration file, **stop MongoDB** to avoid crashes:

```sh
sudo systemctl stop mongod
```

Now, edit the MongoDB config file:

```sh
sudo nano /etc/mongod.conf
```

Modify the following sections (**or use the provided [`mongodb-config.sample`](mongodb-config.sample) file**):

```yaml
security:
    authorization: "enabled"
```

```yaml
net:
    bindIp: 0.0.0.0
```

Restart MongoDB:

```sh
sudo systemctl start mongod
sudo systemctl status mongod
```

> If you want to connect to MongoDB from MongoDB Compass, make sure MongoDB traffic is not blocked by the firewall. If you are using UFW, allow the MongoDB port so Compass can connect:
>
> ```sh
> sudo ufw allow 27017/tcp
> sudo ufw status
> ```
>
> This ensures MongoDB is not excluded by the firewall when using Compass.

✅ **Automate this step:** The [`install.sh`](install.sh) script can also automate MongoDB installation and configuration.

🔴 **Common Mistake:** Always stop MongoDB before modifying its configuration file to prevent crashes.

### Install PostgreSQL

You can install MongoDB or PostgreSQL — choose whichever fits your requirements.

Install PostgreSQL with:

```sh
sudo apt install postgresql postgresql-contrib -y
sudo systemctl status postgresql
```

Create a database user/role and a demo database:

```sh
sudo -u postgres psql
CREATE ROLE admin WITH LOGIN SUPERUSER CREATEDB CREATEROLE PASSWORD 'YOUR_STRONG_PASSWORD';
CREATE DATABASE demo OWNER admin;
\q
```

This gives you a PostgreSQL admin account and a `demo` database for your application.

---

## 📦 3. Clone Your Node.js Application

```sh
git clone <repo-name>
cd <repo-name>
```

---

## 🔧 4. Install Build Tools

```sh
sudo apt install build-essential
```

---

## ⚙️ 5. Install and Configure PM2 (Process Manager)

PM2 helps in **managing and keeping your Node.js app running**.

```sh
sudo npm install pm2@latest -g
pm2 start server.js --name "my-node-server"
pm2 save
pm2 startup
```

Enable PM2 on system startup (**or use the included [`pm2-startup.sh`](pm2-startup.sh) script**):

```sh
sudo systemctl start pm2-my-node-server
```

🔴 **Common Mistake:** Ensure PM2 is properly saved and starts automatically on reboot by running `pm2 save` and `pm2 startup`.

---

## 🌍 6. Install and Configure Nginx as a Reverse Proxy

### Install Nginx

```sh
sudo apt update
sudo apt install nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### Configure Nginx for Your Domain

```sh
sudo nano /etc/nginx/sites-available/yourdomain.com
```

Paste the following configuration (**or use the provided [`nginx-config.sample`](nginx-config.sample) file**):

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        client_max_body_size 5m;
    }

    error_log /var/log/nginx/yourdomain_error.log;
    access_log /var/log/nginx/yourdomain_access.log;
}
```

Enable the configuration:

```sh
sudo ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## 🔒 7. Secure Your Application with SSL (Let's Encrypt)

```sh
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
sudo certbot renew --dry-run
```

---

## 🛡️ 8. Configure Firewall

```sh
sudo ufw status
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

If you are using MongoDB and need to connect with MongoDB Compass, make sure the firewall does not block MongoDB access. For example, allow the MongoDB port:

```sh
sudo ufw allow 27017/tcp
sudo ufw status
```

This prevents MongoDB from being blocked while allowing your app and Compass to connect securely.

🔴 **Common Mistake:** Ensure you allow `OpenSSH` before enabling UFW, or else you might lock yourself out of the server.

---

## 📜 License

This project is licensed under the **MIT License**. See the [`LICENSE`](LICENSE) file for details.

---

## 🙌 Contributions & Support

🔹 **Contributions are welcome!** If you have improvements or suggestions, feel free to open a pull request.

💡 **Found this helpful?** Please **give this repo a ⭐!**

---

## 📚 Additional Resources

- [DigitalOcean Node.js Production Guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-20-04)
- [NodeSource Distributions](https://github.com/nodesource/distributions)
- [GeekyShows Nginx SSL Guide](https://github.com/geekyshow1/GeekyShowsNotes/blob/main/nginx/SSL_Cert_Nginx.md)

---

This guide provides a streamlined approach to setting up a **Node.js application on an Ubuntu server**. If you have any questions or improvements, feel free to **contribute!** 🚀
