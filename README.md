# 100 Days of DevOps

---

## Day 1: Linux User Setup with Non-Interactive Shell

Create service accounts that cannot log in interactively for security.

```bash
# Available non-interactive shells
/sbin/nologin      # Displays message and exits
/usr/sbin/nologin  # Alternative location
/bin/false         # Simply returns false, no message

# Create a user with non-interactive shell
# -s: Specify login shell
sudo useradd -s /sbin/nologin username

# Verify the user's shell
grep username /etc/passwd
```

---

## Day 2: Temporary User Setup with Expiry

Create users with automatic account expiration for contractors or temporary access.

```bash
# Create user with expiry date
# -e: Account expiration date (YYYY-MM-DD format)
sudo useradd -e 2025-12-31 username

# Verify account expiry settings
sudo chage -l username

# Modify expiry for existing user
sudo chage -E 2025-12-31 username
```

---

## Day 3: Secure Root SSH Access

Disable direct root login via SSH to improve security.

```bash
# View SSH configuration files
ls -lah /etc/ssh

# Edit SSH daemon configuration
sudo vi /etc/ssh/sshd_config

# Find and modify this line:
# PermitRootLogin no

# Restart SSH service to apply changes
sudo systemctl restart sshd

# Verify service is running
sudo systemctl status sshd
```

---

## Day 4: Script Execution Permissions

Manage file permissions for script execution.

```bash
# View current permissions
ls -lah /tmp

# Add read and execute permissions for all users
# a: all users, +rx: add read and execute
sudo chmod a+rx /filepath

# Common permission patterns:
# chmod 755: rwxr-xr-x (owner full, others read/execute)
# chmod 700: rwx------ (owner only)
# chmod +x: Add execute permission
```

---

## Day 5: SELinux Installation and Configuration

Configure Security-Enhanced Linux for mandatory access control.

```bash
# Check system information
hostnamectl
cat /etc/os-release

# View SELinux configuration
ls /etc/selinux/

# Edit SELinux config file
sudo vi /etc/selinux/config

# Check current SELinux mode
grep ^SELINUX= /etc/selinux/config

# SELinux modes:
# enforcing: Policies enforced, violations blocked
# permissive: Policies not enforced, violations logged
# disabled: SELinux turned off
```

---

## Day 6: Create a Cron Job

Schedule automated tasks using cron.

```bash
# Install cron daemon (if not present)
sudo yum install -y cronie

# Check cron service status
sudo systemctl status crond

# Start and enable cron service
sudo systemctl start crond
sudo systemctl enable --now crond

# List current user's crontab
sudo crontab -l

# Edit crontab
sudo crontab -e

# Crontab format: minute hour day month weekday command
# Example: Run backup daily at 2:30 AM
# 30 2 * * * /scripts/backup.sh

# View crontab storage location
sudo ls -lah /var/spool/cron/
```

---

## Day 7: Linux SSH Authentication

Set up passwordless SSH using key pairs.

```bash
# Generate SSH key pair (RSA - traditional)
ssh-keygen

# Generate SSH key pair (Ed25519 - recommended, more secure)
ssh-keygen -t ed25519

# View generated keys
ls -lah ~/.ssh/
# id_ed25519: Private key (keep secure!)
# id_ed25519.pub: Public key (share this)

# Copy public key to remote server
ssh-copy-id username@remote_ip

# Test connection (should not prompt for password)
ssh username@remote_ip
```

---

## Day 8: Install Ansible

Install Ansible automation tool using pip.

```bash
# Verify pip3 is installed
which pip3

# Install specific Ansible version
sudo pip3 install ansible==4.7.0

# Verify installation
ansible --version

# Check Ansible path
which ansible

# Configure sudo for Ansible (if needed)
sudo visudo
# Add: username ALL=(ALL) NOPASSWD: ALL
```

---

## Day 9: MariaDB Troubleshooting

Diagnose and fix common MariaDB startup issues.

```bash
# Try to connect to MariaDB
sudo mysql

# Check service status
sudo systemctl status mariadb

# Start the service
sudo systemctl start mariadb

# Common fix: Missing data directory
sudo ls -lah /var/lib/mysql

# Create missing directory with correct ownership
sudo mkdir /var/lib/mysql
sudo chown -R mysql:mysql /var/lib/mysql

# Initialize database (if needed)
sudo mysql_install_db --user=mysql

# Restart service
sudo systemctl restart mariadb
```

---

## Day 10: Linux Bash Scripts

Create a comprehensive backup script with remote transfer.

```bash
#!/bin/bash
# backup_script.sh - Backup and transfer to remote server

# Variables - customize these
SRC_DIR="/var/www/html/beta"      # Source directory to backup
BACKUP_DIR="/backup"              # Local backup directory
BACKUP_FILE="xfusioncorp_beta.zip"
DEST_USER="backupuser"            # Remote server username
DEST_SERVER="nautilus-backup"     # Remote server hostname/IP
DEST_DIR="/backup"                # Remote destination directory

# Create backup directory if it doesn't exist
mkdir -p $BACKUP_DIR

# Install zip utility if not present
if ! command -v zip &>/dev/null; then
    echo "zip not found, installing..."
    sudo yum install -y zip || sudo apt-get install -y zip
fi

# Create zip archive
echo "Creating backup archive..."
zip -r $BACKUP_DIR/$BACKUP_FILE $SRC_DIR > /dev/null

# Verify archive was created
if [ ! -f "$BACKUP_DIR/$BACKUP_FILE" ]; then
    echo "Backup failed! Archive not created."
    exit 1
fi

# Transfer archive to remote server
echo "Copying archive to backup server..."
scp -o StrictHostKeyChecking=no \
    $BACKUP_DIR/$BACKUP_FILE \
    ${DEST_USER}@${DEST_SERVER}:${DEST_DIR}/

# Check transfer status
if [ $? -eq 0 ]; then
    echo "Backup successfully transferred."
else
    echo "Failed to transfer backup."
    exit 2
fi

echo "Backup process completed."
```

```bash
# Make script executable and set ownership
touch backup_script.sh
chmod +x backup_script.sh
sudo chown -R steve /backup/
```

---

## Day 11: Install and Configure Tomcat Server

Deploy a Java web application on Tomcat.

```bash
# Transfer WAR file to server (from local machine)
scp /tmp/ROOT.war username@server_ip:~/

# Install Tomcat
sudo yum install tomcat

# Edit Tomcat configuration (change port, etc.)
sudo vi /etc/tomcat/server.xml

# View webapps directory
ls -lah /usr/share/tomcat/webapps/

# Deploy WAR file (ROOT.war deploys to root context)
sudo mv ~/ROOT.war /usr/share/tomcat/webapps/

# Start Tomcat
sudo systemctl enable --now tomcat

# Tomcat will auto-extract WAR file on startup
```

---

## Day 12: Linux Network Services

Troubleshoot network connectivity and firewall issues.

```bash
# Test connection to specific port
telnet stapp01 6400

# Check if HTTP service is running
sudo systemctl status httpd

# Find what's listening on a specific port
# -t: TCP, -u: UDP, -l: listening, -n: numeric, -p: process
sudo netstat -tulnp | grep 6400

# View iptables firewall rules
sudo iptables -L -n --line-numbers

# Add firewall rule to allow port 22 (SSH)
# -I INPUT 3: Insert at position 3 in INPUT chain
# -p tcp: Protocol TCP
# --dport 22: Destination port 22
# -j ACCEPT: Accept the connection
sudo iptables -I INPUT 3 -p tcp --dport 22 -j ACCEPT
```

---

## Day 13: IPtables Installation And Configuration

Configure a complete iptables firewall ruleset.

```bash
# Install iptables services
sudo yum install -y iptables-services

# Enable and start iptables
sudo systemctl enable --now iptables

# Flush all existing rules (start clean)
sudo iptables -F

# Allow loopback interface (localhost)
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow established and related connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH from anywhere
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow port 5002 only from specific IP
sudo iptables -A INPUT -p tcp -s 172.16.238.14 --dport 5002 -j ACCEPT

# Drop all other traffic to port 5002
sudo iptables -A INPUT -p tcp --dport 5002 -j DROP

# View rules with line numbers
sudo iptables -L -n --line-numbers

# Save rules to persist across reboots
sudo service iptables save

# Restart iptables
sudo systemctl restart iptables
```

