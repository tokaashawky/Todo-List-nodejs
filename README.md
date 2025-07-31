# 📝 Todo List Node.js Application – DevOps Implementation

## 📌 Task Overview (Part 1)

This project is a fork of the original [Todo-List-nodejs](https://github.com/Ankit6098/Todo-List-nodejs) repository. It has been extended to include:

- MongoDB database configuration
- Docker containerization
- GitHub Actions CI pipeline
- Image push to a **private Docker registry**

## 🚀 Steps to Reproduce

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/Ankit6098/Todo-List-nodejs.git
cd Todo-List-nodejs
```
### 2️⃣ Configure Your MongoDB
Update the .env file to use your own MongoDB instance:
```bash
MONGODB_URL=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<database>
```

### 3️⃣ Dockerize the Application
A **Dockerfile** has been added to containerize the Node.js app.
Build and Run the Docker Image:
```bash  
  docker build -t tokashawky/todo-list-nodejs:latest -f Docker/Dockerfile .
  # docker run --env-file .env -p 4000:4000 tokashawky/todo-list-nodejs:latest
```

### 4️⃣ CI Pipeline with GitHub Actions
GitHub Actions workflow is configured to:
- Build the Docker image
- Authenticate to Docker Hub
- Push the image to a private Docker registry

#### 🔐 Secrets Required
Add the following secrets to your GitHub repository:
```bash  
  DOCKER_USERNAME – Your Docker Hub username
  DOCKER_PASSWORD – Your Docker Hub access token or password
```
Once pushed to main, GitHub Actions will:
- Build the image
- Push it to docker.io/<your-username>/todo-list-nodejs:latest (private)

## 📌 Ansible Setup on remote Ubuntu EC2 (Part 2)

### 1- Download the SSH Private Key
Create Ec2 and download its SSH Private Key.

### 2- Set Secure Permissions
```bash
chmod 400 ~/.ssh/ToDo_KeyPair.pem
# AWS requires 400 permissions for the key to be used with SSH.
```

### 3- Configure SSH Shortcut from your local machine
```bash 
# Edit your SSH config:
vim ~/.ssh/config
```
Add:
```bash 
host TODO
    hostname Ec2_public_IP
    IdentityFile ToDo_KeyPair.pem
    User ubuntu
    StrictHostKeyChecking no
```

### 4- Test the Connection
```bash
  ssh TODO
```
If it logs in to the EC2 instance, you're ready to use Ansible from your machine:
Make sure **ToDo_KeyPair.pem** inside **Ansible** directory
```bash
cd Ansible
ansible-playbook playbook.yaml
```

### 5- Verify Docker on EC2
```bash
ssh TODO
docker --version
```

## 📌 On the VM, use docker compose to run the application (Part 3)

### 1- Prerequisites on the EC2
Make sure your Ansible playbook (from Part 2) has already done the following:
- Docker & Docker Compose installed
- ubuntu user added to the docker R
- Your app image pushed to Docker Hub (private or public)
- .env file placed at /home/ubuntu/.env

### 2- Upload Compose + .env File
From local:
```bash
scp -i ToDo_KeyPair.pem ../Docker/docker-compose.yaml ubuntu@ec2_public_IP:~
scp -i ToDo_KeyPair.pem ../.env ubuntu@ec2_public_IP:~
```

### 3- Run the Application On the EC2
On EC2: 
```bash 
  # on ubuntu home
  docker login -u tokashawky 
  docker-compose up 
  # Check container status
  docker ps
```
Note: Don't forget
- Whitelist your EC2 public IP in MongoDB Atlas
  Go to your MongoDB Atlas dashboard: Navigate to Network Access, Click “Add IP Address” 
- You need to allow port 4000 in the security group attached to your EC2

### 3- Test auto Update Justification (Watchtower)
To continuously check for changes in the Docker image and pull updates, use Watchtower, a tool that monitors Docker images and automatically updates running containers. Watchtower is chosen for its simplicity, lightweight nature, and ability to handle private registries securely.

1- Add Watchtower to docker-compose.yml
2- Update and Restart Docker Compose:
  - Copy the updated docker-compose.yml to the VM
  - On the VM, restart Docker Compose:
```bash
    docker compose down
    docker compose up -d
```
3- Make a change in your app repo.
4- Rebuild and push the Docker image:
```bash 
  docker build -t tokashawky/todo-list-nodejs:latest .
  docker push tokashawky/todo-list-nodejs:latest
```
Wait 30–60 seconds and on the VM, run:
```bash 
docker ps
```
→ You should see that todo-app was restarted with the new image (check home page).

NOTE:
To have **Secure Approach for Watchtower Credentials**
```bash 
docker login -u tokashawky
cat ~/.docker/config.json
```
Verify the login:You should see an auths section with a base64-encoded token

## 📌 Todo List Node.js application on a Kubernetes cluster (Part 4)

### Step 1: Set Up a Remote EC2 Instance
Launch an EC2 Instance:
- Choose Ubuntu Server 22.04 LTS (HVM, SSD Volume Type).
- Select an instance type (e.g., t3.medium for Kubernetes, with at least 2 vCPUs and 4 GiB RAM).
- Configure:
  Key Pair: Create or use an existing SSH key pair
  Security Group: Allow inbound traffic:
    SSH (port 22) from your local IP.
    HTTP (port 80) and HTTPS (port 443) for application access.
    Port 4000 for the Todo app (temporary for testing).
    Port 6443 for Kubernetes API access.
- Storage: At least 20 GiB for Docker and Kubernetes.
- Launch the instance and note its public IP

### Step 2: Use Ansible from Local Machine to Configure EC2
**Update Ansible Inventory, Ansible playbook and add K8s role**
- Run the Playbook:
```bash
  ansible-playbook playbook.yaml
```
- Verify Kubernetes:
```bash
  kubectl get nodes
```
- Set Up Docker Hub Credentials on EC2
```bash
  docker login -u tokashawky
```

### Step 3: Create Kubernetes Manifests
- Create a GitHub Repository
- Define Kubernetes Manifests
  "Create ConfigMap from .env file"
```bash 
    kubectl create configmap todo-config --from-env-file=.env
```
- Push to GitHub

### Step 4: Install and Configure 
```bash
# Install ArgoCD:
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Verify pods
kubectl get pods -n argocd
``` 
You're running ArgoCD inside an EC2 instance using minikube, and you're using:
kubectl port-forward svc/argocd-server -n argocd 8080:443
This means ArgoCD is only accessible from inside the EC2 instance itself via https://localhost:8080

SSH Tunnel to Forward Port to Your Local Machine
```bash 
ssh -i /path/to/your-key.pem -L 8080:localhost:8080 ubuntu@<EC2-PUBLIC-IP>
```
Then, visit: https://localhost:8080

Login with:
```bash
Username: admin
To get Password run:
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
```


