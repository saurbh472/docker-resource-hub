# 🐳 Docker Resource Hub
A comprehensive collection of Docker commands, networking concepts, remote API configuration, and TLS security setup. This repository serves as a quick reference for beginners as well as experienced DevOps engineers who want to learn and work with Docker efficiently.

---

# Docker Commands Cheat Sheet

## 1. Basic Docker Commands

### Installation & Version Check

```bash
# Check Docker version
docker --version

# Check system-wide Docker information
docker info
```

### Working with Images

```bash
# Search for an image
docker search <image_name>

# Pull an image from Docker Hub
docker pull <image_name>

# List all images
docker images

# Remove an image
docker rmi <image_id>
```

### Working with Containers

```bash
# Run a container
docker run <image_name>

# Run a container in detached mode
docker run -d <image_name>

# Run a container with an interactive shell
docker run -it <image_name> /bin/bash

# List running containers
docker ps

# List all containers (including stopped ones)
docker ps -a

# Stop a running container
docker stop <container_id>

# Start a stopped container
docker start <container_id>

# Restart a container
docker restart <container_id>

# Remove a container
docker rm <container_id>
```

### Logs & Monitoring

```bash
# View logs of a container
docker logs <container_id>

# Show running processes in a container
docker top <container_id>

# Display resource usage statistics
docker stats
```

---

## 2. Intermediate Docker Commands

### Managing Volumes

```bash
# Create a volume
docker volume create <volume_name>

# List all volumes
docker volume ls

# Inspect a volume
docker volume inspect <volume_name>

# Remove a volume
docker volume rm <volume_name>
```

### Networking in Docker

```bash
# List available networks
docker network ls

# Create a network
docker network create <network_name>

# Connect a container to a network
docker network connect <network_name> <container_id>

# Disconnect a container from a network
docker network disconnect <network_name> <container_id>

# Inspect a network
docker network inspect <network_name>
```

### Working with Docker Compose

```bash
# Start services defined in docker-compose.yml
docker-compose up -d

# Stop services
docker-compose down

# Restart services
docker-compose restart

# Check the logs
docker-compose logs

# View running containers
docker-compose ps
```

### Copying Files to & from Containers

```bash
# Copy from host to container
docker cp <file_path> <container_id>:<destination_path>

# Copy from container to host
docker cp <container_id>:<file_path> <destination_path>
```

---

## 3. Advanced Docker Commands

### Building Custom Images

```bash
# Build an image from a Dockerfile
docker build -t <image_name> .

# Build an image with a specific tag
docker build -t <image_name>:<tag> .
```

### Tagging & Pushing Images

```bash
# Tag an image
docker tag <image_id> <repository>:<tag>

# Push an image to Docker Hub
docker push <repository>:<tag>
```

### Managing Container Resources

```bash
# Limit CPU usage
docker run --cpus="1.5" <image_name>

# Limit memory usage
docker run -m 512m <image_name>
```

### Docker Security

```bash
# Scan an image for vulnerabilities
docker scan <image_name>

# Run a container with read-only filesystem
docker run --read-only <image_name>
```

### Docker Swarm & Orchestration

```bash
# Initialize a swarm
docker swarm init

# Add a worker node
docker swarm join --token <token> <manager_ip>:2377

# List swarm nodes
docker node ls

# Deploy a service
docker service create --name <service_name> <image_name>

# List services
docker service ls
```

### Cleanup & Maintenance

```bash
# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune -a

# Remove all unused volumes
docker volume prune

# Remove all unused networks
docker network prune
```

---

# Docker Networking

## Bridge Network

### Practical Steps

```bash
1. Sudo docker run -itd –rm –name thor busybox

2. Sudo docker run -itd –rm –name mjolnir busybox

3. Sudo docker run -itd –rm –name stromebreaker nginx

4. Bridge link

5. Docker inspect bridge

6. Docker exec -it thor sh
   (ping each other and internet also in bridge)

7. Sudo docker run -itd –rm -p 80:80 –name stromebreaker nginx
   (redeploy for accessing the data on host)
```

---

## User Defined Bridge

**Why?** Isolation and Docker recommends user-defined bridges.

```bash
8. Sudo docker network create asgard

9. Ip address show

10. Network ls

11. Sudo docker run -itd –rm –network asgard –name loki busybox

12. Sudo docker run -itd –rm –network asgard –name odin busybox
```

---

## Host Network

**By default:** You don't need to expose ports manually.

```bash
13. Sudo docker stop stromebreaker

14. Sudo docker run -itd –rm –name –network host stromebreaker nginx
    (move to near to host)

15. sudo systemctl restart docker
```

---

## Macvlan Network

Container directly connects to the physical network (home/office network).

`-o parent=enp6s18` ties the Macvlan network to the host network interface.

```bash
16. Sudo docker stop thor mjolnir

17. Sudo docker run -itd –rm –network newasgard \
--ip 192.168.0.61 \
--name thor busybox
```

When you do this then you need to enable promiscuous mode.

Your network may not allow multiple MAC addresses on one switch port.

### Enable Promiscuous Mode

```bash
18. Sudo ip link set enp6s18 promisc on
```

Also enable it from VirtualBox network settings.

