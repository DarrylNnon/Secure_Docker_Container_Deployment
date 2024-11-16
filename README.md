# Secure Docker Container Deployment
Securing Docker container deployment is a critical task in DevSecOps to ensure containerized applications are robust against vulnerabilities. This project involves building a secure containerized application, scanning images for vulnerabilities, applying best-practice security configurations, and securely deploying to a Docker registry. Below is a real-world, step-by-step guide to successfully complete this project and train your team.


## Project: **Secure Docker Container Deployment**

### Steps:
1. **Build a containerized application using Docker**.
2. **Implement image scanning** (e.g., using Anchore or Clair).
3. **Apply security configurations** (e.g., rootless containers).
4. **Test and deploy securely to a Docker registry**.


### Prerequisites
1. **Docker Installed** on your workstation/server.
2. **Docker Hub Account** or private Docker registry like AWS ECR or GitHub Container Registry.
3. **Basic Docker Knowledge**: Understanding of Dockerfiles, images, and containers.
4. **Security Scanning Tools**: Install Anchore CLI or Clair scanner.

### Step 1: Build a Containerized Application Using Docker

We’ll containerize a simple Python application.

#### Step 1.1: Create the Application

Create a directory for your project:

```bash
mkdir secure-docker-app && cd secure-docker-app
```

Create a Python file, `app.py`:

```python
# app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Hello, Secure Docker!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

Create a requirements file, `requirements.txt`:

```plaintext
flask==2.2.3
```

#### Step 1.2: Write a Dockerfile

Create a `Dockerfile`:

```dockerfile
# Use a minimal base image
FROM python:3.10-slim

# Set working directory
WORKDIR /app

# Copy application files
COPY app.py requirements.txt /app/

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose the application port
EXPOSE 5000

# Run the application
CMD ["python", "app.py"]
```

#### Step 1.3: Build the Docker Image

Build the container image:

```bash
docker build -t secure-docker-app:1.0 .
```

Verify the image was built:

```bash
docker images
```

#### Step 1.4: Run the Container

Run the container locally:

```bash
docker run -d -p 5000:5000 secure-docker-app:1.0
```

Test the application:

```bash
curl http://localhost:5000
```

You should see: `Hello, Secure Docker!`

### Step 2: Implement Image Scanning (e.g., Anchore or Clair)

Image scanning identifies vulnerabilities in your container image.

#### Step 2.1: Install Anchore CLI (Example)

Install Anchore CLI:

```bash
pip install anchorecli
```

Set up Anchore Engine (locally or as a cloud service). For local use, deploy it via Docker Compose (Anchore's official docs have details).

#### Step 2.2: Scan the Image

Add your image to Anchore for scanning:

```bash
anchore-cli image add secure-docker-app:1.0
```

Run the scan:

```bash
anchore-cli image wait secure-docker-app:1.0
anchore-cli image vuln secure-docker-app:1.0 all
```

**Output Example**:
- Vulnerabilities are categorized as `High`, `Medium`, or `Low`.
- Address critical vulnerabilities by updating your `Dockerfile` (e.g., using the latest dependencies).

#### Step 2.3: Automate Scanning in CI/CD

Integrate Anchore or Clair into your CI/CD pipeline using tools like Jenkins, GitHub Actions, or GitLab CI.

**GitHub Actions Example**:
Create `.github/workflows/docker-scan.yml`:

```yaml
name: Docker Image Scan

on:
  push:
    branches:
      - main

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Login to DockerHub
      run: echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin

    - name: Build Docker Image
      run: docker build -t secure-docker-app:1.0 .

    - name: Scan Image with Anchore
      run: |
        pip install anchorecli
        anchore-cli image add secure-docker-app:1.0
        anchore-cli image wait secure-docker-app:1.0
        anchore-cli image vuln secure-docker-app:1.0 all
```

### Step 3: Apply Security Configurations (e.g., Rootless Containers)

#### Step 3.1: Use Rootless Containers

Running containers as root increases the attack surface. Convert your Docker installation to run in **rootless mode**.

**Enable Rootless Docker**:
1. Follow the official Docker guide for enabling rootless mode:
   ```bash
   dockerd-rootless-setuptool.sh install
   ```
2. Confirm rootless Docker:
   ```bash
   docker info | grep Rootless
   ```

#### Step 3.2: Use Non-Root User in Dockerfile

Modify the `Dockerfile` to create a non-root user:

```dockerfile
FROM python:3.10-slim

# Create non-root user
RUN useradd -ms /bin/bash appuser
USER appuser

# Set working directory and copy files
WORKDIR /app
COPY app.py requirements.txt /app/

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose port and run the app
EXPOSE 5000
CMD ["python", "app.py"]
```

Rebuild and run the container. Confirm it runs as `appuser`:

```bash
docker exec -it <container_id> whoami
```

#### Step 3.3: Enable Seccomp and AppArmor Profiles

1. **Default Seccomp Profile**: Docker uses a default seccomp profile to restrict syscalls. Customize it if needed.
2. **AppArmor Profile**: Enable AppArmor profiles to restrict container behavior.

### Step 4: Test and Deploy Securely to a Docker Registry

#### Step 4.1: Test the Container Locally

Run security testing tools like **Docker Bench for Security**:

```bash
docker run -it --net host --pid host --cap-add audit_control \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --label docker_bench_security \
  docker/docker-bench-security
```

Review the output and address any recommendations.

#### Step 4.2: Push to a Secure Docker Registry

Push the secure image to Docker Hub (or another registry).

1. Login to Docker Hub:

```bash
docker login
```

2. Tag and push the image:

```bash
docker tag secure-docker-app:1.0 <your_dockerhub_username>/secure-docker-app:1.0
docker push <your_dockerhub_username>/secure-docker-app:1.0
```

3. Verify the image in the registry.

#### Step 4.3: Set Up Registry Scanning

Enable registry scanning (e.g., Docker Hub has a vulnerability scanner for Pro accounts).

#### Step 4.4: Deploy Using Orchestrators

Deploy the container securely using Kubernetes or Docker Swarm. Add security policies (e.g., Pod Security Policies in Kubernetes).

### Step 5: Document the Secure Deployment

Create documentation for your setup:

#### **Secure Docker Deployment Documentation**

**1. Overview**  
This document details the process for securely building, scanning, and deploying Docker containers.

**2. Steps**  
- Build:  
  ```bash
  docker build -t secure-docker-app:1.0 .
  ```
- Scan:  
  ```bash
  anchore-cli image vuln secure-docker-app:1.0 all
  ```
- Secure Configuration:
  - Rootless mode: Enabled.
  - Non-root user in Dockerfile.
- Push to Registry:
  ```bash
  docker push <your_dockerhub_username>/secure-docker-app:1.0
  ```

**3. Tools Used**  
- Docker
- Anchore CLI
- Docker Bench for Security

**4. Training Material**  
Provide a hands-on workshop for team members to follow this process using your documentation and scripts.


By completing these steps, you will have implemented a secure Docker container deployment process that you can replicate and train your team on.
