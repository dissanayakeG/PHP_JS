An in-depth understanding of each of these topics.

# 1) Reverse Proxy

## What is it?

A reverse proxy is a server that sits between client devices (like browsers or apps) and the backend servers. Instead of clients directly accessing your web server, they send requests to the reverse proxy, which then forwards those requests to the appropriate backend server.

## Why Should You Install It?

 Security: It hides your actual web server's IP, making it harder for attackers to target.
 Load Balancing: Can distribute traffic across multiple backend servers.
 SSL Termination: Can handle SSL encryption/decryption to reduce the load on backend servers.
 Caching: Can store copies of static content to improve response times.
 Compression: Reduces the size of responses to improve performance.

## Examples of Reverse Proxy Servers

 Nginx (Most popular, lightweight, and efficient)
 Apache HTTP Server (mod\_proxy)
 HAProxy (High-performance proxy, mostly used for load balancing)
 Traefik (Used in containerized environments)

## How Does It Work?

1. A user requests `https://yourwebsite.com`
2. The request goes to the reverse proxy.
3. The reverse proxy checks its configuration and forwards the request to the correct backend server.
4. The backend server processes the request and sends a response.
5. The reverse proxy receives the response and forwards it to the user.

# 2) SSL Certificate

## Why Do You Need It?

 Encryption: Protects sensitive data (like login credentials, payment details) during transmission.
 Authentication: Ensures the user is communicating with the correct server and not a malicious one.
 SEO Benefits: Google ranks HTTPS websites higher than HTTP ones.
 User Trust: Browsers mark non-HTTPS sites as "Not Secure."

## Where Should You Install It?

 On your reverse proxy if you're using one.
 On your web server if no reverse proxy is present.
 On a load balancer, if traffic is distributed among multiple servers.

## How Does It Work?

1. A browser requests `https://yourwebsite.com`
2. The server responds with its SSL certificate.
3. The browser verifies the certificate's authenticity using a Certificate Authority (CA).
4. If the certificate is valid, an encrypted connection (TLS handshake) is established.
5. Data is securely exchanged.

## How to Get an SSL Certificate?

 Free: Let's Encrypt (Most common for small websites)
 Paid: From providers like DigiCert, GlobalSign, GoDaddy (Better for enterprise-level security)

## How to Install SSL on Nginx?