---

## Day 14: Linux Process Troubleshooting

Debug web server connectivity issues.

```bash
# Test if web server responds
curl http://stapp01:5000

# Check HTTP daemon status
sudo systemctl status httpd

# View detailed logs with journalctl
# -x: Add explanation, -e: Jump to end, -u: Specific unit
sudo journalctl -xeu httpd

# Check what's listening on port 5000
# ss: Socket statistics (modern replacement for netstat)
sudo ss -tulnp | grep 5000

# Common issues:
# - Service not running
# - Wrong port configured
# - Firewall blocking
# - SELinux preventing access
```

---

## Day 15: Setup SSL for Nginx

Configure Nginx with SSL/TLS certificates.

```bash
# Install Nginx
sudo yum install -y nginx
sudo systemctl enable --now nginx

# View Nginx directory structure
ls -lah /etc/nginx/

# Create SSL certificate directory
sudo mkdir /etc/nginx/ssl

# Move certificates to SSL directory
# (assuming certs are in /tmp)
sudo mv /tmp/server.crt /etc/nginx/ssl/
sudo mv /tmp/server.key /etc/nginx/ssl/

# Secure the private key (read only by root)
sudo chmod 600 /etc/nginx/ssl/server.key

# Create virtual host configuration
sudo touch /etc/nginx/conf.d/nautilus.conf

# Example SSL configuration:
# server {
#     listen 443 ssl;
#     server_name example.com;
#     ssl_certificate /etc/nginx/ssl/server.crt;
#     ssl_certificate_key /etc/nginx/ssl/server.key;
#     root /usr/share/nginx/html;
# }

# Test configuration syntax
sudo nginx -t

# Reload Nginx configuration
sudo nginx -s reload
# Or restart completely
sudo systemctl restart nginx

# Edit default page
sudo vi /usr/share/nginx/html/index.html
```

---

## Day 16: Install and Configure Nginx as an LBR

Configure Nginx as a Load Balancer.

```bash
# Check backend servers are running
sudo ss -tulnp | grep httpd

# Edit main Nginx configuration
sudo vi /etc/nginx/nginx.conf

# Example load balancer configuration:
# upstream backend {
#     server app1.example.com:8080;
#     server app2.example.com:8080;
#     server app3.example.com:8080;
# }
#
# server {
#     listen 80;
#     location / {
#         proxy_pass http://backend;
#     }
# }

# Test and reload
sudo nginx -t
sudo nginx -s reload
```

---

## Day 17: Install and Configure PostgreSQL

Set up PostgreSQL database with users and permissions.

```bash
# Switch to postgres system user
sudo su - postgres

# Enter PostgreSQL interactive terminal
psql

# Create a new database user
CREATE USER kodekloud_roy WITH PASSWORD 'securepassword';

# Create a new database
CREATE DATABASE kodekloud_db4;

# Grant all privileges on database to user
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db4 TO kodekloud_roy;

# Useful psql commands:
\du    # Display/list users (roles)
\l     # List all databases
\dt    # List tables in current database
\c db  # Connect to database
\q     # Quit psql
```

---

## Day 18: Configure LAMP server

Set up Linux, Apache, MySQL/MariaDB, PHP stack.

```bash
# Install Apache and PHP with MySQL support
sudo yum install -y httpd php php-mysqlnd php-cli php-common

# Start and enable Apache
sudo systemctl enable --now httpd

# Configure Apache (change port, document root, etc.)
sudo vi /etc/httpd/conf/httpd.conf

# Restart Apache to apply changes
sudo systemctl restart httpd

# Install MariaDB server
sudo yum install -y mariadb-server

# Start and enable MariaDB
sudo systemctl enable --now mariadb

# Secure MariaDB installation (recommended)
sudo mysql_secure_installation

# Create database user with remote access
# '%' allows connection from any host
mysql -u root -p
CREATE USER 'webuser'@'%' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON webdb.* TO 'webuser'@'%';
FLUSH PRIVILEGES;
```

---

## Day 19: Install and Configure Web Application

Deploy web application files to server.

```bash
# Transfer entire directory to remote server
# -r: Recursive (include subdirectories)
scp -r blog/ tony@stapp01:~/

# On remote server, move to web root
sudo mv ~/blog/* /var/www/html/

# Set correct ownership
sudo chown -R apache:apache /var/www/html/

# Set correct permissions
sudo chmod -R 755 /var/www/html/
```

---

## Day 20: Configure Nginx + PHP-FPM Using Unix Sock

Set up Nginx with PHP-FPM for PHP processing.

```bash
# Edit Nginx configuration for PHP
sudo vi /etc/nginx/nginx.conf

# Install EPEL and Remi repositories (for newer PHP versions)
sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm

# Reset PHP module and enable Remi PHP 8.1
sudo dnf module reset php
sudo dnf module enable php:remi-8.1 -y

# Install PHP and PHP-FPM
sudo yum install -y php php-fpm php-common

# Start PHP-FPM service
sudo systemctl enable --now php-fpm

# Configure PHP-FPM pool
sudo vi /etc/php-fpm.d/www.conf
# Key settings:
# listen = /run/php-fpm/www.sock
# listen.owner = nginx
# listen.group = nginx

# Restart PHP-FPM
sudo systemctl restart php-fpm

# Nginx location block for PHP:
# location ~ \.php$ {
#     fastcgi_pass unix:/run/php-fpm/www.sock;
#     fastcgi_index index.php;
#     fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
#     include fastcgi_params;
# }
```

---

## Day 21: Set Up Git Repository on Storage Server

Create a bare Git repository for centralized version control.

```bash
# Install Git
sudo yum install -y git

# Create a bare repository
# --bare: No working directory, only Git data
sudo git init --bare /opt/repos/myproject.git

# Set ownership for Git user
sudo chown -R git:git /opt/repos/myproject.git

# Clone from another machine:
# git clone git@server:/opt/repos/myproject.git
```

---

## Day 22-32: Git Operations Reference

Common Git operations covered in these days.

```bash
# Day 22: Clone Repository
git clone https://github.com/user/repo.git

# Day 23: Fork Repository (done via GitHub/GitLab UI)

# Day 24: Create Branches
git checkout -b feature-branch
git branch new-branch

# Day 25: Merge Branches
git checkout main
git merge feature-branch

# Day 26: Manage Remotes
git remote add origin https://github.com/user/repo.git
git remote -v

# Day 27: Revert Changes
git revert HEAD        # Revert last commit
git revert <commit>    # Revert specific commit

# Day 28: Cherry Pick
git cherry-pick <commit-hash>

# Day 29: Pull Requests (done via GitHub/GitLab UI)

# Day 30: Hard Reset
git reset --hard HEAD~1    # Remove last commit
git reset --hard <commit>  # Reset to specific commit

# Day 31: Stash
git stash              # Save changes temporarily
git stash list         # List stashes
git stash pop          # Apply and remove latest stash

# Day 32: Rebase
git rebase main        # Rebase current branch onto main
git rebase -i HEAD~3   # Interactive rebase last 3 commits
```

---

## Day 33: Resolve Git Merge Conflicts

Handle merge conflicts and create Git hooks.

```bash
# View available hooks
ls -lah .git/hooks/

# Show all refs (branches and tags)
git show-ref

# Example: post-update hook for automatic tagging
# File: .git/hooks/post-update
#!/bin/bash

echo "Starting post-update hook"

for refname in "$@"; do
    echo "Processing ref: $refname"
    
    # Only create tag for master branch updates
    if [[ "$refname" == "refs/heads/master" ]]; then
        echo "Creating release tag..."
        TODAY=$(date +%F)
        git tag -a "release-$TODAY" -m "Release for $TODAY"
    fi
done

echo "Finished post-update hook"
exit 0

# Make hook executable
chmod +x .git/hooks/post-update
```

---

## Day 34-35: Additional Git Topics

Reserved for advanced Git operations.

---

## Day 36: Deploy Nginx Container on Application Server

Run Nginx in a Docker container.

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# List local images
docker images

# Pull Nginx Alpine image (lightweight)
docker pull nginx:alpine

# Run container in foreground
docker run nginx:alpine

# Run container in detached mode with custom name
# -d: Detached (background)
# --name: Container name
docker run -d --name mywebserver nginx:alpine

# View container logs
docker logs mywebserver

