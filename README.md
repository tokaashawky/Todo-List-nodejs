# 📝 Todo List Node.js Application – DevOps Implementation

## 📌 Task Overview (Part 1)

This project is a fork of the original [Todo-List-nodejs](https://github.com/Ankit6098/Todo-List-nodejs) repository. It has been extended to include:

- MongoDB database configuration
- Docker containerization
- GitHub Actions CI pipeline
- Image push to a **private Docker registry**

---

## 🚀 Steps to Reproduce

### 1️⃣ Clone the Repository

```bash
git clone https://github.com/Ankit6098/Todo-List-nodejs.git
cd Todo-List-nodejs
```
### 2️⃣ Configure Your MongoDB
Update the .env file to use your own MongoDB instance:
```bash
MONGODB_URL=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<database>?retryWrites=true&w=majority
```

### 3️⃣ Dockerize the Application
A **Dockerfile** has been added to containerize the Node.js app.
Build and Run the Docker Image:
```bash
  docker build -t todo-app -f Docker/Dockerfile 
  docker run --env-file .env -p 4000:4000 todo-app
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
If it logs in to the EC2 instance, you're ready to use Ansible:
```bash
ansible-playbook playbook.yaml
```

### 5- Verify Docker on EC2
```bash
ssh TODO
docker --version
```

## 📌 On the VM, use docker compose to run the application (Part 3)


