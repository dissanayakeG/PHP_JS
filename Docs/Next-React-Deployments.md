- Setting up a VPS to host a React + Next.js project with security in mind requires several steps. Below is a step-by-step guide:

# Setup the VPS

1. Use SSH to connect:
```bash
ssh root@your-server-ip
```
If using an SSH key:
```bash
ssh -i /path/to/private-key root@your-server-ip
```

2. Update System Packages
```bash
sudo apt update && sudo apt upgrade -y
```

3. Create a New User (Non-root)
For security, create a new user and assign sudo privileges:
```bash
adduser myuser
usermod -aG sudo myuser
```
Switch to the new user:
```bash
su - myuser
```

4. Set Up a Firewall (UFW)
```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

5. Disable Root Login & Password Authentication
Edit SSH config:
```bash
sudo nano /etc/ssh/sshd_config
```
Change:
```plaintext
PermitRootLogin no
PasswordAuthentication no
```
Restart SSH:
```bash
sudo systemctl restart sshd
or
sudo systemctl start ssh
```

6. Install Node.js & Dependencies
Install Node.js (LTS)
```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo bash -
sudo apt install -y nodejs
```
Verify installation:
```bash
node -v
npm -v
```

7. Install PM2 for Process Management
```bash
sudo npm install -g pm2
```

8. Set Up Next.js App

   ```bash
   cd /var/www/
   git clone https://github.com/your-username/your-nextjs-app.git
   cd your-nextjs-app
   npm install
   npm run build
   # Start the app with PM2
   pm2 stop all
   pm2 delete all
   pm2 kill #Kill the PM2 Daemon
   
   pm2 start npm --name "app-name" -- start
   pm2 save
   pm2 startup

   pm2 restart app-name
   ```
   
9. Set Up Reverse Proxy with Nginx
**Install Nginx**
```bash
sudo apt install nginx -y
```

**Create an Nginx Config for Next.js**
```bash
sudo nano /etc/nginx/sites-available/nextjs
```
Add the following configuration:
```nginx
server {
    listen 80;
    server_name <YOURDOMAIN>.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
**Enable the Configuration**
```bash
sudo ln -s /etc/nginx/sites-available/<YOURDOMAIN>.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

10. Secure with SSL (Let's Encrypt)
**Install Certbot**
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   ping <YOURDOMAIN>.com //Ensure Domain is Pointing to Your Server
   sudo ufw allow 'Nginx Full' //Allow HTTP/HTTPS Traffic Through Your Firewall
   //Make sure that your firewall allows HTTP (port 80) and HTTPS (port 443) traffic. For example, if you're using UFW:
   ```
**Get an SSL Certificate**
   ```bash
   sudo certbot --nginx -d <YOURDOMAIN>.com -d www.<YOURDOMAIN>.com
   ```
**Auto-Renew SSL**
   ```bash
   sudo certbot renew --dry-run
   ```

11. Set Up Automatic Deployment (Optional)

**Install & Configure Git Hooks**
On VPS, go to your project directory
   ```bash
   cd ~/your-nextjs-app
   ```
Pull the latest changes automatically
   ```bash
   git pull origin main
   npm install
   npm run build
   pm2 restart next-app
   ```

**Automate Deployment with GitHub Actions or Webhooks**
- Set up a GitHub Actions workflow or a webhook to trigger `git pull` and `pm2 restart` on new commits.

12. Additional Security Measures

**Fail2Ban (Brute-force Protection)**
```bash
sudo apt install fail2ban -y
```
Enable it:
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

13. Disable Unused Services
```bash
sudo systemctl disable apache2
```

14. Monitor System Logs
```bash
sudo journalctl -u nginx --follow
sudo journalctl -u pm2-next-app --follow
```

15. Monitoring & Maintenance
- Use htop to monitor resource usage:
  ```bash
  sudo apt install htop -y
  htop
  ```
- Check logs:
  ```bash
  sudo tail -f /var/log/nginx/error.log
  ```
- Restart services if needed:
  ```bash
  sudo systemctl restart nginx
  pm2 restart next-app
  ```

✅ Done! Your Next.js app is now securely hosted on a VPS.

# Bash script to automate the VPS setup

Bash script to automate the VPS setup for hosting a Next.js app securely. 🚀  

It will:  
✅ Create a new user  
✅ Install Node.js, PM2, and Nginx  
✅ Set up a firewall (UFW) and SSH security  
✅ Deploy your Next.js app  
✅ Configure Nginx as a reverse proxy  
✅ Secure the site with Let's Encrypt SSL  

### VPS Setup Script
Save this as `setup-vps.sh` and run it on your VPS.  
```bash
#!/bin/bash

# Variables (Modify these)
USERNAME="deployuser"
APP_NAME="next-app"
GIT_REPO="https://github.com/your-username/your-nextjs-app.git"
DOMAIN="<YOURDOMAIN>.com"
EMAIL="your-email@example.com"

echo "🚀 VPS Setup for Next.js App 🚀"

# Step 1: Update System
echo "🔄 Updating system..."
sudo apt update && sudo apt upgrade -y

# Step 2: Create a New User
echo "👤 Creating new user: $USERNAME"
sudo adduser --gecos "" $USERNAME
sudo usermod -aG sudo $USERNAME

# Step 3: Secure SSH
echo "🔐 Securing SSH..."
sudo sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd

# Step 4: Install Dependencies
echo "📦 Installing required packages..."
sudo apt install -y curl git ufw nginx certbot python3-certbot-nginx fail2ban

# Step 5: Set Up Firewall
echo "🛡️ Configuring firewall..."
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw --force enable

# Step 6: Install Node.js & PM2
echo "⚙️ Installing Node.js & PM2..."
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo bash -
sudo apt install -y nodejs
sudo npm install -g pm2

# Step 7: Clone and Set Up Next.js App
echo "📁 Cloning Next.js app..."
sudo -u $USERNAME git clone $GIT_REPO /home/$USERNAME/$APP_NAME
cd /home/$USERNAME/$APP_NAME
sudo -u $USERNAME npm install
sudo -u $USERNAME npm run build

# Step 8: Start the App with PM2
echo "🚀 Starting Next.js app with PM2..."
pm2 start npm --name "$APP_NAME" -- start
pm2 save
pm2 startup

# Step 9: Configure Nginx
echo "🌐 Setting up Nginx reverse proxy..."
NGINX_CONFIG="/etc/nginx/sites-available/$APP_NAME"
sudo tee $NGINX_CONFIG > /dev/null <<EOF
server {
    listen 80;
    server_name $DOMAIN www.$DOMAIN;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_cache_bypass \$http_upgrade;
    }
}
EOF

sudo ln -s $NGINX_CONFIG /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Step 10: Secure with SSL
echo "🔒 Installing SSL Certificate..."
sudo certbot --nginx -d $DOMAIN -d www.$DOMAIN --email $EMAIL --non-interactive --agree-tos
sudo systemctl reload nginx

# Step 11: Enable Auto Renewal for SSL
echo "🔄 Setting up auto-renew for SSL..."
echo "0 3 * * * root certbot renew --quiet" | sudo tee -a /etc/crontab > /dev/null

# Step 12: Set Up Fail2Ban for Security
echo "🛡️ Configuring Fail2Ban..."
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

echo "🎉 VPS setup complete! Your Next.js app is live at https://$DOMAIN"
```

### How to Run This Script
1. Upload to VPS
   ```bash
   nano setup-vps.sh
   ```
- Paste the script, then press `CTRL + X`, then `Y`, then `ENTER` to save.

2. Make It Executable
   ```bash
   chmod +x setup-vps.sh
   ```

3. Run It
   ```bash
   sudo ./setup-vps.sh
   ```

# From where should i set SSL certificate?

The best place to set up your SSL certificate depends on your setup and security preferences. Here’s a breakdown of your options:

## Option 1: Use Cloudflare's SSL (Recommended)
✅ Best for: Simplicity, no need to install SSL on your VPS

### Steps:
1. Go to Cloudflare Dashboard → Select your domain.
2. Click on "SSL/TLS" from the left sidebar.
3. Select SSL Mode:
- "Full (strict)" → If your VPS already has an SSL certificate installed.
- "Full" → If your VPS doesn’t have a valid SSL certificate.
- "Flexible" → If your VPS has no SSL and you only want encryption between the browser and Cloudflare (not recommended for sensitive data).
4. Enable "Always Use HTTPS" under the "Edge Certificates" tab.

🚀 Pros:
- No need to install an SSL certificate on your VPS.
- Automatic renewal, no manual work required.
- Works instantly.

❌ Cons:
- "Flexible" mode can cause redirect loops.
- If Cloudflare is down, your site might be inaccessible over HTTPS.

## Option 2: Use a Free Let’s Encrypt SSL on Your VPS
✅ Best for: Direct, end-to-end encryption (Best with Cloudflare in "Full (Strict)" mode)

### Steps (For Apache/Nginx):
1. SSH into your VPS:
```sh
ssh root@your-vps-ip
```
2. Install Certbot (Let’s Encrypt SSL Client):
- For Ubuntu/Debian:
```sh
sudo apt update
sudo apt install certbot python3-certbot-apache # For Apache
sudo apt install certbot python3-certbot-nginx # For Nginx
```
3. Run Certbot to Get an SSL Certificate:
- For Apache:
```sh
sudo certbot --apache # For Apache
# sudo certbot --nginx # For Nginx
sudo certbot --nginx -d <YOURDOMAIN>.com -d www.<YOURDOMAIN>.com # For Nginx
```
4. Follow the instructions and select your domain.
5. After installation, Certbot will automatically configure your web server to use the SSL certificate.
6. Restart your web server:
```sh
sudo systemctl restart apache2 # For Apache
sudo systemctl restart nginx # For Nginx
```
7. Set Up Auto-Renewal (SSL expires every 90 days):
```sh
sudo crontab -e
```
Add this line at the bottom to renew SSL automatically:
```
0 3 * * * certbot renew --quiet
```
Or
```bash
sudo certbot renew --dry-run
```

🚀 Pros:
- End-to-end encryption, even if Cloudflare is down.
- Free and widely trusted.

❌ Cons:
- Requires installation and maintenance.

---

## Option 3: Buy an SSL from Hostinger
✅ Best for: If you want a paid SSL and don’t mind managing it

### Steps:
1. Log in to Hostinger → Go to SSL section.
2. Purchase or activate a free SSL if available.
3. Follow Hostinger’s guide to install it on your VPS.

🚀 Pros:
- Support from Hostinger.
- Some premium SSLs offer warranties and extra features.

❌ Cons:
- Costs money.
- Manual setup required.

## Which One Should You Choose?
### Recommended Setup:
1. Install Let’s Encrypt SSL on your VPS (`certbot`).
2. Set Cloudflare to "Full (Strict)" mode to ensure full encryption.

This setup ensures maximum security and end-to-end encryption 🔒.

# Are SSL free?

Yes! SSL certificates can be free depending on where you get them. Here’s a breakdown:

### Free SSL Options
1️⃣ Cloudflare SSL (Easiest)
- Free SSL provided by Cloudflare.
- No need to install anything on your VPS.
- Just enable it in Cloudflare settings.
- Best for simple websites where you don’t need full server-side encryption.

2️⃣ Let’s Encrypt SSL (Best for Full Security)
- Free SSL from Let’s Encrypt (widely trusted).
- Requires installation on your VPS using `certbot`.
- Renews automatically every 90 days.
- Best for full end-to-end encryption when used with Cloudflare Full (Strict) mode.

3️⃣ Hostinger Free SSL (Only for Some Plans)
- Some Hostinger hosting plans include a free SSL.
- If your plan supports it, you can activate it from your Hostinger dashboard.
- Not useful if you’re hosting on a VPS (better to use Let’s Encrypt).

### Paid SSL Options
- Hostinger, GoDaddy, Namecheap, etc. sell SSL certificates ($10–$300 per year).
- Paid SSLs sometimes offer warranties (protection in case of fraud or hacking).
- Not necessary for most websites, unless you need EV SSL (green bar for business validation).

### Recommended for You
Since you have a VPS from InterServer and are using Cloudflare, I recommend:
✅ Let’s Encrypt SSL on VPS + Cloudflare in "Full (Strict)" mode
This gives you 100% free and strong encryption.

# NginX config file for SSL

```bash
# Redirect non-www to www (Port 80 - HTTP)
server {
    listen 80;
    server_name madusankadissanayake.com;
   
    # Redirect non-www to www
    return 301 https://www.madusankadissanayake.com$request_uri;

}

# Redirect non-www to www (Port 443 - HTTPS)
server {
    listen 443 ssl http2;
    server_name madusankadissanayake.com;

    ssl_certificate /etc/letsencrypt/live/madusankadissanayake.com/fullchain.pem; # Your SSL certificate path
    ssl_certificate_key /etc/letsencrypt/live/madusankadissanayake.com/privkey.pem; # Your SSL private key path

    return 301 https://www.madusankadissanayake.com$request_uri;
}

# Main HTTPS block for www.madusankadissanayake.com
# HTTPS server block (Port 443)
server {
    listen 443 ssl http2;
    server_name www.madusankadissanayake.com;

    # SSL settings
    ssl_certificate /etc/letsencrypt/live/madusankadissanayake.com/fullchain.pem; # Your SSL certificate path
    ssl_certificate_key /etc/letsencrypt/live/madusankadissanayake.com/privkey.pem; # Your SSL private key path

    # SSL protocols and ciphers (adjust based on your preferences)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;

    # Enable SSL session cache and settings
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    # Define document root and other settings
    root /var/www/port-folio/.next/;

    location /_next/static/ {
        alias /var/www/port-folio/.next/static/;
        expires 1y;
        access_log off;
        add_header Cache-Control "public, max-age=31536000, immutable";
    }

    location / {
        proxy_pass http://localhost:3000; # Make sure your Next.js app runs on this port
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
   
    location /test {
        alias /var/www/port-folio/;
        index index.html;
    }


    # Enable Gzip compression for performance improvements
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_vary on;

    # Additional SSL headers (security best practices)
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Logging
    access_log /var/log/nginx/madusankadissanayake.com.access.log;
    error_log /var/log/nginx/madusankadissanayake.com.error.log;
}
```
- Check logs for errors

```bash
sudo tail -f /var/log/nginx/<YOURDOMAIN>.com.error.log
```
**Verify HTTP/2:** After reloading NGINX, verify that HTTP/2 is working correctly using:

```bash
curl -I https://<YOURDOMAIN>.com
```

### What’s Next?
- Push updates easily: SSH into the server and pull the latest code:
  ```bash
  cd /home/deployuser/next-app
  git pull origin main
  npm install
  npm run build
  pm2 restart next-app
  ```

- Automate deployments: Set up a GitHub webhook or GitHub Actions for auto-deploy.

🚀 Done! Your Next.js app is running securely on your VPS.

# Additional Considerations

### Optimize Next.js Performance
- Enable Image Optimization by using a CDN like Cloudflare.
- Enable caching with Nginx or a caching service like Redis.
- Run Next.js in production mode (`npm run start` instead of `next dev`).

### Monitor Server Performance
- Install htop to monitor CPU and RAM usage:
  ```bash
  sudo apt install htop -y
  htop
  ```
- Use PM2 logs for debugging:
  ```bash
  pm2 logs next-app
  ```

### Enable Automatic Deployment
- Set up GitHub Actions to SSH into the server and deploy updates automatically.
- Alternatively, use webhooks to trigger a `git pull` and restart the app.

### Set Up Backups
- Regularly backup your VPS using provider snapshots.
- Store important files offsite (e.g., AWS S3, Google Drive).

### DDoS & Security Protection
- Use Cloudflare to protect against DDoS and manage DNS.
- Harden SSH security (disable root login, use SSH keys only).
- Enable fail2ban for brute-force protection.

### Database Setup (If Needed)
- If using a database (e.g., PostgreSQL, MySQL, MongoDB):
- Install and configure it securely.
- Use remote database hosting (like Supabase, PlanetScale, or MongoDB Atlas).

## Summary
If your setup is simple, the original guide is all you need. However, for better security, performance, and automation, consider implementing monitoring, auto-deployment, Cloudflare, and backups.

# jenkins for deployments ?

it's highly recommended to use a separate VPS for Jenkins rather than running it on the same server as your Next.js app. Here’s why:  

## Why Use a Separate VPS for Jenkins?

### Security Isolation  
- Jenkins requires admin/root-level access to deploy applications, making it a high-value target for attacks.  
- If Jenkins gets compromised, an attacker could gain access to your Next.js server as well.  

### Performance Stability  
- Jenkins runs resource-heavy jobs (e.g., builds, tests, deployments).  
- Running it on the same server as Next.js could cause CPU spikes, slow performance, or even crashes.  

### Easier Maintenance & Scaling  
- Keeping Jenkins separate allows you to scale or migrate the CI/CD setup independently.  
- If Jenkins needs updates, it won’t affect your production app.  

### Dedicated Storage for Build Artifacts  
- Jenkins needs storage for logs, artifacts, and pipelines.  
- Keeping it separate prevents disk space issues on your main app server.  

## Recommended Setup  

### Option 1: Use a Small VPS for Jenkins  
- Choose a low-cost VPS (e.g., 2 vCPUs, 4GB RAM).  
- Keep Jenkins firewalled and accessible only via VPN or SSH keys.  
- Use a reverse proxy (Nginx) with SSL to secure the Jenkins UI.  

### Option 2: Use a Cloud-Based CI/CD Service  
Instead of hosting Jenkins, consider GitHub Actions, GitLab CI/CD, or CircleCI for easier management.  

### Option 3: Use a Containerized Jenkins (Docker/Kubernetes)  
- Run Jenkins inside Docker to isolate it from the main system.  
- Deploy Jenkins inside Kubernetes for better scalability.  

## Summary  
- Best Practice: Use a separate VPS for Jenkins to ensure security, performance, and maintainability.  
- Alternative: Use a cloud-based CI/CD service to avoid maintenance overhead.  
- Security Tip: Keep Jenkins behind a firewall, with limited SSH access, and enable authentication.  

Want help setting up Jenkins on a separate VPS? 🚀

# Map Hostinger domain -> Cloudflare -> InterServer VPS

## Add Your Domain to Cloudflare
1. Log in to Cloudflare ([https://dash.cloudflare.com/](https://dash.cloudflare.com/)).
2. Click "Add a Site" and enter your domain (e.g., `<YOURDOMAIN>.com`).
3. Click "Continue" and select the Free Plan (or another plan if needed).
4. Cloudflare will scan existing DNS records. Click "Continue".

## Change Hostinger Nameservers to Cloudflare
1. Go to Hostinger ([https://www.hostinger.com/](https://www.hostinger.com/)) and log in.
2. Open the Domains section and select your domain.
3. Look for the option "Change Nameservers".
4. Replace the current nameservers with the Cloudflare nameservers provided to you (usually something like `ns1.cloudflare.com` and `ns2.cloudflare.com`).
5. Click "Save" or "Update".

> Note: It may take a few hours (up to 24 hours) for the changes to propagate.

## Add A and CNAME Records in Cloudflare
Now, you need to point your domain to your InterServer VPS.

1. In Cloudflare, go to DNS settings.
2. Click "Add Record" and choose:
   - Type: `A`
   - Name: `@` (or `<YOURDOMAIN>.com`)
   - IPv4 Address: Enter your InterServer VPS IP address
   - TTL: Auto
   - Proxy status: DNS Only (Grey Cloud) for now, later you can enable the proxy.
   - Click Save.

3. Add a CNAME Record for `www`:
   - Type: `CNAME`
   - Name: `www`
   - Target: `<YOURDOMAIN>.com`
   - TTL: Auto
   - Proxy Status: DNS Only (Grey Cloud).
   - Click Save.

## Configure SSL in Cloudflare
To make your website secure with HTTPS:

1. In Cloudflare, go to SSL/TLS.
2. Choose "Full (strict)" mode if your VPS has an SSL certificate.
   - If you don’t have an SSL certificate installed on your VPS, select "Flexible" mode.
3. Enable "Always Use HTTPS".

## Update Your VPS Web Server (Apache/Nginx)
On your InterServer VPS, ensure that your web server is properly configured:

- If using Apache, update the Virtual Host configuration:
  ```sh
  sudo nano /etc/apache2/sites-available/<YOURDOMAIN>.com.conf
  ```
  Add:
  ```
  <VirtualHost *:80>
      ServerName <YOURDOMAIN>.com
      ServerAlias www.<YOURDOMAIN>.com
      DocumentRoot /var/www/html
  </VirtualHost>
  ```

- If using Nginx, edit the configuration file:
  ```sh
  sudo nano /etc/nginx/sites-available/<YOURDOMAIN>.com
  ```
  Add:
  ```
  server {
      listen 80;
      server_name <YOURDOMAIN>.com www.<YOURDOMAIN>.com;
      root /var/www/html;
      index index.html index.php;
  }
  ```
- Restart the web server:
  ```sh
  sudo systemctl restart apache2
  # or
  sudo systemctl restart nginx
  ```

## Verify Everything
1. Wait for DNS propagation (can take a few hours).
2. Visit `<YOURDOMAIN>.com` in a browser.
3. Check your Cloudflare DNS settings for errors.

## Final Notes
- If you face any issues, check your VPS firewall and allow ports 80 (HTTP) and 443 (HTTPS):
  ```sh
  sudo ufw allow 80
  sudo ufw allow 443
  sudo ufw enable
  ```
- You can enable Cloudflare Proxy (Orange Cloud) after confirming the site works.

