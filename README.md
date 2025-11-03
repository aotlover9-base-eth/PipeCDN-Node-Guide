# 🚀 RUN PIPE MAINNET WITHOUT ERRORS (Docker Edition)

This guide provides a robust and error-free method for deploying a PipeCDN POP Node. It uses **Docker** to bypass common `GLIBC` (OS version) compatibility issues found on older Ubuntu/Debian distributions and includes essential network troubleshooting steps.

### 1. Requirements

* **Supported OS**: Any modern Linux distribution (Ubuntu 22.04+, Debian 11+, etc.).
* **Software**: **Docker** must be installed on the host system.
* **Network**: Open TCP ports: **80** and **443** (Inbound and Outbound).

---

### 2. Initial Setup and Configuration

#### Step 2.1 — Create Installation Directory & Download Binary

First, create the necessary directory and download the Pipe binary.

```bash
mkdir -p /opt/pipe
cd /opt/pipe
curl -L "[https://pipe.network/p1-cdn/releases/latest/download/pop](https://pipe.network/p1-cdn/releases/latest/download/pop)" -o pop
chmod +x pop
```

#### Step 2.2 — Create `.env` Configuration File

Create the required configuration file and replace the placeholder values with your specific details.

```bash
nano /opt/pipe/.env
```

**Content for `.env`:**

```ini
# Wallet for earnings
NODE_SOLANA_PUBLIC_KEY=your_solana_wallet_address

# Node identity
NODE_NAME=my-pop-node
NODE_EMAIL="operator@example.com"
NODE_LOCATION="San Francisco, USA"

# Cache configuration
MEMORY_CACHE_SIZE_MB=512
DISK_CACHE_SIZE_GB=100
DISK_CACHE_PATH=./cache

# Network ports
HTTP_PORT=80
HTTPS_PORT=443

# IMPORTANT: Set this to false on all VPS/Cloud Servers
UPNP_ENABLED=false
```

---

### 3. Network Troubleshooting (Avoid Registration Errors)

This is a critical step to prevent common "Failed to connect to control plane" errors by ensuring proper network connectivity.

#### Step 3.1 — Stabilize Host DNS Resolution

Temporarily configure your host system to use reliable public DNS servers.

```bash
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
echo "nameserver 1.1.1.1" | sudo tee -a /etc/resolv.conf
```

#### Step 3.2 — Configure Local Firewall (`ufw`)

Ensure your local firewall (`ufw`) allows necessary inbound and outbound traffic.

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable  # WARNING: Confirm SSH (Port 22) is allowed before enabling!
sudo ufw status
```

**Cloud Firewall Check:** If you are using a VPS provider (AWS, DigitalOcean, etc.), log into their dashboard and ensure the **Cloud Firewall/Security Group** allows **INBOUND** traffic on ports **80/443** and **OUTBOUND** traffic on port **443**. This is crucial.

#### Step 3.3 — Test API Connectivity

Verify that your server can successfully connect to the Pipe CDN API endpoint.

```bash
curl -v https://api.plpipe.com
```

*Expected Result: The output should show the connection succeeding, displaying SSL handshake details and ideally a small HTML/JSON response, not a "Could not resolve host" or "Connection reset" error.*

---

### 4. Running the Node with Docker

We use Docker to create a compatible environment (Ubuntu 24.04) and run the Pipe CDN node, isolating it from potential host OS dependency issues.

#### Step 4.1 — Docker Installation (If Not Already Installed)

If Docker is not yet installed on your host system, install it using these commands:

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

#### Step 4.2 — Create the `Dockerfile`

Create this file in your `/opt/pipe` directory. It instructs Docker how to build the container image.

```bash
nano /opt/pipe/Dockerfile
```

**Content for `Dockerfile`:**

```dockerfile
# Use the required OS as the base
FROM ubuntu:24.04

# Set working directory inside the container
WORKDIR /opt/pipe

# Install curl (useful for debugging inside the container) and update package list
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Download and install the binary (included for self-contained image builds)
RUN curl -L "[https://pipe.network/p1-cdn/releases/latest/download/pop](https://pipe.network/p1-cdn/releases/latest/download/pop)" -o pop
RUN chmod +x pop

# Copy your local .env file into the container
COPY .env .

# Expose the necessary ports from the container
EXPOSE 80 443 8081 9090

# Command to run the node when the container starts
CMD ["/bin/bash", "-c", "source .env && /opt/pipe/pop"]
```

#### Step 4.3 — Build the Docker Image

Navigate to your `/opt/pipe` directory and build the Docker image.

```bash
cd /opt/pipe
sudo docker build -t pipe-node .
```

#### Step 4.4 — Run the Container

Run the Docker container, mapping all necessary ports from the host to the container and ensuring it restarts automatically.

```bash
sudo docker run -d \
  --name pipe-node \
  -p 80:80 \
  -p 443:443 \
  -p 8081:8081 \
  -p 9090:9090 \
  --restart always \
  pipe-node
```

#### Step 4.5 — Check Logs

Monitor the container logs to confirm successful startup and registration with the Pipe CDN control plane.

```bash
sudo docker logs -f pipe-node
```

---

### 5. Verification and Monitoring

Use `docker exec` to run commands inside your running container to check the node's status and earnings.

**Check Node Status:**

```bash
sudo docker exec pipe-node /opt/pipe/pop status
```

**Check Node Earnings:**

```bash
sudo docker exec pipe-node /opt/pipe/pop earnings
```

---

### Support and Community

For further assistance, feel free to join our Telegram channel or reach out directly.

* **Telegram Channel:** [https://t.me/earningmastermind69](https://t.me/earningmastermind69)
* **Direct Message:** [@The\_Aotlover9\_bot](https://www.google.com/search?q=https://t.me/The_Aotlover9_bot)

```
