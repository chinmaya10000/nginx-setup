### Nginx Setup with SSL and WebSocket Support

## Steps

### 1. Server Provisioning
- I am using AWS as the Cloud Provider
- Provision an EC2 instance on AWS with a secure Linux distribution (e.g., Ubuntu 24.04).
- Configure security groups to allow necessary incoming traffic and restrict unnecessary access.
- Generate and configure SSH key pairs for secure remote acces.

### 2. Install Docker and Docker-compose on ths machine
```
sudo apt-get update -y
sudo apt-get install docker.io -y
sudo usermod -aG docker ubuntu
sudo systemctl enable --now docker
sudo apt install docker-compose -y
```

### 3. Build Docker Image and run 2 docker container from compose
```
docker build -t myapp:1.0 .
docker-compose up --build -d
```
### 4. Install Nginx
```
sudo apt install nginx -y
nginx -v
```

### 5. Configure nginx.conf
```
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include mime.types;
    
    # Upstream block to define the Node.js backend servers
    upstream nodejs_cluster {
        least_conn;
        server 127.0.0.1:3001;
        server 127.0.0.1:3002;
        server 127.0.0.1:3003;
    }

    server {
        listen 443 ssl;   # Listen on port 443 for HTTPS
        server_name 18.117.86.145;
        
        # SSL certificate settings
        ssl_certificate /home/ubuntu/nginx-certs/nginx-selfsigned.crt;
        ssl_certificate_key /home/ubuntu/nginx-certs/nginx-selfsigned.key;
        
        # Proxying requests to Node.js cluster
        location / {
            proxy_pass http://nodejs_cluster;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
    
    # Optional server block for HTTP to HTTPS redirection
    server {
        listen 8080;  # Listen on port 80 for HTTP
        server_name 18.117.86.145;

        # Redirect all HTTP traffic to HTTPS
        location / {
            return 301 https://$host$request_uri;
        }
    }
}
```

### 6. Reload nginx
```
sudo systemctl reload nginx
cat /var/log/nginx/access.log
cat /var/log/nginx/error.log
http://18.117.86.145:8080
```

### 7. Generate SSL Certificates:
- create folder for nginx certificates
```
mkdir ~/nginx-certs
cd ~/nginx-certs
```
- create self-signed certificate
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx-selfsigned.key -out nginx-selfsigned.crt
ls
cat nginx-certs/nginx-selfsigned.crt
sudo systemctl reload nginx
```

### 8. Test
```
https://18.117.86.145/
```

## System Architecture

### 1. Load Balancer
The diagram below illustrates how the Nginx server acts as a load balancer, routing requests to two backend servers on different ports.
![Load Balancer Diagram](assets/load-balancer.png)

### 2. HTTPS Setup
The diagram below shows the HTTPS configuration for secure communication using SSL/TLS.

![HTTPS Diagram](assets/https-diagram.png)