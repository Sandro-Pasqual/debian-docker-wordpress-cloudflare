# Tutorial: Setting Up a Debian 12 Server with Docker, WordPress, MySQL, and Cloudflare

## Prerequisites
- A Debian 12 server (with SSH access as a `user` or another user with sudo privileges).
- A registered domain (e.g., `yourdomain.com`) configured in Cloudflare.
- Internet access on the server.

# 1. Installing Docker on Debian 12

## Installation Steps
1. Update the system packages:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
   *(This updates all packages and system components on the server.)*

2. Install the necessary packages for Docker:
   ```bash
   sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
   ```

3. Add the official GPG key for Docker:
   ```bash
   curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

4. Add the Docker repository:
   ```bash
   echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

5. Update the packages again and install Docker:
   ```bash
   sudo apt update
   sudo apt install -y docker-ce docker-ce-cli containerd.io
   ```

6. Verify that Docker is installed correctly:
   ```bash
   docker --version
   ```
   *You should see something like `Docker version 20.10.x`. If you get `command not found`, review the previous steps.*

**Common Error: `docker: command not found`**  
- **Cause**: Docker was not installed correctly or the PATH is not set.  
- **Solution**: Repeat the steps above or run `sudo systemctl restart docker` and check again.

# 2. Starting WordPress and MySQL Containers

## Install Docker Compose
Docker Compose simplifies managing containers. Install it with:
```bash
sudo apt install -y docker-compose
```

## Create and Configure the Project
1. Create a directory for the project and navigate to it:
   ```bash
   mkdir ~/wordpress-project
   cd ~/wordpress-project
   ```

2. Create and edit the `docker-compose.yml` file:
   ```bash
   nano docker-compose.yml
   ```
   *(Use the `nano` editor: type the content below, then save with `Ctrl+O`, Enter, and exit with `Ctrl+X`.)*

Add the following content to `docker-compose.yml`:
```yaml
version: '3'

services:
  wordpress:
    image: wordpress:latest
    ports:
      - "80:80"
    networks:
      - wordpress-network
    volumes:
      - wordpress_data:/var/www/html
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: secret_password
      WORDPRESS_DB_NAME: wordpress

  db:
    image: mysql:5.7
    networks:
      - wordpress-network
    volumes:
      - db_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: secret_password

networks:
  wordpress-network:
    driver: bridge

volumes:
  wordpress_data:
  db_data:
```

3. Start the containers in the background:
   ```bash
   docker-compose up -d
   ```
   *(The `-d` flag runs containers in the background, keeping your terminal free.)*

4. Verify that the containers are running:
   ```bash
   docker ps
   ```
   *You should see `wordpress-project-wordpress-1` and `wordpress-project-db-1` with a status of `Up`.*

**Common Error: `ERROR: No such service`**  
- **Cause**: The `docker-compose.yml` file contains a syntax error or incorrect names.  
- **Solution**: Check the `docker-compose.yml` file for errors (use spaces, not tabs) and run `docker-compose up -d` again.

# 3. Installing and Configuring Cloudflared

## Install `wget` (if not already installed)
```bash
sudo apt update && sudo apt install -y wget
```

## Download and Install `cloudflared`
```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
```

## Verify the Installation
```bash
cloudflared --version
```
*You should see something like `cloudflared version 2025.x.x`. If you get `command not found`, check the `wget` installation and retry the steps.*

**Common Error: `bash: wget: command not found`**  
- **Cause**: `wget` is not installed.  
- **Solution**: Run `sudo apt install -y wget` and try again.

# 4. Configuring the Cloudflare Tunnel

## Create the Tunnel in Cloudflare
1. Log in to your Cloudflare account, go to **Zero Trust** > **Tunnels**.
2. Click „Create a tunnel” and name it `wordpress-tunnel`.
3. Copy the generated token (e.g., `eyJhIjoi...`).

## Run the Tunnel with Docker
Ensure the tunnel uses the common network:
```bash
docker run -d --name cloudflared-tunnel --network wordpress-network cloudflare/cloudflared:latest tunnel --no-autoupdate run --token <TOKEN-HERE>
```
*Replace `<TOKEN-HERE>` with your token from Cloudflare.*

## Configure a Public Hostname
1. In Cloudflare, under the `wordpress-tunnel`, go to „Public Hostnames”.
2. Add `yourdomain.com` as a public hostname.
3. Set „Service” to `http://wordpress-project-wordpress-1:80`.
4. Save the changes.

## Verify the DNS
- Ensure you have a CNAME record for `yourdomain.com` pointing to `your-tunnel-id.cfargotunnel.com` (or your tunnel ID), with proxy enabled („Proxied”).

**Common Error: `502 Bad Gateway`**  
- **Cause**: The tunnel cannot reach WordPress (possibly due to network issues or an incorrect route).  
- **Solution**: 
  - Verify if WordPress is running with `curl http://localhost:80`.
  - Ensure the tunnel and WordPress are in the `wordpress-network` network.
  - Check the tunnel logs with `docker logs cloudflared-tunnel`.

# 5. Troubleshooting Common Errors

- **Connection Refused**: Check if port 80 is correctly mapped in the WordPress container (`ports: - "80:80"` in `docker-compose.yml`) and if the container is running.
- **Unable to find image 'cloudflare/cloudflared:latest' locally**: Docker automatically downloads the image, but if you have a slow connection, wait a few minutes or check your internet.

# 6. Final Testing
- Access `yourdomain.com` in a browser.
- Check stability with:
  ```bash
  docker ps
  ```
- Check container logs (if issues arise):
  ```bash
  docker logs cloudflared-tunnel
  docker logs wordpress-project-wordpress-1
  ```

# 7. Enabling HTTPS with Cloudflare (Optional)
- Go to Cloudflare > **SSL/TLS** > **Overview**.
- Set „SSL/TLS encryption mode” to „Full (strict)” (recommended).
- Cloudflare will automatically generate free Let’s Encrypt SSL certificates for `yourdomain.com` via the tunnel.
- Verify the site with `https://yourdomain.com` – you should see a green lock in the browser.

# Final Notes
- **Backup**: Create backups of your Docker containers and WordPress/MySQL data (use volumes or export databases).
- **Updates**: Periodically check for updates to Docker, WordPress, MySQL, and `cloudflared`.
- **Portainer**: Use Portainer (on port 9000) to manage containers more easily in the future.

---