# Stop container
docker stop mywebserver

# Remove container
docker rm mywebserver
```

---

## Day 37: Copy File to Docker Container

Transfer files between host and container.

```bash
# List running containers
docker ps

# Copy file from host to container
# Syntax: docker cp <source> <container>:<destination>
docker cp localfile.txt ubuntu_latest:/opt/

# Verify file was copied
docker exec -it ubuntu_latest ls -lh /opt/

# Copy file from container to host
docker cp ubuntu_latest:/opt/file.txt ./local_file.txt

# Copy entire directory
docker cp ./mydir ubuntu_latest:/opt/mydir
```

---

## Day 38: Pull Docker Image

Download and tag Docker images.

```bash
# Pull specific image tag
docker pull busybox:musl

# Create new tag for existing image
# Syntax: docker tag <source> <target>
docker tag busybox:musl busybox:blog

# List images to verify
docker images | grep busybox

# Push tagged image to registry (if authenticated)
# docker push registry.example.com/busybox:blog
```

---

## Day 39: Create a Docker Image From Container

Create image from modified container state.

```bash
# Make changes to a running container
docker exec -it mycontainer bash
# ... make changes ...
exit

# Create image from container
# Syntax: docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
docker commit mycontainer myimage:v1

# With author and commit message
docker commit -a "Author Name" -m "Added custom config" mycontainer myimage:v1

# View new image
docker images | grep myimage
```

---

## Day 40: Docker EXEC Operations

Execute commands inside running containers.

```bash
# Open interactive shell in container
# -i: Interactive, -t: Allocate TTY
docker exec -it container_name sh

# Inside container, install Apache
apt update
apt install -y apache2

# View Apache configuration
ls -lah /etc/apache2

# Edit Apache ports configuration
vi /etc/apache2/ports.conf

# Restart Apache inside container
service apache2 restart

# Run single command without interactive shell
docker exec container_name cat /etc/hosts
```

---

## Day 41: Write a Docker File

Create custom Docker image with Dockerfile.

```dockerfile
# Dockerfile for Apache on Ubuntu

# Base image
FROM ubuntu:24.04

# Install Apache and clean up in single layer
RUN apt update && \
    apt install -y apache2 && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

# Change Apache to listen on port 3002
RUN sed -i 's/Listen 80/Listen 3002/g' /etc/apache2/ports.conf && \
    sed -i 's/:80/:3002/g' /etc/apache2/sites-enabled/000-default.conf

# Document the port (metadata only)
EXPOSE 3002

# Start Apache in foreground
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

```bash
# Build image from Dockerfile
# -t: Tag the image
# .: Build context (current directory)
docker build -t apache2:ubuntu .

# Run container with port mapping
# -d: Detached
# -p: Port mapping (host:container)
docker run -d -p 3002:3002 apache2:ubuntu

# Test the container
curl http://localhost:3002
```

---

## Day 42: Create a Docker Network

Create custom Docker networks for container communication.

```bash
# Create macvlan network
# --driver: Network driver type
# --subnet: Network subnet
# --ip-range: IP allocation range
docker network create \
    --driver macvlan \
    --subnet 10.10.1.0/24 \
    --ip-range 10.10.1.0/24 \
    blog

# Other network types:
# bridge: Default, isolated network
# host: Share host network stack
# none: No networking
# overlay: Multi-host networking (Swarm)

# Create bridge network
docker network create --driver bridge mynetwork

# List networks
docker network ls

# Connect container to network
docker network connect mynetwork container_name

# Inspect network details
docker network inspect mynetwork
```

---

## Day 43: Docker Ports Mapping

Map container ports to host ports.

```bash
# Run container with port mapping
# -d: Detached mode
# --name: Container name
# -p: Port mapping (host_port:container_port)
docker run -d --name news -p 5002:8000 image_name

# Multiple port mappings
docker run -d -p 80:80 -p 443:443 nginx

# Map to specific host IP
docker run -d -p 127.0.0.1:8080:80 nginx

# Map random host port
docker run -d -P nginx  # Uses random high port

# View port mappings
docker port container_name
```

---

## Day 44: Write a Docker Compose File

Define multi-container applications with Docker Compose.

```yaml
# docker-compose.yml
name: thixpin

services:
  web:
    image: httpd:latest
    container_name: httpd
    ports:
      - "5001:80"    # Host:Container
    volumes:
      - /opt/finance:/usr/local/apache2/htdocs  # Bind mount
    restart: unless-stopped
```

```bash
# Start services in detached mode
docker compose up -d

# View running services
docker compose ps

# View logs
docker compose logs -f

# Stop services
docker compose down

# Stop and remove volumes
docker compose down -v
```

---

## Day 45: Resolve Dockerfile Issues

Debug and fix Dockerfile problems.

```bash
# Pull base image to test
docker pull image_name

# Build image (watch for errors)
docker build -t lab .

# Run interactive shell to debug
docker run -it image_name sh

# Run in detached mode
docker run -d lab

# Common Dockerfile issues:
# - Wrong base image
# - Missing packages
# - Incorrect paths
# - Permission issues
# - Wrong CMD syntax

# View build history
docker history image_name

# Inspect image metadata
docker inspect image_name
```

---

## Day 46: Deploy an App on Docker Containers

Multi-container application with PHP and MariaDB.

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    container_name: php_web
    image: php:apache
    ports:
      - "5000:80"
    volumes:
      - /var/www/html:/var/www/html
    depends_on:
      - db                    # Start db before web
    restart: unless-stopped

  db:
    container_name: mysql_web
    image: mariadb:latest
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:
      MYSQL_DATABASE: mydatabase
      MYSQL_ROOT_PASSWORD: r00tP@ssw0rd!
      MYSQL_USER: myuser
      MYSQL_PASSWORD: Str0ngP@ssw0rd!
    restart: unless-stopped
```

```bash
docker compose up -d
docker compose ps
docker compose logs db
```

---

## Day 47: Docker Python App

Containerize a Python application.

```dockerfile
# Dockerfile for Python application
FROM python:3.10-slim

# Set working directory
WORKDIR /app

# Copy and install dependencies first (layer caching)
COPY src/requirements.txt .
RUN pip3 install --no-cache-dir -r requirements.txt

# Copy application source
COPY src/ .

# Document the port
EXPOSE 3002

# Run the application
CMD ["python3", "server.py"]
```

```bash
# Build the image
docker build -t nautilus/python-app .

# Run the container
docker run -d --name python-app -p 3002:3002 nautilus/python-app

# View logs
docker logs -f python-app
```

---

## Day 48: Deploy Pods in Kubernetes Cluster

Create basic Kubernetes pods.

```bash
# Create pod with dry-run to generate YAML
# --dry-run=client: Don't create, just generate
# -o yaml: Output as YAML
kubectl run pod-httpd \
    --image=httpd:latest \
    --labels=app=httpd_app \
    --dry-run=client -o yaml > pod-httpd.yaml

# Apply the YAML file
kubectl apply -f pod-httpd.yaml

# Verify pod is running
kubectl get pods
kubectl describe pod pod-httpd
```

---

## Day 49: Deploy Applications with Kubernetes Deployments

Use Deployments for scalable applications.

```bash
# List available API resources
kubectl api-resources

# Get help for create deployment
kubectl create deploy --help

# Create deployment YAML
kubectl create deploy httpd \
    --image=httpd:latest \
    --dry-run=client -o yaml > http-deployment.yaml

# Apply deployment
kubectl apply -f http-deployment.yaml

# View deployments
kubectl get deploy

# Scale deployment
kubectl scale deploy httpd --replicas=3
```

---

## Day 50: Set Resource Limits in Kubernetes Pods

Configure CPU and memory limits for pods.

```bash
# Generate pod YAML
kubectl run httpd-pod \
    --image=httpd:latest \
    --dry-run=client -o yaml > pod.yaml

# Edit pod.yaml to add resources:
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
spec:
  containers:
  - name: httpd
    image: httpd:latest
    resources:
      requests:           # Minimum guaranteed
        memory: "64Mi"
        cpu: "250m"       # 250 millicores = 0.25 CPU
      limits:             # Maximum allowed
        memory: "128Mi"
        cpu: "500m"
```

```bash
kubectl apply -f pod.yaml
```

---

## Day 51: Execute Rolling Updates in Kubernetes

Update deployments without downtime.

```bash
# View current deployment
kubectl get deploy
kubectl get pods

