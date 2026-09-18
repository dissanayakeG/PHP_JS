# Guide: Host a Flutter Web App on an Existing VPS Using a Subdomain

This setup assumes:

* Existing domain: `abc.com`
* Existing Next.js app is already hosted on the VPS, for example at `abc.com`
* New Flutter Web app: `sd.abc.com`
* Nginx is already installed and serving the Next.js application
* Domain is registered with Hostinger
* DNS is managed by Cloudflare
* Deployment will eventually use GitHub Actions
* Flutter Web will be served as static files by Nginx

The Flutter application remains completely separate from the existing Next.js application.

---

## Architecture

```text
                        Internet
                           │
                           ▼
                    ┌─────────────┐
                    │ Cloudflare  │
                    │ DNS + Proxy │
                    └──────┬──────┘
                           │
                           ▼
                         VPS
                           │
                        Nginx
                     ┌─────┴─────┐
                     │           │
                     ▼           ▼
                 abc.com     sd.abc.com
                     │           │
                  Next.js     Static files
                               │
                               ▼
                     /var/www/sd.abc.com
```

Nginx decides which application to serve based on the hostname requested by the browser.

---

# 1. Configure Cloudflare DNS

Do this **before requesting the SSL certificate**.

Since Cloudflare manages the DNS, you normally do not need to make additional DNS changes at Hostinger. Hostinger remains the domain registrar, while the domain's nameservers point to Cloudflare.

In Cloudflare:

**DNS → Records → Add record**

Configure:

| Setting      | Value             |
| ------------ | ----------------- |
| Type         | `A`               |
| Name         | `sd`              |
| IPv4 address | `<VPS_PUBLIC_IP>` |
| Proxy status | Proxied           |
| TTL          | Auto              |

This results in:

```text
sd.abc.com → VPS_PUBLIC_IP
```

The existing `abc.com` record does not need to change.

### Verify DNS

From your machine:

```bash
nslookup sd.abc.com
```

or:

```bash
dig sd.abc.com
```

With Cloudflare proxy enabled, seeing **Cloudflare IP addresses instead of your VPS IP is normal**.

---

# 2. Check Cloudflare SSL Mode

Go to:

**Cloudflare → SSL/TLS → Overview**

Use:

```text
Full (strict)
```

This is the preferred configuration once the VPS has a valid Let's Encrypt certificate.

Avoid:

```text
Flexible
```

Flexible SSL means:

```text
Browser ──HTTPS──> Cloudflare ──HTTP──> VPS
```

With Full (strict):

```text
Browser ──HTTPS──> Cloudflare ──HTTPS──> VPS
```

This avoids common redirect-loop and origin-security problems.

---

# 3. Create a Directory for Flutter

SSH into the VPS:

```bash
ssh user@VPS_IP
```

Create a dedicated directory:

```bash
sudo mkdir -p /var/www/sd.abc.com
```

Initially set appropriate ownership/permissions:

```bash
sudo chown -R www-data:www-data /var/www/sd.abc.com
sudo chmod -R 755 /var/www/sd.abc.com
```

The directory will eventually contain files similar to:

```text
/var/www/sd.abc.com/
├── index.html
├── flutter.js
├── flutter_bootstrap.js
├── main.dart.js
├── manifest.json
├── assets/
└── icons/
```

Do **not** place the Flutter project source code here. Only the output from:

```bash
flutter build web --release
```

needs to be deployed.

---

# 4. Create a Separate Nginx Configuration

Do not modify the existing Next.js server block unnecessarily.

Create:

```bash
sudo nano /etc/nginx/sites-available/sd.abc.com
```

Add:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name sd.abc.com;

    root /var/www/sd.abc.com;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Save and exit.

### Why `try_files` matters

Flutter Web commonly uses client-side routing.

For example:

```text
sd.abc.com/
sd.abc.com/loans
sd.abc.com/settings
```

Nginx doesn't actually have files named `/loans` or `/settings`.

Therefore:

```nginx
try_files $uri $uri/ /index.html;
```

