

# Azure Voting App -  Rolling deployment 
This sample creates a multi-container application in an Azure Kubernetes Service (AKS) cluster. It has been forked from https://github.com/Azure-Samples/azure-voting-app-redis and used in this repo for personal education purpose.  
The application interface has been built using Python / Flask. The data component is using Redis.



# nd9991-Capstone
Capstone project solution. ND9991 Cloud DevOps using AWS. 
## Step 1. Test the application locally
```bash
docker-compose up -d
```

The command above will fetch the public images:
1. mcr.microsoft.com/oss/bitnami/redis
2. mcr.microsoft.com/azuredocs/azure-vote-front 

and create the following containers correspondingly:
1. `azure-vote-back` - It is a Redis backend.
2. `azure-vote-front` - It is a Flask frontend. 

Once the containers are up and running, the application can be accessed at http://localhost:8080 locally.  To stop the running containers, run:
```bash
docker-compose down
```


## Step 2. Push the built Docker containers (from Local) to the Dockerhub repositories

1. Create two repos in Docker hub - `voting-app-backend` and `voting-app-frontend`. Now, tag the local image per the Docker hub repo naming convention. 
```bash
# Tag the backend image
docker tag mcr.microsoft.com/oss/bitnami/redis:6.0.8 sudkul/voting-app-backend:latest
# Tag the frontend image
docker tag mcr.microsoft.com/azuredocs/azure-vote-front:v1 sudkul/voting-app-frontend:latest
# Check the new tags 
docker image ls
# Login and push the images to the Docker hub
docker login --username=sudkul --email=sudhanshu.kulshrestha@gmail.com
docker push sudkul/voting-app-backend:latest
docker push sudkul/voting-app-frontend:latest
```

## Step 3. Deploying the Docker containers (from Dockerhub) to a Kubernetes cluster. 
>Deploying these Docker container(s) to a small Kubernetes cluster. For your Kubernetes cluster you can either use AWS Kubernetes as a Service, or build your own Kubernetes cluster. To deploy your Kubernetes cluster, use either Ansible or Cloudformation. Preferably, run these from within Jenkins or Circle CI as an independent pipeline.


After an edit is made to the repo, the CircleCI should
- Lint the application code
- Test the EKS cluster
- Repackage the image and push to the Dockerhub
- Deploy the application

## [NOT WORKING] Creating a Kubernetes cluster in AWS EC2 instance
### 1. Install Docker
```bash
# Installing Docker in Amazon Linux 2
sudo yum update -y
sudo amazon-linux-extras install docker -y
sudo yum install git -y
# Start the Docker service
sudo service docker start
# Add the ec2-user to the docker group
sudo usermod -a -G docker ec2-user
# Log out and log back in for the group permission to take effect.
# Verify that the ec2-user can run Docker commands without sudo
docker --version
# Install Docker-compose
# Refer to https://docs.docker.com/compose/install/
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
git clone https://github.com/SudKul/nd9991-Capstonoe.git
cd nd9991-Capstonoe/
```

### [Optional] Run the app in the Docker containers (only for testing purpose)
```bash
# Pull the images from Dockerhub
docker pull sudkul/voting-app-backend:latest
docker pull sudkul/voting-app-frontend:latest
docker-compose up -d
# Access the application at [Public DNS]:8080
docker-compose down
```
Open a [Public DNS]:8080 in the browser tab in your local machine to access the application. 


### 2. Install kubectl binary with curl on Linux 
[Reference](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux)
```bash
# Download the latest release with the command
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
# Install kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```
### 3. Install minikube
```bash
# To install the latest minikube stable release on x86-64 Linux using binary download:
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
# Start the cluster. Refer https://minikube.sigs.k8s.io/docs/start/ for more details
minikube start
# Deploying the application using the template
kubectl apply -f voting-app.yaml
kubectl get pods,svc,deployments
```
Use the EXTERNAL-IP address of the Load balancer to access the application from your local machine, provided the security group of  the EC2 instance allows inbound access on port 80 and 8080



NOT WORKING
```bash
docker run -it --name voting-app-backend  -p 6379:6379 -e ALLOW_EMPTY_PASSWORD="yes" sudkul/voting-app-backend:latest
docker run -it --name voting-app-frontend -p 8080:80 -e REDIS=azure-vote-back sudkul/voting-app-frontend:latest 
````