# Check current image
kubectl describe deploy nginx-deployment

# Update container image
kubectl set image deployment/nginx-deployment \
    nginx-container=nginx:1.21

# Watch rollout progress
kubectl rollout status deployment/nginx-deployment

# View rollout history
kubectl rollout history deployment/nginx-deployment
```

---

## Day 52: Revert Deployment to Previous Version in Kubernetes

Rollback failed deployments.

```bash
# Undo last rollout
kubectl rollout undo deploy deployment-name

# Rollback to specific revision
kubectl rollout undo deploy deployment-name --to-revision=2

# View revision history
kubectl rollout history deploy deployment-name

# View specific revision details
kubectl rollout history deploy deployment-name --revision=2
```

---

## Day 53: Resolve VolumeMounts Issue in Kubernetes

Troubleshoot ConfigMap and volume issues.

```bash
# List ConfigMaps
kubectl get cm

# View ConfigMap details
kubectl describe cm config-name

# List pods
kubectl get po

# Describe pod for errors
kubectl describe po pod-name

# Edit pod directly (not recommended for production)
kubectl edit po pod-name

# Force replace pod
kubectl apply -f file-name --force

# Copy file into pod
# -c: Specify container (if multiple)
kubectl cp file-path pod-name:file-path -c container-name

# Execute shell in pod
kubectl exec -it pod-name -c container-name -- sh
```

---

## Day 54: Kubernetes Shared Volumes

Share data between containers using emptyDir.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-devops
  labels:
    run: volume-share-devops
spec:
  volumes:
  - name: volume-share
    emptyDir: {}           # Temporary storage, deleted when pod removed

  containers:
  - name: volume-container-devops-1
    image: ubuntu:latest
    command: ['sleep', '3600']
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/ecommerce

  - name: volume-container-devops-2
    image: ubuntu:latest
    command: ['sleep', '3600']
    volumeMounts:
    - name: volume-share
      mountPath: /tmp/games    # Same volume, different path
```

```bash
kubectl apply -f pod.yaml

# Test shared volume
kubectl exec -it volume-share-devops -c volume-container-devops-1 -- bash
echo "Hello" > /tmp/ecommerce/test.txt
exit

kubectl exec -it volume-share-devops -c volume-container-devops-2 -- bash
cat /tmp/games/test.txt  # Should show "Hello"
```

---

## Day 55: Kubernetes Sidecar Containers

Implement sidecar pattern for log aggregation.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
  labels:
    run: webserver
spec:
  volumes:
  - name: shared-logs
    emptyDir: {}

  containers:
  # Main application container
  - name: nginx-container
    image: nginx:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx

  # Sidecar container for log processing
  - name: sidecar-container
    image: ubuntu:latest
    command: 
      - "sh"
      - "-c"
      - "while true; do cat /var/log/nginx/access.log /var/log/nginx/error.log; sleep 30; done"
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
```

```bash
kubectl apply -f webserver.yaml

# View logs from sidecar
kubectl logs -f webserver -c sidecar-container
```

---

## Day 56: Deploy Nginx Web Server on Kubernetes Cluster

Create deployment with service exposure.

```bash
# Create deployment with replicas
kubectl create deploy nginx-deployment \
    --image=nginx:latest \
    --replicas=3 \
    --dry-run=client -o yaml > deployment.yaml

# Expose deployment as NodePort service
kubectl expose deploy nginx-deployment \
    --name=nginx-service \
    --type=NodePort \
    --target-port=80 \
    --port=80 \
    --dry-run=client -o yaml > service.yaml

# Apply both
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Get service details (note the NodePort)
kubectl get svc nginx-service
```

---

## Day 57: Print Environment Variables

Pass environment variables to pods.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
  labels:
    run: print-envars-greeting
spec:
  containers:
  - name: print-env-container
    image: bash
    env:
    - name: GREETING
      value: "Welcome to"
    - name: COMPANY
      value: "Nautilus"
    - name: GROUP
      value: "Group"
    # Use shell variable expansion
    command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
  restartPolicy: Never
```

```bash
# Generate with kubectl
kubectl run print-envars-greeting \
    --image=bash \
    --env="GREETING=Welcome to" \
    --env="COMPANY=Nautilus" \
    --env="GROUP=Group" \
    --dry-run=client -o yaml

# View pod logs for output
kubectl logs print-envars-greeting
```

---

## Day 58: Deploy Grafana on Kubernetes Cluster

Deploy Grafana with NodePort service.

```bash
# Create deployment
kubectl create deployment grafana-deployment-devops \
    --image=grafana/grafana \
    --dry-run=client -o yaml > grafana-deploy.yaml

kubectl apply -f grafana-deploy.yaml

# Create NodePort service with specific port
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana-service-devops
  labels:
    app: grafana-deployment-devops
spec:
  type: NodePort
  selector:
    app: grafana-deployment-devops
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 32000       # Specific NodePort (30000-32767)
    protocol: TCP
```

```bash
kubectl apply -f grafana-service.yaml
kubectl get svc -o wide
```

---

## Day 59: Troubleshoot Deployment issues in Kubernetes

Debug failing deployments.

```bash
# Describe deployment for events and status
kubectl describe deploy deployment-name

# Describe pod for detailed error messages
kubectl describe po pod-name

# Edit deployment directly
kubectl edit deploy deployment-name

# Edit pod (creates replacement)
kubectl edit po pod-name

# Common issues to check:
# - Image pull errors (wrong image name/tag)
# - Resource constraints
# - Failed health checks
# - ConfigMap/Secret missing
# - PVC not bound

# View pod logs
kubectl logs pod-name

# View previous container logs (if crashed)
kubectl logs pod-name --previous
```

---

## Day 60: Persistent Volumes in Kubernetes

Configure persistent storage for pods.

```yaml
# PersistentVolume - Cluster-level storage resource
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nautilus
spec:
  storageClassName: manual
  capacity:
    storage: 3Gi
  accessModes:
    - ReadWriteOnce          # RWO: Single node read-write
  hostPath:
    path: "/mnt/data"

---
# PersistentVolumeClaim - Request for storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nautilus
spec:
  storageClassName: manual   # Must match PV
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi

---
# Pod using the PVC
apiVersion: v1
kind: Pod
metadata:
  name: pod-nautilus
  labels:
    app: nautilus
spec:
  volumes:
  - name: nautilus-storage
    persistentVolumeClaim:
      claimName: pvc-nautilus
  containers:
  - name: container-nautilus
    image: httpd:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: nautilus-storage
      mountPath: "/usr/local/apache2/htdocs/"

---
# Service to expose the pod
apiVersion: v1
kind: Service
metadata:
  name: web-nautilus
spec:
  type: NodePort
  selector:
    app: nautilus
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30009
```

```bash
kubectl apply -f pv-pvc-pod.yaml
kubectl get pv,pvc,pods
```

---

## Day 61: Init Containers in Kubernetes

Use init containers for setup tasks.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-devops
  labels:
    app: ic-devops
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-devops
  template:
    metadata:
      labels:
        app: ic-devops
    spec:
      volumes:
      - name: ic-volume-devops
        emptyDir: {}

      # Init containers run before main containers
      initContainers:
      - name: ic-msg-devops
        image: fedora:latest
        command: 
          - "/bin/bash"
          - "-c"
          - "echo 'Init Done - Welcome to xFusionCorp Industries' > /ic/media"
        volumeMounts:
        - name: ic-volume-devops
          mountPath: /ic

      # Main container starts after init completes
      containers:
      - name: ic-main-devops
        image: fedora:latest
        command: 
          - "/bin/bash"
          - "-c"
          - "while true; do cat /ic/media; sleep 5; done"
        volumeMounts:
        - name: ic-volume-devops
          mountPath: /ic
```

---

## Day 62: Manage Secrets in Kubernetes

Store sensitive data using Secrets.

```bash
# Create secret from file
kubectl create secret generic ecommerce \
    --from-file=/opt/ecommerce.txt

# Create secret from literals
kubectl create secret generic db-creds \
    --from-literal=username=admin \
    --from-literal=password=secret123

# View secret (base64 encoded)
kubectl get secret ecommerce -o yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-nautilus
  labels:
    run: secret-nautilus