makes Nginx return `index.html`, after which Flutter's router handles the route.

Without this, refreshing something like:

```text
sd.abc.com/loans
```

can result in an Nginx `404`.

---

# 5. Enable the Nginx Site

Create the symbolic link:

```bash
sudo ln -s \
  /etc/nginx/sites-available/sd.abc.com \
  /etc/nginx/sites-enabled/sd.abc.com
```

Check the entire Nginx configuration:

```bash
sudo nginx -t
```

You should get something similar to:

```text
syntax is ok
test is successful
```

If the test fails, **do not reload/restart Nginx**. Fix the configuration first.

If successful:

```bash
sudo systemctl reload nginx
```

`reload` is preferable to `restart` here because Nginx can apply the new configuration gracefully without unnecessarily stopping the existing service.

Your existing Next.js application should continue running normally.

---

# 6. Initial Flutter Deployment

From the Flutter project:

```bash
flutter pub get
flutter build web --release
```

The production application will be generated under:

```text
build/web/
```

For the first deployment, you can upload it manually.

For example:

```bash
scp -r build/web/* user@VPS_IP:/var/www/sd.abc.com/
```

However, if your deployment SSH user cannot write to `/var/www/sd.abc.com`, don't solve that by broadly opening permissions such as `chmod 777`.

A better long-term solution is to configure the deployment user's ownership/group permissions appropriately before setting up GitHub Actions.

---

# 7. Test HTTP Before SSL

At this stage, test:

```text
http://sd.abc.com
```

Also verify directly on the VPS:

```bash
curl -I http://localhost -H "Host: sd.abc.com"
```

You should receive a successful HTTP response.

If Flutter loads correctly, proceed with HTTPS.

---

# 8. Install Certbot if Necessary

Check whether Certbot already exists:

```bash
certbot --version
```

If it is already being used for the Next.js domain, you can use the existing installation.

Otherwise, install Certbot according to your VPS distribution's supported installation method.

You can inspect existing certificates with:

```bash
sudo certbot certificates
```

---

# 9. Generate the SSL Certificate

Run:

```bash
sudo certbot --nginx -d sd.abc.com
```

Certbot will:

```text
Let's Encrypt
     │
     ▼
Validate sd.abc.com
     │
     ▼
Issue certificate
     │
     ▼
Update Nginx
```

After completion:

```bash
sudo nginx -t
```

Then:

```bash
sudo systemctl reload nginx
```

Now test:

```text
https://sd.abc.com
```

### Cloudflare proxy consideration

Cloudflare's proxied DNS can work with Let's Encrypt/Certbot, but if HTTP validation fails because of proxy/security rules, temporarily change the `sd` DNS record from **Proxied** to **DNS only**, issue the certificate, then turn **Proxied** back on.

Do not disable the proxy unless necessary.

---

# 10. Verify Automatic Certificate Renewal

Certbot normally configures automatic renewal.

Test it:

```bash
sudo certbot renew --dry-run
```

You should not need to manually renew the certificate every few months.

---

# 11. Configure GitHub Actions Deployment

Once the infrastructure works, automate Flutter deployments.

The desired pipeline is:

```text
git push main
      │
      ▼
GitHub Actions
      │
      ├── Checkout
      ├── Install Flutter
      ├── flutter pub get
      ├── flutter build web --release
      │
      ▼
Upload build/web/
      │
      ▼
/var/www/sd.abc.com
      │
      ▼
Nginx serves new version
```

No Flutter SDK is required on the VPS.

No Node.js process, PM2 process, Docker container, or application server is required for a normal Flutter Web build. Nginx simply serves the generated static files.

---

# 12. Configure SSH Deployment Access

Prefer a **dedicated deployment SSH key** rather than putting your normal personal SSH private key into GitHub.

Generate one locally:

```bash
ssh-keygen -t ed25519 -C "github-actions-flutter-deploy"
```

Add the **public key** to the VPS user's:

```text
~/.ssh/authorized_keys
```

Put the **private key** into GitHub Actions secrets.

Go to:

