# Mini-Project: Deploy a Cloud Service Using Docker on Ubuntu

## Objective

To set up and deploy a simple web server using Apache within a Docker container on an Ubuntu server.

## Prerequisites

- Ubuntu Server (18.04 or later recommended).
- Basic knowledge of Linux commands.
- A user account with sudo privileges.

---

## Step 1: Update the System

Start by updating your package index to ensure your system has the latest updates:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Step 2: Install Docker

### 2.1 Install Docker Prerequisites

```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```

### 2.2 Add Docker's Official GPG Key

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### 2.3 Set Up Docker Repository

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 2.4 Install Docker

```bash
sudo apt update
sudo apt install docker-ce -y
```

### 2.5 Verify Docker Installation

```bash
docker --version
```

### 2.6 Start Docker and Enable on Boot

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

---

## Step 3: Deploy an Apache Web Server Container

### 3.1 Pull the Official Apache HTTP Server Image

```bash
docker pull httpd:latest
```

### 3.2 Run the Apache Container

```bash
docker run -dit --name apache-server -p 8080:80 httpd:latest
```

**Explanation:**

- `--name apache-server`: Names the container `apache-server`.
- `-p 8080:80`: Maps port 80 in the container to port 8080 on the host.
- `-dit`: Runs the container in detached mode with an interactive terminal.

### 3.3 Verify Container is Running

```bash
docker ps
```

---

## Step 4: Test the Web Server

### 4.1 Access the Web Server

Open a browser and navigate to:

```
http://<server-ip>:8080
```

Replace `<server-ip>` with your Ubuntu server's IP address.

### 4.2 Find Your Server's IP Address

If you do not know your server's IP, you can find it using:

```bash
ip a
```

You should see the default Apache web page when the server is running correctly.

---

## Step 5: Managing the Container

### 5.1 View Running Containers

```bash
docker ps
```

### 5.2 Stop the Container

```bash
docker stop apache-server
```

### 5.3 Restart the Container

```bash
docker start apache-server
```

### 5.4 Remove the Container

```bash
docker rm apache-server
```

### 5.5 Remove the Image

```bash
docker rmi httpd:latest
```

---

## Advanced Configuration (Optional)

### Serve Custom Content

Create a directory for your custom content and add a sample HTML file:

```bash
mkdir -p ~/apache-content
echo "<h1>Welcome to My Dockerized Apache Server</h1>" | sudo tee ~/apache-content/index.html
```

### Volume Permissions

If you're using a custom volume for your Apache content, ensure that the permissions are correctly set.

**Check Ownership and Permissions:**

```bash
ls -ld ~/apache-content
```

The directory should be owned by the `www-data` user (inside the container). To fix ownership and permissions, run:

```bash
sudo chown -R 33:33 ~/apache-content
sudo chmod -R 755 ~/apache-content
```

> **Note:** `33:33` corresponds to `www-data` (UID and GID) used by Apache inside the container. The `755` permission ensures the directory and its files are readable by the server.

**Restart the Container with Custom Content:**

```bash
docker stop apache-server
docker rm apache-server
docker run -dit --name apache-server -p 8080:80 -v ~/apache-content:/usr/local/apache2/htdocs/ httpd:latest
```

**Test the Web Server Again:**
Visit `http://<server-ip>:8080` to see your custom content.

---

## Troubleshooting

### Check Docker Service Status

```bash
sudo systemctl status docker
```

### View Container Logs

```bash
docker logs apache-server
```

### Check Port Availability

```bash
sudo netstat -tulpn | grep 8080
```

### Firewall Configuration

If you cannot access the web server, ensure the firewall allows port 8080:

```bash
sudo ufw allow 8080/tcp
sudo ufw reload
```

---

## Conclusion

🎉 **Congratulations!** You have successfully deployed an Apache web server using Docker on Ubuntu.

### What's Next?

Here are some suggestions for what you can explore next to enhance your knowledge and expand on this project:

1. **Container Orchestration**: Learn Kubernetes or Docker Swarm for managing multiple containers.
2. **CI/CD Integration**: Set up automated builds and deployments using GitHub Actions or Jenkins.
3. **Container Security**: Implement security best practices and scan for vulnerabilities.
4. **Application Stack Deployment**: Deploy a multi-tier application with a database and backend service.
5. **Docker Compose**: Use Docker Compose to define and run multi-container applications.
6. **Monitoring**: Implement container monitoring with Prometheus and Grafana.
7. **Container Networking**: Explore advanced Docker networking configurations.
8. **Persistent Volumes**: Learn about Docker volumes for persistent storage.
9. **Custom Docker Images**: Create your own Docker images using Dockerfiles.
10. **Cloud Integration**: Deploy your Docker containers to cloud services like AWS ECS or Azure Container Instances.

---

## Resources

- [Docker Official Documentation](https://docs.docker.com/)
- [Apache HTTP Server Documentation](https://httpd.apache.org/docs/)
- [Ubuntu Server Documentation](https://ubuntu.com/server/docs)
- [Docker Hub](https://hub.docker.com/)

---

## License

This project is open-source and available under the MIT License.

## Support

For issues or questions, please create an issue in the repository or consult the Docker community forums.