spec:
  volumes:
  - name: secret-volume
    secret:
      secretName: ecommerce
  containers:
  - name: secret-container-nautilus
    image: ubuntu:latest
    command: ["sleep", "3600"]
    volumeMounts:
    - name: secret-volume
      mountPath: "/opt/demo"
      readOnly: true
```

```bash
kubectl apply -f secret-pod.yaml

# Verify secret is mounted
kubectl exec -it secret-nautilus -c secret-container-nautilus -- sh
ls /opt/demo/
cat /opt/demo/ecommerce.txt
```

---

## Day 63: Deploy Iron Gallery App on Kubernetes

Multi-component application deployment.

```bash
# Create namespace
kubectl create namespace iron-namespace-devops

# Expose database deployment as ClusterIP (internal only)
kubectl expose deploy iron-gallery-deployment-devops \
    -n iron-namespace-devops \
    --name=iron-db-service-devops \
    --port=3306 \
    --type=ClusterIP
```

```yaml
# Frontend service with NodePort
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-devops
  namespace: iron-namespace-devops
  labels:
    run: iron-gallery
spec:
  type: NodePort
  selector:
    run: iron-gallery
  ports:
  - port: 80
    targetPort: 80
    nodePort: 32678
    protocol: TCP
```

---

## Day 64: Fix Python App Deployed on Kubernetes Cluster

Troubleshoot Python application deployment issues.

```bash
# Check deployment status
kubectl get deploy -n namespace

# Check pod logs
kubectl logs pod-name -n namespace

# Common Python app issues:
# - Missing dependencies in requirements.txt
# - Wrong Python version
# - Port mismatch
# - Environment variables not set
# - Volume mount issues
```

---

## Day 65: Deploy Redis Deployment on Kubernetes

Deploy Redis with ConfigMap configuration.

```bash
# Create ConfigMap from literals
kubectl create cm my-redis-config \
    --from-literal=maxmemory=2mb \
    --from-literal=maxmemory-policy=allkeys-lru
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  labels:
    app: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis-deployment
  template:
    metadata:
      labels:
        app: redis-deployment
    spec:
      volumes:
      - name: data
        emptyDir: {}
      - name: redis-config
        configMap:
          name: my-redis-config

      containers:
      - name: redis-container
        image: redis:alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "1"
        volumeMounts:
        - name: data
          mountPath: /redis-master-data
        - name: redis-config
          mountPath: /redis-master
```

---

## Day 66: Deploy MySQL on Kubernetes

Complete MySQL deployment with secrets and persistent storage.

```bash
# Create secrets for MySQL
kubectl create secret generic mysql-root-pass \
    --from-literal=password=YUIidhb667

kubectl create secret generic mysql-user-pass \
    --from-literal=username=kodekloud_tim \
    --from-literal=password=ksH85UJjhb

kubectl create secret generic mysql-db-url \
    --from-literal=database=kodekloud_db5
```

```yaml
# PersistentVolume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/mysql

---
# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi

---
# MySQL Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root-pass
              key: password
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql-db-url
              key: database
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: password
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim

---
# MySQL Service
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
    nodePort: 30007
```

---

## Day 67: Deploy Guest Book App on Kubernetes

Multi-tier application deployment (similar patterns as above).

---

## Day 68: Set Up Jenkins Server

Install Jenkins on RHEL/CentOS.

```bash
# SSH to Jenkins server
ssh root@jenkins

# Install wget
yum install -y wget

# Add Jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo

# Import Jenkins GPG key
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

# Update system
sudo yum upgrade -y

# Install Java (required for Jenkins)
sudo yum install -y fontconfig java-21-openjdk

# Install Jenkins
sudo yum install -y jenkins

# Reload systemd and start Jenkins
sudo systemctl daemon-reload
sudo systemctl enable --now jenkins

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# Access Jenkins at http://server-ip:8080
```

---

## Day 69: Install Jenkins Plugins

Manage Jenkins plugins via UI or CLI.

---

## Day 70: Configure Jenkins User Access

Set up authentication and authorization.

---

## Day 71: Configure Jenkins Job for Package Installation

Create Jenkins job that installs packages on remote servers.

```bash
# Jenkins build step (Execute shell)
sudo yum install -y $PACKAGE

# Setup SSH credentials on remote server
ssh natasha@ststor01
ssh-keygen -t ed25519
ssh-copy-id natasha@ststor01

# Configure sudo for passwordless execution
sudo visudo
# Add: natasha ALL=(ALL) NOPASSWD: ALL

# Get private key for Jenkins credentials
cat ~/.ssh/id_ed25519
```

---

## Day 72: Jenkins Parameterized Builds

Create builds with input parameters (configured in Jenkins UI).

---

## Day 73: Jenkins Scheduled Jobs

Schedule automated jobs with cron syntax.

```bash
# Jenkins build step for log backup
scp /var/log/httpd/access_log \
    /var/log/httpd/error_log \
    /usr/src/sysops/

# Schedule syntax (Build Triggers > Build periodically):
# H 2 * * *  = Run daily at 2 AM (H = hash for load distribution)
# */15 * * * * = Every 15 minutes
# H H * * 0 = Weekly on Sunday
```

---

## Day 74: Jenkins Database Backup Job

Automated database backup with remote transfer.

```bash
#!/bin/bash
# Jenkins build step

# Generate filename with date
FILE_PATH="/tmp/db_$(date +%F).sql"

# Dump database on remote server
ssh -o StrictHostKeyChecking=no -t peter@stdb01 \
    "mysqldump -u ${DB_USER} -p${DB_PASS} ${DB_NAME} > ${FILE_PATH}"

# Transfer dump to backup server
scp -o StrictHostKeyChecking=no \
    peter@stdb01:${FILE_PATH} \
    clint@stbkp01:/home/clint/db_backups/

# DB_USER, DB_PASS, DB_NAME are Jenkins parameters
```

---

## Day 75: Jenkins Slave Nodes

Add build agents to Jenkins.

```bash
# On slave node, install Java
sudo yum install -y java-17-openjdk

# Verify Java installation
java -version

# Create Jenkins user and workspace directory
sudo useradd jenkins
sudo mkdir -p /var/lib/jenkins
sudo chown jenkins:jenkins /var/lib/jenkins

