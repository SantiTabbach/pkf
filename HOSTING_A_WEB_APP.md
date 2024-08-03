# Hosting a Web Application on a Local Nginx Server: A Step-by-Step Guide

## Technologies Used

- Nginx: A high-performance web server that can also be used as a reverse proxy, load balancer, and HTTP cache.
- Docker: A platform for developing, shipping, and running applications in containers.
- Certbot: A tool to automatically use Let’s Encrypt certificates for HTTPS.
- UFW (Uncomplicated Firewall): A simple firewall to manage iptables firewall rules on Ubuntu.
- Let's Encrypt: A free, automated, and open certificate authority (CA) that provides SSL certificates.

## 1. Install Necessary Dependencies

First, update your package list and install the required dependencies:

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y nginx docker.io certbot python3-certbot-nginx ufw
```

## 2. Start and Enable Nginx and Docker

Next, start and enable Nginx and Docker to ensure they run on system startup:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx

sudo systemctl start docker
sudo systemctl enable docker
```

## 3. Configure Docker

Run the following command to create and run a Docker container for your application:

```bash
sudo docker run -d --name my_container -p 3000:3000 my_docker_image
```

## 4. Configure Nginx for Your Domain

Create a configuration file for your domain:

```bash
sudo nano /etc/nginx/sites-available/example.com
```

Add the following configuration to the file:

```
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://localhost:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Create a symbolic link to enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```

Verify the Nginx configuration:

```bash
sudo nginx -t
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

## 5. Obtain SSL Certificates with Let's Encrypt

Run Certbot to obtain and configure SSL certificates for your domain:

```bash
sudo certbot --nginx -d example.com
```

## 6. Verify SSL Configuration

Ensure the final configuration in `/etc/nginx/sites-available/example.com` looks similar to this:

```nginx
server {
    listen 443 ssl;
    server_name example.com;
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";

    location / {
        proxy_pass http://localhost:3000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```

Restart Nginx to apply the changes:

```bash
sudo systemctl restart nginx
```

## 7. Configure the Firewall

Set up the firewall to allow Nginx traffic:

```bash
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
sudo ufw enable
```

## 8. Deploy Automatically

Create a shell script on your server for automated deployment, e.g., `deploy.sh`:

```bash
#!/bin/bash

# Navigate to the project directory
cd /path/to/your/project

# Stop and remove the existing container if it exists
if [ "$(sudo docker ps -aq -f name=my_container)" ]; then
    echo "Stopping and removing the existing 'my_container' container..."
    sudo docker-compose down
fi

# Update the image
echo "Updating the image..."
sudo docker-compose pull

# Bring up the container with the new image
echo "Bringing up the container with the new image..."
sudo docker-compose up -d
```

Make the script executable:

```bash
chmod +x deploy.sh
```

Run the script each time you need to deploy a new version:

```bash
./deploy.sh
```

### Script Explanation

- `cd /path/to/your/project`: Navigates to the directory where your `docker-compose.yml` file is located.
- `if [ "$(sudo docker ps -aq -f name=my_container)" ]; then`: Checks if a container named `my_container` exists.
- `sudo docker-compose down`: Stops and removes containers, networks, and volumes defined in the `docker-compose.yml` file.
- `sudo docker-compose pull`: Pulls the latest version of the container image from Docker Hub.
- `sudo docker-compose up -d`: Starts the containers in detached mode with the new image.
