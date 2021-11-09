

# Azure Voting App -  Rolling deployment 
This sample creates a multi-container application in an Azure Kubernetes Service (AKS) cluster. It has been forked from https://github.com/Azure-Samples/azure-voting-app-redis and used in this repo for personal education purpose.  
The application interface has been built using Python / Flask. The data component is using Redis.


# nd9991-Capstone
Capstone project solution. ND9991 Cloud DevOps using AWS.  Here is the directory structure of the curreent repo:
```bash
.
├── LICENSE
├── README.md
├── azure-vote                  # Multi-container application 
│   ├── Dockerfile              # Dockerfile that encapsulates the azure-vote app
│   ├── Makefile                # For setup, installation, and linting
│   ├── app_init.supervisord.conf
│   ├── azure-vote
│   │   ├── config_file.cfg
│   │   ├── main.py
│   │   ├── static
│   │   │   └── default.css
│   │   └── templates
│   │       └── index.html
│   └── requirements.txt        # package dependencies
├── docker-compose.yaml         # To test the app locally
└── voting-app.yaml             # To deploy the app on k8s cluster
```

## Step 0. Prerequisites
1. AWS CLI should be configured locally.
2. An AWS EKS cluster should be in place. I have already create a cluster using EKSCTL utility, which internally uses Cloudformation. Alternatively, it can be created using AWS web console. 
```bash
# Install EKSCTL on Mac
# Check Homebrew 
brew --version
# If you do not have Homebrew installed - https://brew.sh/ 
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
# Install eksctl
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl
# Assming eksctl is installed in the local
eksctl create cluster --name my-eks-demo --region=us-east-2
```
3. kubectl should also be installed locally. See the instructions [here](https://kubernetes.io/docs/tasks/tools/). 
4. Create two repos in Docker hub - `voting-app-backend` and `voting-app-frontend`. 


## Step 1. Test the application locally [Optional]
```bash
# Run from the project directory
docker-compose up -d
```
The command above will fetch the public images:
- mcr.microsoft.com/oss/bitnami/redis
- mcr.microsoft.com/azuredocs/azure-vote-front 

and create the following containers correspondingly:
- `azure-vote-back` - It is a Redis backend.
- `azure-vote-front` - It is a Flask frontend. 

Later, I have retagged the local images to match my Dockerhub repo name. 
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


Once the containers are up and running, the application can be accessed at http://localhost:8080 locally.  To stop the running containers, run:
```bash
docker-compose down
```


## Step 2. CircleCI Pipeline
There are three prime jobs in the CircleCI config file:
1. **build-application** - This job installs the required dependeencies, and lints the application. It basically runs:
```bash
cd azure-vote
# Check the azure-vote/Makefile
# Installs dependencies mentioned in the requirements.txt
make install
# Lints the Dockerfile and the main Python file
make lint
```


2. **push-images** - Once the *build-application* job succeeds, the CircleCI will rebuild the Docker image (backend only) and push to the Dockerhub using the following code:
```bash
cd azure-vote 
docker build . -t sudkul/voting-app-frontend:latest          
docker image ls
echo "$DOCKER_PASS" | docker login --username $DOCKER_USER --password-stdin
docker push sudkul/voting-app-frontend:latest
```


3. **deploy-application** - Once the *push-images* job succeeds, this job will install kubectl, aws cli, install IAM authnticator for kubernetes, update kubectl configuration file to use the IAM authnticator for kubernetes, and finally deploy the application using the `voting-app.yaml` file using these commands:
```bash
kubectl get services
ls -l
kubectl apply -f voting-app.yaml
kubectl get pods,svc,deployments
``` 