**GitHub repository → Settings → Secrets and variables → Actions**

Create:

```text
VPS_HOST
VPS_USER
VPS_SSH_KEY
```

For example:

```text
VPS_HOST = 123.123.123.123

VPS_USER = deploy

VPS_SSH_KEY = -----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
```

The deployment user must have permission to update:

```text
/var/www/sd.abc.com
```

---

# 13. Create the GitHub Actions Workflow

Create:

```text
.github/workflows/deploy.yml
```

Example:

```yaml
name: Deploy Flutter Web

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          channel: stable
          cache: true

      - name: Install dependencies
        run: flutter pub get

      - name: Build Flutter Web
        run: flutter build web --release

      - name: Deploy to VPS
        uses: appleboy/scp-action@v1
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          source: "build/web/*"
          target: "/var/www/sd.abc.com"
          strip_components: 2
```

A Java setup step generally isn't necessary merely to build a standard Flutter Web application, so it can be omitted unless something in your project's dependencies specifically requires it.

For a production pipeline, I would also **pin third-party GitHub Actions to a specific release/tag rather than `@master`**.

---

# 14. Important Deployment Improvement

There is one weakness in the original deployment approach.

Simply copying:

```text
build/web/*
```

over the previous deployment can leave **obsolete files** from older builds on the server.

For example:

```text
Old build:
assets/old-file.js

New build:
assets/new-file.js
```

`scp` adds/replaces files but doesn't necessarily remove `old-file.js`.

A cleaner deployment strategy is to upload a release directory and then replace/synchronize the web root, or use `rsync --delete` over SSH.

For a small personal Flutter app, SCP may still work adequately. For a production application, a synchronized or atomic deployment strategy is preferable.

---

# 15. Optional Nginx Static Asset Caching

Once everything works correctly, you can improve static asset delivery.

For example:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name sd.abc.com;

    root /var/www/sd.abc.com;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /assets/ {
        try_files $uri =404;
        expires 7d;
        add_header Cache-Control "public";
    }
}
```

Be conservative with caching `index.html`; you generally want browsers to discover new Flutter deployments rather than retaining an old application entry point.

Cloudflare can provide additional caching in front of Nginx.

---

# 16. Final Configuration

The resulting infrastructure looks like:

```text
Hostinger
└── Domain registration: abc.com
        │
        ▼
Cloudflare Nameservers
        │
        ├── abc.com
        │      │
        │      ▼
        │    VPS
        │      │
        │      └── Nginx → Existing Next.js app
        │
        └── sd.abc.com
               │
               ▼
             VPS
               │
               └── Nginx
                    │
                    ▼
             /var/www/sd.abc.com
                    │
                    ▼
               Flutter Web
```

Deployment becomes:

```text
Developer
   │
 git push
   │
   ▼
GitHub
   │
   ▼
GitHub Actions
   │
   ├── flutter pub get
   ├── flutter build web --release
   └── deploy
          │
          ▼
       VPS
          │
          ▼
 /var/www/sd.abc.com
          │
          ▼
        Nginx
          │
          ▼
 https://sd.abc.com
```

## Recommended order

1. **Cloudflare:** create `sd` A record → VPS IP.
2. **Cloudflare:** confirm SSL mode is `Full (strict)`.
3. **VPS:** create `/var/www/sd.abc.com`.
4. **Nginx:** create a separate `sd.abc.com` server block.
5. Run `sudo nginx -t`.
6. Reload Nginx.
7. Build and deploy Flutter once manually.
8. Verify `http://sd.abc.com`.
9. Run `sudo certbot --nginx -d sd.abc.com`.
10. Verify `https://sd.abc.com`.
11. Run `sudo certbot renew --dry-run`.
12. Configure a dedicated deployment SSH key.
13. Add GitHub repository secrets.
14. Add the GitHub Actions workflow.
15. Push to `main` and verify automatic deployment.

This keeps the **Next.js runtime and Flutter static application isolated at the Nginx virtual-host level**, so adding the Flutter subdomain does not require changing how the existing Next.js application runs.