# Configure agent in Jenkins UI:
# Manage Jenkins > Manage Nodes > New Node
```

---

## Day 76: Jenkins Project Security

Configure project-level security (Matrix Authorization).

---

## Day 77: Jenkins Deploy Pipeline

Declarative pipeline for code deployment.

```groovy
pipeline {
    agent { label 'ststor01' }   // Run on specific agent

    stages {
        stage('Deploy') {
            steps {
                echo 'Starting deployment process...'
                
                sh '''
                    cd /var/www/html
                    # Mark directory as safe for Git
                    git config --global --add safe.directory /var/www/html
                    # Pull latest code
                    git pull
                '''
                
                echo 'Finishing deployment process...'
            }
        }
    }
    
    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
```

---

## Day 78: Jenkins Conditional Pipeline

Pipeline with branch-based deployment.

```groovy
pipeline {
    agent { label 'ststor01' }

    parameters {
        string(
            name: 'BRANCH', 
            defaultValue: 'master', 
            description: 'Branch to deploy (master or feature)'
        )
    }

    stages {
        stage('Deploy') {
            steps {
                script {
                    echo "Starting deployment from branch: ${params.BRANCH}"

                    if (params.BRANCH == 'master') {
                        echo "Deploying master branch..."
                        sh '''
                            cd /var/www/html
                            git fetch origin master
                            git checkout master
                            git pull origin master
                        '''
                    } else if (params.BRANCH == 'feature') {
                        echo "Deploying feature branch..."
                        sh '''
                            cd /var/www/html
                            git fetch origin feature
                            git checkout feature
                            git pull origin feature
                        '''
                    } else {
                        error "Invalid branch name '${params.BRANCH}'. Only 'master' or 'feature' are allowed."
                    }

                    echo "Deployment completed successfully."
                }
            }
        }
    }
}
```

```bash
# Useful: Run Jenkins in screen for persistence
screen -S jenkins
# Detach: Ctrl + A, then D
# Reattach: screen -r jenkins
```

---

## Day 79: Jenkins Deployment Job

Standard deployment job configuration.

---

## Day 80: Jenkins Chained Builds

Create build pipeline with dependent jobs.

```bash
# Job 1: Pull Code (triggers Job 2 on success)
ssh -o StrictHostKeyChecking=no natasha@ststor01 \
    "cd /var/www/html && sudo git checkout master && sudo git pull origin master"

# Job 2: Restart Services (triggered by Job 1)
ssh -o StrictHostKeyChecking=no tony@stapp01 "sudo systemctl restart httpd"
ssh -o StrictHostKeyChecking=no steve@stapp02 "sudo systemctl restart httpd"
ssh -o StrictHostKeyChecking=no banner@stapp03 "sudo systemctl restart httpd"

# Configure in Jenkins:
# Job 1 > Post-build Actions > Build other projects > Job 2
```

---

## Day 81: Jenkins Multistage Pipeline

Pipeline with deploy and test stages.

```groovy
pipeline {
    agent any

    stages {
        stage('Deploy') {
            steps {
                script {
                    // Deploy using sshpass for password auth
                    sh '''
                        sshpass -p "Bl@kW" ssh -o StrictHostKeyChecking=no \
                            natasha@ststor01 "cd /var/www/html && git pull origin master"
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo "Testing website accessibility..."
                    
                    sh '''
                        STATUS_CODE=$(curl -o /dev/null -s -w "%{http_code}" http://stlb01:8091)
                        
                        if [ "$STATUS_CODE" -ne 200 ]; then
                            echo "Website test failed with status code $STATUS_CODE"
                            exit 1
                        else
                            echo "Website is accessible and returned HTTP 200 OK"
                        fi
                    '''
                }
            }
        }
    }
    
    post {
        failure {
            echo 'Pipeline failed - check logs for details'
        }
    }
}
```

---

## Day 82: Create Ansible Inventory for App Server Testing

Set up Ansible inventory file.

```ini
# inventory file
[apps]
stapp02 ansible_host=stapp02 ansible_user=steve ansible_password='Am3ric@'

# Alternative format with more servers
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_password='Ir0nM@n'
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_password='Am3ric@'
stapp03 ansible_host=172.16.238.12 ansible_user=banner ansible_password='BigGr33n'

[db_servers]
stdb01 ansible_host=172.16.238.20 ansible_user=peter ansible_password='Sp!dy'
```

```bash
# Test connectivity
ansible all -m ping -i inventory

# Run playbook
ansible-playbook -i inventory playbook.yml

# List hosts in inventory
ansible all -i inventory --list-hosts
```

---

## Day 83: Troubleshoot and Create Ansible Playbook

Basic Ansible playbook structure.

```yaml
# /home/thor/ansible/playbook.yml
---
- name: Create a file on App Server 2
  hosts: stapp02
  become: yes              # Run with sudo

  tasks:
    - name: Create an empty file /tmp/file.txt
      file:
        path: /tmp/file.txt
        state: touch       # Create empty file
```

```bash
# Run playbook
ansible-playbook -i inventory playbook.yml

# Dry run (check mode)
ansible-playbook -i inventory playbook.yml --check

# Verbose output
ansible-playbook -i inventory playbook.yml -v
```

---

## Day 84: Copy Data to App Servers using Ansible

Copy files to multiple servers.

```ini
# inventory
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_password=Ir0nM@n
stapp02 ansible_host=172.16.238.11 ansible_user=steve ansible_password=Am3ric@
stapp03 ansible_host=172.16.238.12 ansible_user=bruce ansible_password=BigGr33n
```

```yaml
---
- name: Copy index.html to application servers
  hosts: all
  become: true

  tasks:
    - name: Copy index.html to /opt/finance
      copy:
        src: /usr/src/finance/index.html   # Source on control node
        dest: /opt/finance/                 # Destination on targets
        remote_src: false                   # false = source is on control node
        owner: root
        group: root
        mode: '0644'
```

---

## Day 85: Create Files on App Servers using Ansible

Create files with specific ownership and permissions.

```yaml
---
- name: Create and set permissions for webdata.txt on app servers
  hosts: app_servers
  become: yes

  tasks:
    - name: Create a blank file /opt/webdata.txt
      file:
        path: /opt/webdata.txt
        state: touch
        mode: '0744'                    # rwxr--r--
        owner: "{{ ansible_user }}"     # Dynamic based on inventory
        group: "{{ ansible_user }}"
```

---

## Day 86: Ansible Ping Module Usage

Test connectivity with ping module.

```bash
# List all hosts in inventory
ansible all -i inventory --list-hosts

# Ping specific host
ansible stapp01 -i inventory -m ping

# Ping all hosts
ansible all -i inventory -m ping

# Ping hosts in specific group
ansible app_servers -i inventory -m ping

# Run arbitrary command
ansible all -i inventory -m shell -a "uptime"
```

---

## Day 87: Ansible Install Package

Install packages using yum module.

```yaml
---
- name: Install zip package on app servers
  hosts: app_servers
  become: yes

  tasks:
    - name: Install zip package
      yum:
        name: zip
        state: present      # present=install, absent=remove, latest=upgrade

    # Install multiple packages
    - name: Install multiple packages
      yum:
        name:
          - vim
          - wget
          - curl
        state: present
```

---

## Day 88: Ansible Blockinfile Module

Manage blocks of text in files.

```yaml
---
- name: Install httpd web server and configure
  hosts: all
  become: yes

  tasks:
    - name: Install httpd web server
      yum:
        name: httpd
        state: present

    - name: Start httpd service
      systemd:
        name: httpd
        state: started
        enabled: yes

    - name: Ensure index.html exists with correct permissions
      file:
        path: /var/www/html/index.html
        state: touch
        owner: apache
        group: apache
        mode: '0655'

    - name: Add content block to index.html
      blockinfile:
        path: /var/www/html/index.html
        block: |
          Welcome to XfusionCorp!
          This is Nautilus sample file, created using Ansible!
          Please do not modify this file manually!
        marker: "<!-- {mark} ANSIBLE MANAGED BLOCK -->"
```

---

## Day 89: Ansible Manage Services

Install and manage system services.

```yaml
---
- name: Install and start vsftpd
  hosts: all
  become: yes

  tasks:
    - name: Install vsftpd package
      yum:
        name: vsftpd
        state: present

    - name: Start and enable vsftpd service
      systemd:
        name: vsftpd
        state: started
        enabled: yes          # Start on boot

    # Service states: started, stopped, restarted, reloaded
```

---

## Day 90: Managing ACLs Using Ansible

Set Access Control Lists on files.

```yaml
---
- name: Create files and set ACL properties
  hosts: all
  become: yes

  tasks:
    # App Server 1 - Group ACL
    - name: Create blog.txt on App Server 1
      file:
        path: /opt/finance/blog.txt
        state: touch
      when: ansible_hostname == 'stapp01'

    - name: Set group ACL for blog.txt
      acl:
        path: /opt/finance/blog.txt
        entity: tony
        etype: group          # user or group
        permissions: 'r'      # read only
        state: present
      when: ansible_hostname == 'stapp01'

    # App Server 2 - User ACL
    - name: Create story.txt on App Server 2
      file:
        path: /opt/finance/story.txt
        state: touch
      when: ansible_hostname == 'stapp02'

    - name: Set user ACL for story.txt
      acl:
        path: /opt/finance/story.txt
        entity: steve
        etype: user
        permissions: 'rw'     # read-write
        state: present
      when: ansible_hostname == 'stapp02'

    # App Server 3 - Group ACL with rw
    - name: Create media.txt on App Server 3
      file:
        path: /opt/finance/media.txt
        state: touch
      when: ansible_hostname == 'stapp03'

    - name: Set group ACL for media.txt
      acl:
        path: /opt/finance/media.txt
        entity: banner
        etype: group
        permissions: 'rw'
        state: present
      when: ansible_hostname == 'stapp03'
```

---

## Day 91: Ansible Lineinfile Module

Manage single lines in files.

```yaml
---
- name: Install httpd and manage index.html
  hosts: all
  become: yes

  tasks:
    - name: Install httpd
      yum:
        name: httpd
        state: present

    - name: Start httpd service
      systemd:
        name: httpd
        state: started
        enabled: yes

    - name: Create index.html with initial content
      copy:
        dest: /var/www/html/index.html
        content: |
          This is a Nautilus sample file, created using Ansible!
        owner: apache
        group: apache
        mode: '0755'

    - name: Add line at beginning of file
      lineinfile:
        path: /var/www/html/index.html
        line: "Welcome to Nautilus Group!"
        insertbefore: BOF     # Beginning of file
        # insertafter: EOF    # End of file
        # insertafter: '^pattern'  # After matching pattern
```

---

## Day 92: Managing Jinja2 Templates Using Ansible

Use templates for dynamic content.

```yaml
# playbook.yml
---
- name: Deploy web page using template
  hosts: all
  become: yes

  tasks:
    - name: Create index.html from template
      template:
        src: index.html.j2              # Jinja2 template file
        dest: /var/www/html/index.html
        mode: '0655'
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
```

```jinja2
{# templates/index.html.j2 #}
<!DOCTYPE html>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h1>Welcome to {{ ansible_hostname }}</h1>
    <p>Server IP: {{ ansible_default_ipv4.address }}</p>
    <p>OS: {{ ansible_distribution }} {{ ansible_distribution_version }}</p>
    <p>Generated by Ansible</p>
</body>
</html>
```

---

## Day 93: Using Ansible Conditionals

Conditional task execution.

```bash
# Get facts about hosts
ansible all -i inventory -m setup -a 'filter=ansible_nodename'
```

```yaml
---
- name: Conditional file copy based on hostname
  hosts: all
  become: yes

  tasks:
    - name: Copy blog.txt to App Server 1
      copy:
        src: /usr/src/dba/blog.txt
        dest: /opt/dba/blog.txt
        owner: tony
        group: tony
        mode: '0644'
      when: ansible_nodename == 'stapp01.stratos.xfusioncorp.com'

    - name: Copy story.txt to App Server 2
      copy:
        src: /usr/src/dba/story.txt
        dest: /opt/dba/story.txt
        owner: steve
        group: steve
        mode: '0644'
      when: ansible_nodename == 'stapp02.stratos.xfusioncorp.com'

    - name: Copy media.txt to App Server 3
      copy:
        src: /usr/src/dba/media.txt
        dest: /opt/dba/media.txt
        owner: banner
        group: banner
        mode: '0644'
      when: ansible_nodename == 'stapp03.stratos.xfusioncorp.com'
```

```yaml
# Other conditional examples
tasks:
  # Condition based on OS family
  - name: Install on RedHat systems
    yum:
      name: httpd
      state: present
    when: ansible_os_family == "RedHat"

  - name: Install on Debian systems
    apt:
      name: apache2
      state: present
    when: ansible_os_family == "Debian"

  # Multiple conditions (AND)
  - name: Run only on CentOS 7
    debug:
      msg: "This is CentOS 7"
    when: 
      - ansible_distribution == "CentOS"
      - ansible_distribution_major_version == "7"

  # OR condition
  - name: Run on app servers
    debug:
      msg: "App server"
    when: ansible_hostname == 'stapp01' or ansible_hostname == 'stapp02'
```

---

## Day 94: Create VPC Using Terraform

Create AWS VPC with Terraform.

```hcl
# main.tf

# Configure AWS Provider
provider "aws" {
  region = "us-east-1"
}

# Create VPC
resource "aws_vpc" "devops_vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true      # Enable DNS resolution
  enable_dns_hostnames = true      # Enable DNS hostnames

  tags = {
    Name        = "devops-vpc"
    Environment = "development"
    ManagedBy   = "Terraform"
  }
}

# Output VPC ID
output "vpc_id" {
  value = aws_vpc.devops_vpc.id
}
```

```bash
# Initialize Terraform (download providers)
terraform init

# Preview changes
terraform plan

# Apply changes
terraform apply

# Apply without confirmation prompt
terraform apply -auto-approve

# Destroy resources
terraform destroy
```

---

## Day 95: Create Security Group Using Terraform

Configure AWS Security Group.

```hcl
# main.tf

# Reference existing default VPC
data "aws_vpc" "default" {
  default = true
}

# Create Security Group
resource "aws_security_group" "nautilus_sg" {
  name        = "nautilus-sg"
  description = "Security group for Nautilus App Servers"
  vpc_id      = data.aws_vpc.default.id

  # Inbound rule - HTTP
  ingress {
    description = "Allow HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]    # Allow from anywhere
  }

  # Inbound rule - SSH
  ingress {
    description = "Allow SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Outbound rule - Allow all
  egress {
    description = "Allow all outbound"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"              # All protocols
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "nautilus-sg"
  }
}

output "security_group_id" {
  value = aws_security_group.nautilus_sg.id
}
```

---

## Day 96: Create EC2 Instance Using Terraform

Launch EC2 instance with key pair.

```hcl
# main.tf

# Create SSH key pair
resource "aws_key_pair" "datacenter_kp" {
  key_name   = "datacenter-kp"
  # Paste your public key content here
  public_key = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB..."
  
  # Or read from file:
  # public_key = file("~/.ssh/id_rsa.pub")
}

# Launch EC2 instance
resource "aws_instance" "datacenter_ec2" {
  ami             = "ami-0c101f26f147fa7fd"   # Amazon Linux 2 AMI
  instance_type   = "t2.micro"
  key_name        = aws_key_pair.datacenter_kp.key_name
  security_groups = ["default"]

  tags = {
    Name        = "datacenter-ec2"
    Environment = "development"
  }
}

# Outputs
output "instance_id" {
  value = aws_instance.datacenter_ec2.id
}

output "public_ip" {
  value = aws_instance.datacenter_ec2.public_ip
}
```

---

## Day 97: Create IAM Policy Using Terraform

Create IAM policy for EC2 read-only access.

```hcl
# main.tf

resource "aws_iam_policy" "iampolicy_mariyam" {
  name        = "iampolicy_mariyam"
  description = "Policy that allows read-only access to EC2 resources"
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "EC2ReadOnly"
        Effect = "Allow"
        Action = [
          "ec2:DescribeInstances",
          "ec2:DescribeImages",
          "ec2:DescribeSnapshots",
          "ec2:DescribeVolumes",
          "ec2:DescribeSecurityGroups",
          "ec2:DescribeKeyPairs",
          "ec2:DescribeRegions",
          "ec2:DescribeAvailabilityZones",
          "ec2:DescribeVpcs",
          "ec2:DescribeSubnets"
        ]
        Resource = "*"
      }
    ]
  })

  tags = {
    Name      = "ec2-readonly-policy"
    ManagedBy = "Terraform"
  }
}

output "policy_arn" {
  value = aws_iam_policy.iampolicy_mariyam.arn
}
```

---

## Day 98: Launch EC2 in Private VPC Subnet Using Terraform

Complete VPC infrastructure with private subnet.

```hcl
# variables.tf

variable "KKE_VPC_CIDR" {
  description = "CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "KKE_SUBNET_CIDR" {
  description = "CIDR block for the subnet"
  type        = string
  default     = "10.0.1.0/24"
}
```

```hcl
# main.tf

# Get available AZs
data "aws_availability_zones" "available" {
  state = "available"
}

# Get latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux_2" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Create VPC
resource "aws_vpc" "devops_priv_vpc" {
  cidr_block           = var.KKE_VPC_CIDR
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "devops-priv-vpc"
  }
}

# Create Private Subnet
resource "aws_subnet" "devops_priv_subnet" {
  vpc_id                  = aws_vpc.devops_priv_vpc.id
  cidr_block              = var.KKE_SUBNET_CIDR
  availability_zone       = data.aws_availability_zones.available.names[0]
  map_public_ip_on_launch = false    # Private subnet - no public IPs

  tags = {
    Name = "devops-priv-subnet"
  }
}

# Create Security Group (VPC internal only)
resource "aws_security_group" "devops_priv_sg" {
  name        = "devops-priv-sg"
  description = "Security group allowing access only from within VPC"
  vpc_id      = aws_vpc.devops_priv_vpc.id

  # Allow all traffic from within VPC
  ingress {
    description = "Allow all from VPC"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = [var.KKE_VPC_CIDR]
  }

  # Allow all outbound
  egress {
    description = "Allow all outbound"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "devops-priv-sg"
  }
}

# Create EC2 Instance in Private Subnet
resource "aws_instance" "devops_priv_ec2" {
  ami                    = data.aws_ami.amazon_linux_2.id
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.devops_priv_subnet.id
  vpc_security_group_ids = [aws_security_group.devops_priv_sg.id]

  tags = {
    Name = "devops-priv-ec2"
  }
}
```

```hcl
# outputs.tf

output "KKE_vpc_name" {
  description = "Name of the VPC"
  value       = aws_vpc.devops_priv_vpc.tags["Name"]
}

output "KKE_subnet_name" {
  description = "Name of the subnet"
  value       = aws_subnet.devops_priv_subnet.tags["Name"]
}

output "KKE_ec2_private" {
  description = "Name of the EC2 instance"
  value       = aws_instance.devops_priv_ec2.tags["Name"]
}

output "private_ip" {
  description = "Private IP of EC2 instance"
  value       = aws_instance.devops_priv_ec2.private_ip
}
```

---

## Day 99: Attach IAM Policy for DynamoDB Access Using Terraform

DynamoDB table with IAM role and read-only policy.

```hcl
# variables.tf

variable "KKE_TABLE_NAME" {
  description = "Name of the DynamoDB table"
  type        = string
  default     = "xfusion-table"
}

variable "KKE_ROLE_NAME" {
  description = "Name of the IAM role"
  type        = string
  default     = "xfusion-role"
}

variable "KKE_POLICY_NAME" {
  description = "Name of the IAM policy"
  type        = string
  default     = "xfusion-readonly-policy"
}
```

```hcl
# main.tf

# Create DynamoDB Table
resource "aws_dynamodb_table" "xfusion_table" {
  name         = var.KKE_TABLE_NAME
  billing_mode = "PAY_PER_REQUEST"    # On-demand capacity
  hash_key     = "id"                  # Partition key

  attribute {
    name = "id"
    type = "S"    # String type
  }

  tags = {
    Name        = var.KKE_TABLE_NAME
    Environment = "production"
    ManagedBy   = "Terraform"
  }
}

# Create IAM Role
resource "aws_iam_role" "xfusion_role" {
  name = var.KKE_ROLE_NAME

  # Trust policy - who can assume this role
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = [
            "ec2.amazonaws.com",
            "lambda.amazonaws.com",
            "ecs-tasks.amazonaws.com"
          ]
        }
      }
    ]
  })

  tags = {
    Name      = var.KKE_ROLE_NAME
    ManagedBy = "Terraform"
  }
}

# Create IAM Policy for DynamoDB Read-Only Access
resource "aws_iam_policy" "xfusion_readonly_policy" {
  name        = var.KKE_POLICY_NAME
  description = "Read-only access policy for DynamoDB table ${var.KKE_TABLE_NAME}"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "DynamoDBReadOnly"
        Effect = "Allow"
        Action = [
          "dynamodb:GetItem",
          "dynamodb:Scan",
          "dynamodb:Query",
          "dynamodb:DescribeTable",
          "dynamodb:BatchGetItem"
        ]
        Resource = aws_dynamodb_table.xfusion_table.arn
      }
    ]
  })

  tags = {
    Name      = var.KKE_POLICY_NAME
    ManagedBy = "Terraform"
  }
}

# Attach Policy to Role
resource "aws_iam_role_policy_attachment" "xfusion_policy_attachment" {
  role       = aws_iam_role.xfusion_role.name
  policy_arn = aws_iam_policy.xfusion_readonly_policy.arn
}
```

```hcl
# outputs.tf

output "kke_dynamodb_table" {
  description = "Name of the DynamoDB table"
  value       = aws_dynamodb_table.xfusion_table.name
}

output "kke_iam_role_name" {
  description = "Name of the IAM role"
  value       = aws_iam_role.xfusion_role.name
}

output "kke_iam_policy_name" {
  description = "Name of the IAM policy"
  value       = aws_iam_policy.xfusion_readonly_policy.name
}

output "dynamodb_table_arn" {
  description = "ARN of the DynamoDB table"
  value       = aws_dynamodb_table.xfusion_table.arn
}

output "iam_role_arn" {
  description = "ARN of the IAM role"
  value       = aws_iam_role.xfusion_role.arn
}

output "iam_policy_arn" {
  description = "ARN of the IAM policy"
  value       = aws_iam_policy.xfusion_readonly_policy.arn
}
```

---

## Day 100: Create and Configure Alarm Using CloudWatch Using Terraform

Set up CloudWatch alarm with SNS notification.

```hcl
# main.tf

# Create SNS Topic for notifications
resource "aws_sns_topic" "sns_topic" {
  name = "datacenter-sns-topic"

  tags = {
    Name = "datacenter-sns-topic"
  }
}

# Optional: Add email subscription
resource "aws_sns_topic_subscription" "email" {
  topic_arn = aws_sns_topic.sns_topic.arn
  protocol  = "email"
  endpoint  = "admin@example.com"    # Change to your email
}

# Create EC2 Instance to monitor
resource "aws_instance" "datacenter_ec2" {
  ami           = "ami-0c02fb55956c7d316"    # Amazon Linux 2
  instance_type = "t2.micro"

  tags = {
    Name = "datacenter-ec2"
  }
}

# Create CloudWatch Alarm
resource "aws_cloudwatch_metric_alarm" "datacenter_alarm" {
  alarm_name          = "datacenter-alarm"
  alarm_description   = "Alarm when CPU exceeds 90%"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = 1                    # Number of periods to evaluate
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 300                  # 5 minutes in seconds
  statistic           = "Average"
  threshold           = 90                   # 90% CPU threshold

  # Dimensions to identify the resource
  dimensions = {
    InstanceId = aws_instance.datacenter_ec2.id
  }

  # Actions when alarm triggers
  alarm_actions = [
    aws_sns_topic.sns_topic.arn
  ]

  # Optional: Actions when alarm returns to OK
  ok_actions = [
    aws_sns_topic.sns_topic.arn
  ]

  tags = {
    Name = "datacenter-cpu-alarm"
  }
}
```

```hcl
# outputs.tf

output "KKE_instance_name" {
  description = "Name of the EC2 instance"
  value       = aws_instance.datacenter_ec2.tags["Name"]
}

output "KKE_alarm_name" {
  description = "Name of the CloudWatch alarm"
  value       = aws_cloudwatch_metric_alarm.datacenter_alarm.alarm_name
}

output "sns_topic_arn" {
  description = "ARN of the SNS topic"
  value       = aws_sns_topic.sns_topic.arn
}

output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.datacenter_ec2.id
}
```

```bash
# Apply the configuration
terraform init
terraform plan
terraform apply

# Check alarm status in AWS Console or CLI
aws cloudwatch describe-alarms --alarm-names "datacenter-alarm"
```

---

## Quick Reference Commands

### Linux
```bash
systemctl status|start|stop|restart|enable SERVICE
journalctl -xeu SERVICE          # View service logs
netstat -tulnp                   # Show listening ports
ss -tulnp                        # Modern netstat alternative
iptables -L -n --line-numbers    # List firewall rules
```

### Docker
```bash
docker ps -a                     # List all containers
docker logs -f CONTAINER         # Follow container logs
docker exec -it CONTAINER sh     # Shell into container
docker compose up -d             # Start compose services
docker compose down -v           # Stop and remove volumes
```

### Kubernetes
```bash
kubectl get po,svc,deploy        # List multiple resources
kubectl describe TYPE NAME       # Detailed info
kubectl logs -f POD -c CONTAINER # Follow pod logs
kubectl exec -it POD -- sh       # Shell into pod
kubectl apply -f FILE            # Apply configuration
kubectl delete -f FILE           # Delete resources
```

### Ansible
```bash
ansible all -m ping -i inventory
ansible-playbook -i inventory playbook.yml
ansible-playbook playbook.yml --check    # Dry run
ansible-playbook playbook.yml -v         # Verbose
```

### Terraform
```bash
terraform init                   # Initialize
terraform plan                   # Preview changes
terraform apply                  # Apply changes
terraform destroy                # Destroy resources
terraform state list             # List managed resources
terraform output                 # Show outputs
```

---

## Congratulations! 🎉

You've completed 100 Days of DevOps covering:
- **Linux Administration** (Days 1-20)
- **Git Version Control** (Days 21-35)
- **Docker Containerization** (Days 36-47)
- **Kubernetes Orchestration** (Days 48-67)
- **Jenkins CI/CD** (Days 68-81)
- **Ansible Configuration Management** (Days 82-93)
- **Terraform Infrastructure as Code** (Days 94-100)
