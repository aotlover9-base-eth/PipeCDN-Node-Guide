````markdown
## 🚀 RUN PIPE MAINNET WITHOUT ERRORS (Docker Edition)

This guide provides a robust and error-free method for deploying a PipeCDN POP Node. It uses **Docker** to bypass common `GLIBC` (OS version) compatibility issues found on older Ubuntu/Debian distributions and includes essential network troubleshooting steps.

### 1. Requirements

* **Supported OS**: Any modern Linux distribution (Ubuntu 22.04+, Debian 11+, etc.).
* **Software**: **Docker** must be installed on the host system.
* **Network**: Open TCP ports: **80** and **443** (Inbound and Outbound).

---

### 2. Initial Setup and Configuration

#### Step 2.1 — Create Installation Directory & Download Binary

```bash
mkdir -p /opt/pipe
cd /opt/pipe
curl -L "[https://pipe.network/p1-cdn/releases/latest/download/pop](https://pipe.network/p1-cdn/releases/latest/download/pop)" -o pop
chmod +x pop
````

#### Step 2.2 — Create `.env` Configuration File

Create the required configuration file and replace the placeholder values with your details.

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

-----

### 3\. Network Troubleshooting (Avoid Registration Errors)

This section ensures the node has stable connectivity required for registration and heartbeats.

#### Step 3.1 — Stabilize Host DNS Resolution

Temporarily use reliable public DNS servers to prevent connection failures.

```bash
echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf
echo "nameserver 1.1.1.1" | sudo tee -a /etc/resolv.conf
```

#### Step 3.2 — Configure Local Firewall (`ufw`)

Ensure local ports are open for inbound traffic. Check your **Cloud Firewall/Security Group** to ensure both **INBOUND** (80, 443) and **OUTBOUND** (443) traffic is allowed.

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable  # Enable only if not already enabled and SSH is allowed
sudo ufw status
```

#### Step 3.3 — Test API Connectivity

Verify the server can successfully connect to the API endpoint:

```bash
curl -v [https://api.plpipe.com](https://api.plpipe.com)
```

-----

### 4\. Running the Node with Docker

We use a `Dockerfile` to create a compatible environment (Ubuntu 24.04) and run the node.

#### Step 4.1 — Create the `Dockerfile`

Create the file in your `/opt/pipe` directory:

```bash
nano /opt/pipe/Dockerfile
```

**Content for `Dockerfile`:**

```dockerfile
# Use the required OS as the base
FROM ubuntu:24.04

# Set working directory
WORKDIR /opt/pipe

# Install curl and update package list
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# Download and install the binary (for self-contained image)
RUN curl -L "[https://pipe.network/p1-cdn/releases/latest/download/pop](https://pipe.network/p1-cdn/releases/latest/download/pop)" -o pop
RUN chmod +x pop

# Copy your local .env file into the container
COPY .env .

# Expose the necessary ports
EXPOSE 80 443 8081 9090

# Command to run the node
CMD ["/bin/bash", "-c", "source .env && /opt/pipe/pop"]
```

#### Step 4.2 — Build the Docker Image

Run this command from inside the `/opt/pipe` directory:

```bash
cd /opt/pipe
sudo docker build -t pipe-node .
```

#### Step 4.3 — Run the Container

Run the node, mapping all ports and enabling auto-restart:

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

#### Step 4.4 — Check Logs

Monitor the logs to confirm successful startup and registration:

```bash
sudo docker logs -f pipe-node
```

-----

### 5\. Verification and Monitoring

Use `docker exec` to run internal commands and check your node's status and rewards.

**Check Node Status:**

```bash
sudo docker exec pipe-node /opt/pipe/pop status
```

**Check Node Earnings:**

```bash
sudo docker exec pipe-node /opt/pipe/pop earnings
```

-----

### Support and Community

For further assistance, feel free to join our Telegram channel or reach out directly.

  * **Telegram Channel:** [https://t.me/earningmastermind69](https://t.me/earningmastermind69)
  * **Direct Message:** [@The\_Aotlover9\_bot](https://www.google.com/search?q=https://t.me/The_Aotlover9_bot)

<!-- end list -->

```
```
