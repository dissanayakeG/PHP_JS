# Securely deploy a Next.js app on an Ubuntu/Debian VPS

This guide deploys a server-rendered Next.js application behind Nginx, runs it with PM2, and obtains a Let's Encrypt certificate. It assumes a fresh Ubuntu or Debian VPS, a domain you control, and a Git repository that contains `build` and `start` scripts (normally `next build` and `next start`).

> Keep your current root SSH session open until you have successfully signed in as the new user with an SSH key. Otherwise, a mistake in SSH configuration can lock you out of the server.

Replace these values throughout:

- `deploy` — the non-root Linux user that will own and run the app.
- `example.com` — your domain.
- `my-next-app` — the app and PM2 process name.
- `https://github.com/your-account/your-repository.git` — your repository URL.

## 1. Connect and update the VPS

Connect using the credentials supplied by your VPS provider:

```bash
ssh root@your-server-ip
```

On the first connection, verify the SSH host-key fingerprint against the one shown in your VPS provider's console before accepting it. This prevents accepting an unexpected server.

Update the server, then reboot if the update requests it:

```bash
apt update
apt upgrade -y
reboot
```

Reconnect after the reboot.

## 2. Create a non-root deployment user and configure SSH keys

Create the account and grant it administrative access:

```bash
adduser deploy
usermod -aG sudo deploy
```

On your **local computer**, create a key if you do not already have one:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