1. Install Certbot (Let's Encrypt tool):
   ```
   sudo apt update
   sudo apt install certbot python3-certbot-nginx
   ```
2. Obtain a certificate:
   ```
   sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
   ```
3. Auto-renew SSL certificate:
   ```
   sudo certbot renew --dry-run
   ```
# 3) Firewall

## What is a Firewall?

A firewall is a security system that monitors and controls incoming and outgoing network traffic based on predefined rules.

## Why Do You Need It?

 Blocks unauthorized access to your server.
 Protects against brute-force attacks and malicious traffic.
 Filters traffic to allow only necessary services (e.g., web traffic on port 80/443).

## How Does It Work?

A firewall inspects packets (units of data) and decides whether to allow or block them based on security rules.

## Types of Firewalls:

1. Network Firewall: Filters traffic between networks (e.g., cloud-based firewalls).
2. Host-based Firewall: Runs on individual servers (e.g., UFW, iptables).
3. Web Application Firewall (WAF): Protects web apps from attacks like SQL injection, XSS.

## How to Configure UFW (Uncomplicated Firewall) on Ubuntu?

1. Enable UFW:
   ```
   sudo ufw enable
   ```
2. Allow HTTP/HTTPS traffic:
   ```
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   ```
3. Allow SSH (so you don't lock yourself out):
   ```
   sudo ufw allow 22/tcp
   ```
4. Check status:
   ```
   sudo ufw status
   ```

```bash
#in simple way
sudo apt install ufw  # Install UFW if not installed
sudo ufw enable       # Enable the firewall
sudo ufw allow 22     # Allow SSH
sudo ufw allow 80     # Allow HTTP
sudo ufw allow 443    # Allow HTTPS
sudo ufw status       # Check firewall status

```

# 4) Load Balancer

## Why Do You Need It?

 Distributes traffic across multiple backend servers.
 Improves availability by preventing a single server from becoming overloaded.
 Enhances performance by serving requests from the least busy server.
 Ensures redundancy so that if one server fails, traffic is redirected.

## How Does It Work?

1. A client requests `https://yourwebsite.com`
2. The load balancer receives the request.
3. Based on its algorithm, it selects the best backend server.
4. The backend server processes the request and sends a response.
5. The load balancer forwards the response back to the client.

## Load Balancing Algorithms

 Round Robin: Requests go to each server in order.
 Least Connections: The request goes to the server with the fewest active connections.
 IP Hashing: Requests from the same client go to the same server.

## Popular Load Balancers

 Nginx: Can work as both a reverse proxy and load balancer.
 HAProxy: High-performance, widely used for large-scale applications.
 Apache Traffic Server: Good for caching and load balancing.
 Cloud-based: AWS Elastic Load Balancer (ELB), Google Cloud Load Balancer.

## How to Set Up Load Balancing with Nginx?

1. Install Nginx:
   ```
   sudo apt update
   sudo apt install nginx
   ```
2. Configure `nginx.conf`:
   ```nginx
   upstream backend_servers {
       server server1.example.com;
       server server2.example.com;
       #or like
       #server 192.168.1.10;
       #server 192.168.1.11;
   }

   server {
       listen 80;
       server_name yourdomain.com;

       location / {
           proxy_pass http://backend_servers;
       }
   }
   ```
3. Restart Nginx:
   ```
   sudo systemctl restart nginx
   ```

# What Else Should You Learn?

1. Basic Linux Commands (file management, permissions, users)
2. Database Management (MySQL, PostgreSQL)
3. Web Server Configurations (Apache vs. Nginx)
4. Docker & Containers (For modern deployments)
5. Monitoring & Logging (Prometheus, Grafana, Logrotate)
6. Automation & Scripting (Bash scripting, Ansible)
7. Automation & Configuration Management
    - Ansible: Automate server configurations
    - Terraform: Infrastructure as Code
8. Security Best Practices
    - SSH Hardening (disable root login, use keys instead of passwords)
    - Regular software updates
    - Intrusion detection systems like Fail2ban

# 1) Basic Linux Commands (File Management, Permissions, Users)

## A. File Management
Linux uses a hierarchical file system where everything is treated as a file, including devices.

### Important File Management Commands
1. List Files (`ls`)
   - `ls` → List files in the current directory.
   - `ls -l` → Detailed list with permissions, ownership, and size.
   - `ls -a` → Shows hidden files (starting with `.`).
   - `ls -lh` → Human-readable file sizes.

2. Navigate Directories (`cd`)
   - `cd /path/to/directory` → Move to a directory.
   - `cd ..` → Move one level up.
   - `cd -` → Go back to the previous directory.

3. Create and Remove Directories
   - `mkdir newdir` → Create a new directory.
   - `mkdir -p parentdir/childdir` → Create nested directories.
   - `rmdir emptydir` → Remove an empty directory.
   - `rm -r dir` → Remove a non-empty directory.

4. Create, View, and Modify Files
   - `touch filename` → Create an empty file.
   - `cat file` → View contents of a file.
   - `less file` → View large files page by page.
   - `nano file` → Edit a file using Nano.
   - `vim file` → Edit using Vim.
   - `echo "Hello" > file.txt` → Write to a file (overwrites).
   - `echo "Hello" >> file.txt` → Append to a file.

5. Copy, Move, and Delete Files
   - `cp file1 file2` → Copy a file.
   - `cp -r dir1 dir2` → Copy a directory.
   - `mv file1 file2` → Move/rename a file.
   - `rm file` → Delete a file.
   - `rm -rf dir` → Force delete a directory and its contents.

## B. File Permissions
Linux uses a permission system with three entities:
- Owner (u) → The user who created the file.
- Group (g) → Users in the same group.
- Others (o) → Everyone else.

### Permission Structure (`ls -l`)
Example output:
```
-rw-r--r-- 1 user group 1234 Mar 9 10:00 file.txt
```
- `-` → File type (`d` for directories, `-` for files).
- `rw-` → Owner (read, write, no execute).
- `r--` → Group (read-only).
- `r--` → Others (read-only).

### Changing Permissions (`chmod`)
- `chmod 777 file` → Full access to everyone.
- `chmod 644 file` → Owner can read/write, others can only read.
- `chmod +x script.sh` → Make a script executable.

### Changing Ownership (`chown`)
- `chown user:group file` → Change file owner and group.
- `chown -R user:group dir/` → Change ownership for a directory and its contents.

## C. User Management
### User Commands
- `whoami` → Show current user.
- `id` → Show user ID (UID) and group ID (GID).
- `adduser username` → Create a new user.
- `passwd username` → Change a user's password.
- `deluser username` → Remove a user.
- `usermod -aG groupname username` → Add user to a group.

### Group Commands
- `groups` → Show groups the user belongs to.
- `groupadd newgroup` → Create a group.
- `gpasswd -a username groupname` → Add user to a group.
- `gpasswd -d username groupname` → Remove user from a group.

## Understanding Linux File Permissions and the Number System (chmod)

Linux file permissions are based on a three-level system:
1. User (Owner) → The person who created or owns the file.
2. Group → Other users who belong to the same group as the owner.
3. Others (Everyone else) → Any user who is not the owner or in the group.

Each file has three types of permissions:
- Read (r) → Allows viewing the file’s contents.
- Write (w) → Allows modifying or deleting the file.
- Execute (x) → Allows running the file (if it’s a script or program).

### 1. Checking File Permissions
To check the permissions of a file or directory, use:
```
ls -l filename
```
Example output:
```
-rwxr--r-- 1 user group 1234 Mar 9 10:00 myscript.sh
```
Breakdown:
- `-` → Regular file (if `d`, it’s a directory).
- `rwx` → Owner (user) can read, write, and execute.
- `r--` → Group can only read.
- `r--` → Others can only read.

### 2. Changing Permissions with `chmod`

### A) Using Symbolic Mode (`+`, `-`, `=`)
**+** -> Add permission
**-** -> Remove permission
**=** -> Set exact permission

Examples:
1. Give execute permission to the owner (`u`):
   ```
   chmod u+x myscript.sh
   ```
2. Remove write permission from others (`o`):
   ```
   chmod o-w myscript.sh
   ```
3. Give read & write permission to group (`g`):
   ```
   chmod g+rw myscript.sh
   ```
4. Set exact permissions (Owner: rwx, Group: r, Others: -):
   ```
   chmod u=rwx,g=r,o= myscript.sh
   ```
   
### B) Using Numeric (Octal) Mode
Linux uses a three-digit number to represent permissions.

Each permission has a value:
- Read (`r`) = 4
- Write (`w`) = 2
- Execute (`x`) = 1

To set permissions, add up the values for each level:
| `---` | 0 |
| `--x` | 1 |
| `-w-` | 2 |
| `-wx` | 3 |(2+1) |
| `r--` | 4 |
| `r-x` | 5 |(4+1) |
| `rw-` | 6 |(4+2) |
| `rwx` | 7 |(4+2+1) |

### Examples
1. Full permissions for the owner, read for others (`chmod 744 filename`)  
   ```
   chmod 744 myscript.sh
   ```
   - Owner: `7` → `rwx`
   - Group: `4` → `r--`
   - Others: `4` → `r--`

2. Only the owner can read and write (`chmod 600 filename`)  
   ```
   chmod 600 secret.txt
   ```
   - Owner: `6` → `rw-`
   - Group: `0` → `---`
   - Others: `0` → `---`

3. Everyone can read and execute (`chmod 755 filename`)  
   ```
   chmod 755 myscript.sh
   ```
   - Owner: `7` → `rwx`
   - Group: `5` → `r-x`
   - Others: `5` → `r-x`

4. Give full permissions to everyone (`chmod 777 filename`)  
   ```
   chmod 777 public.sh
   ```
   - Owner: `7` → `rwx`
   - Group: `7` → `rwx`
   - Others: `7` → `rwx`
   - ⚠️ Security risk: Anyone can modify or delete the file.

## 3. Special Permissions
Beyond the basic `rwx` permissions, Linux has three special permission bits:

### A) SetUID (`chmod u+s`)
- Applies to executable files.
- When executed, runs with the owner's permissions instead of the user's.
- Example (allows a normal user to execute a command as root):
  ```
  chmod u+s /usr/bin/passwd
  ```
  - `ls -l /usr/bin/passwd`
    ```
    -rwsr-xr-x 1 root root 47032 Mar 9 10:00 /usr/bin/passwd
    ```

### B) SetGID (`chmod g+s`)
- Applies to directories.
- Files created inside inherit the group of the directory.
- Example:
  ```
  chmod g+s shared_folder
  ```

### C) Sticky Bit (`chmod +t`)
- Applies to directories.
- Only the file owner can delete their own files inside the directory.
- Used in `/tmp` to prevent users from deleting others' files.
- Example:
  ```
  chmod +t /tmp
  ```
  - `ls -ld /tmp`
    ```
    drwxrwxrwt 1 root root 4096 Mar 9 10:00 /tmp
    ```
### 4. Recursively Changing Permissions
To change permissions for all files inside a directory:
```
chmod -R 755 mydirectory
```
This applies `755` to all files and subdirectories inside `mydirectory`.

### 5. Practical Scenarios

| Make a script executable | `chmod +x script.sh` |
| Secure a private file | `chmod 600 myfile.txt` |
| Allow users to read and execute a program | `chmod 755 myprogram` |
| Share a directory with group members | `chmod 770 shared_folder` |
| Protect `/tmp` from users deleting others' files | `chmod +t /tmp` |

### Final Summary
- Use `ls -l` to check permissions.
- Symbolic mode (`chmod u+x file`) is easy for small changes.
- Numeric mode (`chmod 755 file`) is faster and used in scripts.
- Use `-R` to change permissions recursively.
- Special permissions (SetUID, SetGID, Sticky Bit) are useful for security.

# Docker & Containers (For Modern Deployments)

## What is Docker?
Docker is a tool for running applications inside containers, which are lightweight, isolated environments that package applications and their dependencies.

## Why Use Docker?
- Portability: Runs the same way on any system.
- Efficiency: Uses fewer resources than VMs.
- Consistency: Works across development, testing, and production environments.
- Scalability: Easily deploy multiple instances of an app.

## Docker vs Virtual Machines
| Feature      | Docker (Containers)  | Virtual Machines |
| Isolation    | Process-level        | Full OS-level    |
| Performance  | Fast, lightweight    | Slower, heavier  |
| Size         | Small (MBs)          | Large (GBs)      |
| Startup Time | Seconds              | Minutes          |

## Installing Docker on Ubuntu
1. Update Packages
   ```
   sudo apt update
   sudo apt install apt-transport-https ca-certificates curl software-properties-common
   ```
2. Add Docker Repository
   ```
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```
3. Install Docker
   ```
   sudo apt update
   sudo apt install docker-ce docker-ce-cli containerd.io
   ```
4. Verify Installation
   ```
   sudo systemctl status docker
   docker --version
   ```

## Basic Docker Commands
1. Check Installed Docker Version
   ```
   docker --version
   ```
2. Run a Container
   ```
   docker run hello-world
   ```
3. List Running Containers
   ```
   docker ps
   ```
4. List All Containers
   ```
   docker ps -a
   ```
5. Stop a Container
   ```
   docker stop container_id
   ```
6. Remove a Container
   ```
   docker rm container_id
   ```
7. Remove an Image
   ```
   docker rmi image_id
   ```
8. Pull an Image
   ```
   docker pull nginx
   ```
9. Run a Container in the Background
   ```
   docker run -d -p 8080:80 nginx
   ```
10. View Logs of a Container
    ```
    docker logs container_id
    ```

## Docker Compose
Docker Compose helps run multi-container applications using a YAML file.

## Example `docker-compose.yml` for a Web App
```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  database:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: example
```
To start this:
```
docker-compose up -d
```

## Docker Networking
- `bridge` (default) → Containers communicate within the same network.
- `host` → Shares host's networking (no isolation).
- `none` → No networking.
- Creating a Custom Network:
  ```
  docker network create mynetwork
  ```

## Docker Volumes (Persistent Storage)
Docker containers are ephemeral (data is lost when stopped). Volumes store persistent data.
```
docker volume create mydata
docker run -v mydata:/data ubuntu
```