```bash
19. Reboot system
```

---

## IPVLAN (L2/L3)

```bash
Sudo docker network create -d ipvlan \
--subnet 192.168.0.0/16 \
--gateway 192.168.0.1 \
-o parent=enp6s18 newasgard

Docker run -itd –rm \
--network newasgard \
--ip 192.168.0.61 \
--name thor busybox
```

*(Uses the same MAC address as the host.)*

---

## Overlay Network

```
Pending (Update Soon Regarding This...)
```

---

## None Network

```
Loopback Interface
```

---

# Enable Docker Remote API on Docker Host

## Step 1: Open Docker Service File

```bash
vi /lib/systemd/system/docker.service
```

## Step 2: Modify ExecStart

Find the line starting with `ExecStart` and add `-H=tcp://0.0.0.0:2375`.

```bash
ExecStart=/usr/bin/dockerd -H=fd:// -H=tcp://0.0.0.0:2375
```

## Step 3: Save the Modified File

---

## Step 4: Reload Docker Daemon

```bash
systemctl daemon-reload
```

---

## Step 5: Restart Docker

```bash
sudo service docker restart
```

---

## Step 6: Test the API

```bash
curl http://localhost:2375/images/json
```

If everything is configured correctly, this command should return a JSON response.

---

## Step 7: Test Remotely

Use the Docker Host PC name or IP address from another machine.

---

# Docker TLS Certificate Configuration

## Step 1: Generate SSL/TLS Certificates

### Create Certificate Directory

```bash
mkdir -p /etc/docker/certs
cd /etc/docker/certs
```

### Generate CA Private Key

```bash
openssl genrsa -aes256 -out ca-key.pem 4096
```

**Note:** This command creates a private key for your Certificate Authority (CA) with 4096 bits and encrypts it with AES-256.

---

### Generate CA Certificate

```bash
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
```

**Note:** You'll be prompted to enter details like country, state, organization, etc. This creates a self-signed certificate valid for 365 days.

---

### Generate Server Key

```bash
openssl genrsa -out server-key.pem 4096
```

**Note:** This creates a private key for the Docker server with 4096 bits.

---

### Create Server CSR

```bash
openssl req -new -key server-key.pem -out server.csr
```

**Note:** You'll be prompted for details. This CSR will be signed by your CA to create the server certificate.

---

### Create Extension File

```bash
touch extfile.cnf
```

Paste:

```text
subjectAltName = IP:192.168.0.90
extendedKeyUsage = serverAuth
```

---

### Sign Server Certificate

```bash
openssl x509 -req -days 365 -sha256 \
-in server.csr \
-CA ca.pem \
-CAkey ca-key.pem \
-CAcreateserial \
-out server-cert.pem \
-extfile extfile.cnf
```

**Note:** This command creates a server certificate signed by your CA, valid for 365 days.

---

### Secure Keys and Certificates

```bash
chmod -v 0400 ca-key.pem server-key.pem

chmod -v 0444 ca.pem server-cert.pem
```

**Note:** These commands set appropriate permissions on your key and certificate files.

---

# Step 2: Configure Docker to Use SSL/TLS

### Move Certificates

```bash
mkdir -p /etc/docker

cp server-cert.pem /etc/docker/

cp server-key.pem /etc/docker/

cp ca.pem /etc/docker/
```

---

### Edit Docker Service File

```bash
sudo nano /lib/systemd/system/docker.service
```

Modify the `ExecStart` line:

```bash
ExecStart=/usr/bin/dockerd \
--tlsverify \
--tlscacert=/etc/docker/ca.pem \
--tlscert=/etc/docker/server-cert.pem \
--tlskey=/etc/docker/server-key.pem \
-H=0.0.0.0:2376
```

**Note:** Replace `0.0.0.0` with your server IP (`192.168.0.90`) if needed.

---

### Reload and Restart Docker

```bash
sudo systemctl daemon-reload

sudo systemctl restart docker
```

This reloads the Docker service configuration and restarts Docker to apply the new settings.

---

# Step 3: Configure Client for SSL/TLS

### Copy CA Certificate

```bash
scp user@192.168.0.90:/etc/docker/ca.pem ~/
```

---

### Generate Client Key and CSR

```bash
openssl genrsa -out client-key.pem 4096

openssl req -new -key client-key.pem -out client.csr
```

**Note:** The first command creates a private key for the client, and the second generates a CSR.

---

### Create Client Extension File

```bash
mkdir extfile-client.cnf
```

---

### Sign Client Certificate

```bash
openssl x509 -req -days 365 -sha256 \
-in client.csr \
-CA ca.pem \
-CAkey /etc/docker/certs/ca-key.pem \
-CAcreateserial \
-out client-cert.pem \
-extfile extfile-client.cnf
```

This creates a client certificate signed by your CA.

---

# Step 4: Test the SSL/TLS Setup

Run Docker commands from the client machine:

```bash
docker --tlsverify \
--tlscacert=ca.pem \
--tlscert=client-cert.pem \
--tlskey=client-key.pem \
-H=192.168.0.90:2376 info
```

If everything is configured correctly, the client should successfully communicate with the Docker daemon over TLS.
