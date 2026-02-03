# Dockerizing Project examples

## Option 1: In AWS ECR
**Key Steps:**
A. **Dockerize App** - Create Dockerfile with Gunicorn and production settings
B. **Push to Amazon ECR** - Create ECR repository and push your Docker image
C. **Create EKS cluster** - Use eksctl to provision a managed Kubernetes cluster
D. **Deploy to Kubernetes** - Set up deployments, services, secrets, and optional PostgreSQL database
E. **Configure ingress** - Optional AWS Load Balancer Controller for external access
F. **Run migrations** - Execute Django management commands in the cluster
G. **CI/CD setup** - Optional GitHub Actions workflow for automated deployments

### 1A. Dockerize App
1. As a precondition, install and start docker (maybe also enable it on machine startup).
```shell
sudo dnf upgrade --releasever=2023.9.20251110
sudo dnf install docker -y
sudo systemctl start docker
#sudo systemctl enable docker
sudo usermod -aG docker $USER   # Adds user to docker group (to no longer need sudo command)
sudo mkdir -p /usr/libexec/docker/cli-plugins
sudo curl -SL "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-$(uname -m)" -o /usr/libexec/docker/cli-plugins/docker-compose
sudo chmod +x /usr/libexec/docker/cli-plugins/docker-compose
```
2. Build container ex. "django-app" then test building it with a mariadb docker image database.
```shell
docker build -t django-app .
#docker run -p 8000:8000 django-app
docker compose up --build
```

### 1B. Push to Amazon ECR
1. If project built successfully,

### 1C. Create EKS Cluster
1. Create cluster
```shell
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed
```
```shell
OR using AWS CLI
aws eks create-cluster \
  --name my-cluster \
  --role-arn arn:aws:iam::ACCOUNT-ID:role/EKSClusterRole \
  --resources-vpc-config subnetIds=subnet-xxx,subnet-yyy,securityGroupIds=sg-xxx
```
2. Configure Kubernetes
```shell
aws eks update-kubeconfig --name my-cluster --region us-east-1
```
3. Verify connection
```shell
kubectl get nodes
```


TBD


### 1Z.
```shell
# (If private) Create Kubernetes secret
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=yourusername \
  --docker-password=yourpassword \
  --docker-email=youremail@example.com

# 5. Deploy to Kubernetes
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# 6. Verify deployment
kubectl get pods
kubectl get service my-app-service
```



## Option 2: In Docker Hub
TBD