Copy its public key to the VPS (use the provider's initial authentication method if password login is not available):

```bash
ssh-copy-id deploy@your-server-ip
```

Open a **second terminal** and confirm that key-based login and sudo work before changing SSH settings:

```bash
ssh deploy@your-server-ip
sudo -v
```

## 3. Configure the firewall and harden SSH

Allow SSH first, then enable the firewall. If you use a non-default SSH port, allow that port instead of `OpenSSH`.

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
```

Create a separate SSH configuration drop-in rather than editing the packaged file:

```bash
sudo nano /etc/ssh/sshd_config.d/99-hardening.conf
```

Add the following only after confirming the new user's SSH key works:

```text
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
```

Validate the configuration, then reload the SSH service. Do not close your working sessions until you have tested another login.

```bash
sudo sshd -t
sudo systemctl reload ssh
```

Optional but recommended: enable automatic security updates.

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## 4. Install system packages, Node.js, and PM2

Log in as the deployment user, then install the required packages:

```bash
ssh deploy@your-server-ip
sudo apt update
sudo apt install -y ca-certificates curl git build-essential nginx
```

Install the current Node.js LTS release. The following uses NodeSource's LTS channel; inspect its script before running it if your organisation requires reviewed package sources.

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x -o /tmp/nodesource_setup.sh
less /tmp/nodesource_setup.sh
sudo -E bash /tmp/nodesource_setup.sh
sudo apt install -y nodejs
node --version
npm --version
```

Ensure the installed version satisfies the `engines` field in your project's `package.json`. Install PM2 once, using the same Node.js installation that will run the app:

```bash
sudo npm install -g pm2
pm2 --version
```

## 5. Clone, configure, build, and start the app

Create an application directory owned by the deployment user, then clone the repository:

```bash
sudo mkdir -p /var/www
sudo chown deploy:deploy /var/www
cd /var/www
git clone https://github.com/your-account/your-repository.git my-next-app
cd /var/www/my-next-app
```

Create the production environment file required by your application. Never commit credentials or copy a development `.env` file containing local values. Variables beginning with `NEXT_PUBLIC_` are embedded in the browser bundle and must not contain secrets.

```bash
nano .env.production
chmod 600 .env.production
```

Install exactly the dependency versions captured in the lock file, build, and start the production server on loopback only:

```bash
npm ci
npm run build
PORT=3000 pm2 start npm --name my-next-app -- start -- -H 127.0.0.1
pm2 status
curl -I http://127.0.0.1:3000
```

Configure PM2 to restore this user's process list at boot. `pm2 startup` prints a `sudo ...` command; copy and run the exact command it displays, then save the process list.

```bash
pm2 startup
pm2 save
```

Do not use `pm2 stop all`, `pm2 delete all`, or `pm2 kill` as part of normal deployment: those commands also affect any other applications owned by this user.

## 6. Configure Nginx as the reverse proxy

Create the virtual-host configuration:

```bash
sudo nano /etc/nginx/sites-available/example.com
```

Paste this configuration, replacing `example.com` with your domain. It proxies to the local-only Next.js process, preserves the original host and client IP, and supports WebSocket upgrades.

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;

    client_max_body_size 10m;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

Enable the site, remove the default site if it is still enabled, and validate before reloading Nginx:

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

Before requesting a certificate, create DNS `A` (and, if applicable, `AAAA`) records for `example.com` and `www.example.com` pointing to this VPS. Wait for DNS propagation, then check that the site responds on HTTP:

```bash
curl -I http://example.com
```

If you do not use `www.example.com`, remove it from both `server_name` and the certificate command below.

## 7. Add HTTPS with Let's Encrypt

Install Certbot and its Nginx plugin:

```bash
sudo apt install -y certbot python3-certbot-nginx
```

Request the certificate and let Certbot add the HTTPS server block and redirect:

```bash
sudo certbot --nginx -d example.com -d www.example.com --redirect
```

Certificates expire every 90 days. On supported Ubuntu/Debian packages, Certbot installs a systemd renewal timer; confirm it and perform a safe renewal test:

```bash
sudo systemctl list-timers | grep certbot
sudo certbot renew --dry-run
```

## 8. Deploy later updates safely

For each release, run the following as `deploy` from the application directory. `npm ci` keeps installs reproducible, and the build must succeed before the running process is restarted.

```bash
cd /var/www/my-next-app
git fetch origin
git switch main
git pull --ff-only origin main
npm ci
npm run build
pm2 reload my-next-app --update-env
pm2 save
```

For automated deployments, use a CI workflow that connects with a dedicated SSH key and runs the same commands. Do not expose an unauthenticated webhook that executes shell commands on the VPS.

## 9. Verify and operate the deployment

```bash
curl -I https://example.com
pm2 status
pm2 logs my-next-app
sudo journalctl -u nginx -f
sudo tail -f /var/log/nginx/error.log
```

Useful follow-up hardening:

- Install and configure Fail2ban only after reviewing its SSH jail settings: `sudo apt install -y fail2ban`.
- Remove or disable services you have confirmed are unused; do not blindly disable Apache if another site depends on it.
- Apply regular OS, Node.js, and application dependency updates, and keep verified VPS backups.
- Use the VPS provider's firewall as an additional network layer where available.

## References

- [Next.js self-hosting guide](https://nextjs.org/docs/app/guides/self-hosting)
- [PM2 startup-script documentation](https://pm2.keymetrics.io/docs/usage/startup/)
- [Certbot Nginx instructions](https://certbot.eff.org/instructions?ws=nginx)
- [NodeSource Debian/Ubuntu installation guide](https://github.com/nodesource/distributions/blob/master/DEV_README.md)

✅ Done! Your Next.js app is now securely hosted on a VPS.

# After deployment: beginner questions and next steps

Your application is running. This section explains what to do next without repeating the VPS setup above.

## Start here: choose your setup

Use this path for most projects:

1. Keep the Let's Encrypt certificate created above.
2. Keep the Nginx configuration created above; do not add a second manual SSL configuration.
3. If you use Cloudflare for DNS, set its encryption mode to **Full (strict)** after the origin certificate works.
4. Test the site, then set up backups and a repeatable deployment process.

A paid certificate is usually unnecessary. Let's Encrypt is free, trusted by modern browsers, and automatically renewable. The certificate from Cloudflare protects the connection between a visitor and Cloudflare; it does **not** replace a valid certificate on your VPS when you want end-to-end encryption.

## Do I need Cloudflare?

No. The site works with DNS managed by your registrar and the Let's Encrypt certificate already installed on the VPS.

Cloudflare is optional. It can manage DNS and add CDN, DDoS mitigation, caching, and web-application-firewall features. Use it only if those benefits suit your site.

### Option A — no Cloudflare

At your domain registrar or DNS host, create these records:

| Host | Type | Value | Use it for |
| --- | --- | --- | --- |
| `@` | `A` | Your VPS IPv4 address | `example.com` |
| `www` | `CNAME` | `example.com` | `www.example.com` |

Add an `AAAA` record only when your VPS has a working IPv6 address and Nginx is reachable on IPv6. DNS changes can take time to propagate.

### Option B — use Cloudflare

1. Add the domain to Cloudflare and copy the two Cloudflare nameservers it shows.
2. At your registrar, replace the current nameservers with those Cloudflare nameservers.
3. In Cloudflare **DNS**, create the `A` record for `@` and the `CNAME` record for `www` shown above.
4. Leave both records **DNS only** (grey cloud) while you first test the VPS and obtain or renew the Let's Encrypt certificate.
5. After `https://example.com` works, you may enable the **Proxied** (orange cloud) setting for web records. Do not proxy non-web services such as mail records.

> DNS only still serves your website normally. It simply connects visitors directly to the VPS rather than routing them through Cloudflare.

## Which Cloudflare SSL/TLS setting should I choose?

For this guide, choose **Full (strict)** once your Let's Encrypt certificate is valid:

~~~text
Visitor ── HTTPS ──> Cloudflare ── HTTPS, certificate verified ──> Your VPS
~~~

In the Cloudflare dashboard, open **SSL/TLS → Overview** and select **Full (strict)**.

Avoid **Flexible** for this deployment. It encrypts only the visitor-to-Cloudflare connection and uses HTTP from Cloudflare to your VPS. Because the Nginx setup above redirects HTTP to HTTPS, Flexible can also produce an `ERR_TOO_MANY_REDIRECTS` loop.

Use **Full** only as a short-lived troubleshooting option when the VPS has HTTPS but its certificate cannot yet be validated. Return to Full (strict) as soon as the certificate is valid.

### Should I enable “Always Use HTTPS” in Cloudflare?

The Certbot command in the main guide used `--redirect`, so Nginx already redirects HTTP to HTTPS. Keep that configuration and do not add another redirect unless you have a reason to manage redirects at Cloudflare instead.

If you later move the redirect to Cloudflare, enable **SSL/TLS → Edge Certificates → Always Use HTTPS** and remove the duplicate origin redirect. Test in a private browser window after changing either setting.

## SSL certificate questions

### Is SSL free?

Yes. Let's Encrypt certificates are free and suitable for most personal and business websites. Their short lifetime is intentional; Certbot renews them automatically.

### Do I need to create a cron job?

Usually no. The Ubuntu/Debian Certbot package normally installs a systemd timer. Confirm it and perform a test renewal:

~~~bash
sudo systemctl list-timers | grep certbot
sudo certbot renew --dry-run
~~~

Do not add a cron job unless your system has no Certbot timer and you have confirmed how Certbot was installed.

### Where are the certificate files?

For `example.com`, Certbot stores them here:

~~~text
/etc/letsencrypt/live/example.com/fullchain.pem
/etc/letsencrypt/live/example.com/privkey.pem
~~~

Do not move, edit, commit, or share `privkey.pem`. Certbot and the Nginx plugin manage these files and the web-server configuration for you.

### Why should I not copy a separate Nginx SSL block?

The main guide already gives Certbot control of the Nginx configuration. Adding another certificate block can create duplicate `server_name` entries, conflicting redirects, and renewal problems. Only write a custom HTTPS configuration when you understand the existing Certbot-generated configuration and have a specific need.

## Pick one canonical domain

Choose whether visitors should use `example.com` or `www.example.com` as the primary address. Both names need DNS records and must be included in the certificate request if you want both to work.

The main guide allows both names. To redirect one to the other, add a small dedicated Nginx server block after the certificate is working, test it with `sudo nginx -t`, then reload Nginx. Do not make this change while Cloudflare is in Flexible mode.

## Updating the application

For a manual release, use the deployment commands from **8. Deploy later updates safely** above:

~~~bash
cd /var/www/my-next-app
git fetch origin
git switch main
git pull --ff-only origin main
npm ci
npm run build
pm2 reload my-next-app --update-env
pm2 save
~~~

If any command fails, stop and fix it before continuing. In particular, do not restart PM2 after a failed build.

For automatic releases, use a dedicated deployment SSH key and a CI workflow rather than exposing a public webhook that runs shell commands. See [Set up GitHub Actions](setup-github-action.md) for the related workflow guide.

## Everyday monitoring

Use these commands when checking the app or investigating an error:

| What you need | Command |
| --- | --- |
| Is the app process online? | `pm2 status` |
| Application logs | `pm2 logs my-next-app` |
| Nginx service logs | `sudo journalctl -u nginx -f` |
| Nginx errors | `sudo tail -f /var/log/nginx/error.log` |
| Disk and memory overview | `df -h` and `free -h` |
| Live CPU/RAM view (optional) | `sudo apt install -y htop` then `htop` |

Press `Ctrl+C` to stop following a log. If the site responds with a `502 Bad Gateway`, check `pm2 status` and the PM2 logs first; it normally means the Next.js process is stopped, crashing, or listening on a different port.

## Fail2ban: optional SSH protection

Fail2ban watches logs and temporarily blocks repeated failed login attempts. It is useful on a public VPS, but it is not a replacement for SSH keys, a firewall, and updates.

Install and enable it only after you have confirmed SSH-key access:

~~~bash
sudo apt install -y fail2ban
sudo systemctl enable --now fail2ban
sudo systemctl status fail2ban
~~~

Check the SSH jail status:

~~~bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
~~~

Keep a second SSH session open when changing access controls. If you are banned by mistake, use the VPS provider's web console to regain access.

## Backups and recovery

A Git repository is not a complete backup. It usually does not contain production environment files, uploaded user content, databases, or VPS configuration.

At minimum:

1. Enable scheduled VPS snapshots with your hosting provider.
2. Back up databases according to their database engine and store encrypted copies off the VPS.
3. Back up user uploads and the information needed to recreate `.env.production` securely.
4. Record your DNS records and keep deployment keys in a password manager or secret manager.
5. Test restoring a backup. A backup you have never restored is not yet proven.

## Database and environment-variable safety

Keep database credentials and API keys in `.env.production` with restrictive permissions. Never expose secret values through a variable beginning with `NEXT_PUBLIC_`: Next.js includes those values in the browser bundle during the build.

For a database running on another server, restrict its firewall to the VPS IP address where possible. For a database on the same VPS, bind it to localhost unless another trusted service genuinely needs network access.

## Troubleshooting

| Symptom | Check first | Typical fix |
| --- | --- | --- |
| Domain opens the wrong site or does not resolve | DNS records and propagation | Confirm the `A` and `CNAME` values and wait for DNS propagation. |
| `502 Bad Gateway` | `pm2 status` and `pm2 logs my-next-app` | Start/restart the named app after fixing its error; confirm it listens on port 3000. |
| Certificate request fails | DNS, ports 80/443, and Nginx | Make sure the domain reaches this VPS and `sudo nginx -t` succeeds. |
| `ERR_TOO_MANY_REDIRECTS` | Cloudflare SSL/TLS mode | Use Full (strict), not Flexible; make sure only one layer owns the HTTP-to-HTTPS redirect. |
| Cloudflare shows error 526 | VPS certificate | Renew or fix the origin certificate, then use Full (strict). |
| Site changed but visitors see old content | Build and browser cache | Confirm `npm run build` succeeded; test in a private window and clear CDN cache only when necessary. |
| SSH access stopped working | A second SSH session or provider console | Check `sudo sshd -t`, firewall rules, and the deployment user's authorized key. |

## Recommended maintenance routine

- **Weekly:** review PM2/Nginx errors and free disk space.
- **Monthly:** apply system updates, update application dependencies after testing, and run `sudo certbot renew --dry-run`.
- **Before major changes:** create a VPS snapshot or verified backup.
- **After changing Nginx:** always run `sudo nginx -t` before `sudo systemctl reload nginx`.

## Further reading

- [Cloudflare: Full (strict) encryption](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/full-strict/)
- [Cloudflare: Always Use HTTPS](https://developers.cloudflare.com/ssl/edge-certificates/additional-options/always-use-https/)
- [Cloudflare: DNS records](https://developers.cloudflare.com/dns/manage-dns-records/how-to/create-dns-records/)
- [Certbot documentation](https://certbot.eff.org/)
