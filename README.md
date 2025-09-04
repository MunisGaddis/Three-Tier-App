Three Tier Application Deployment on Kubernetes

git clone https://github.com/LondheShubham153/TWSThreeTierAppChallenge.git

cd TWSThreeTierAppChallenge
cd Application-Code
cd frontend 

Now Create Reposiry name three-tier-frontend in Amazon ECR
Click view push commands
Retrieve an authentication token and authenticate your Docker client to your registry. Use the AWS CLI:
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 585768171289.dkr.ecr.ap-south-1.amazonaws.com

Note: If you receive an error using the AWS CLI, make sure that you have the latest version of the AWS CLI and Docker installed.

Build your Docker image using the following command. For information on building a Docker file from scratch see the instructions here .
docker build -t three-tier-frontend .

After the build completes, tag your image so you can push the image to this repository:
docker tag three-tier-frontend:latest 585768171289.dkr.ecr.ap-south-1.amazonaws.com/three-tier-frontend:latest

Run the following command to push this image to your newly created AWS repository:
docker push 585768171289.dkr.ecr.ap-south-1.amazonaws.com/three-tier-frontend:latest

Now cd ...
cd backend 

Retrieve an authentication token and authenticate your Docker client to your registry. Use the AWS CLI:
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 585768171289.dkr.ecr.ap-south-1.amazonaws.com

Note: If you receive an error using the AWS CLI, make sure that you have the latest version of the AWS CLI and Docker installed.
Build your Docker image using the following command. For information on building a Docker file from scratch see the instructions here .
docker build -t three-tier-backend .

After the build completes, tag your image so you can push the image to this repository:
docker tag three-tier-backend:latest 585768171289.dkr.ecr.ap-south-1.amazonaws.com/three-tier-backend:latest

Run the following command to push this image to your newly created AWS repository:
docker push 585768171289.dkr.ecr.ap-south-1.amazonaws.com/three-tier-backend:latest

******************************************************************************************************************************************************
Install kubectl
curl -o kubectl https://amazon-eks.s3.ap-south-1.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client

Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

Setup EKS Cluster
eksctl create cluster --name three-tier-cluster --region ap-south-1 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region ap-south-1 --name three-tier-cluster
kubectl get nodes
**************************************************************************************************************************************************************************************************
cd cd TWSThreeTierAppChallenge
cd Kubernetes-Manifests-file

Run Manifests
kubectl create namespace workshop
Before running backend run mongo deployment first
kubectl apply -f .
kubectl delete -f .

Install AWS Load Balancer
# Step 1: Download the IAM policy JSON
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

# Step 2: Create the IAM policy
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json

# Step 3: Associate OIDC provider with your EKS cluster
eksctl utils associate-iam-oidc-provider \
  --region=ap-south-1 \
  --cluster=three-tier-cluster \
  --approve

# Step 4: Create IAM service account for AWS Load Balancer Controller
eksctl create iamserviceaccount \
  --cluster=three-tier-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::626072240565:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --region=ap-south-1

Deploy AWS Load Balancer Controller
Install Helm & Add Repo
sudo snap install helm --classic

helm repo add eks https://aws.github.io/eks-charts
helm repo update

Install AWS Load Balancer Controller
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=three-tier-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller

Step 6: Verify Deployment
kubectl get deployment -n kube-system aws-load-balancer-controller

✅ You should see replicas running (e.g., 1/1 or 2/2).

If not, check logs:
kubectl logs -n kube-system deployment/aws-load-balancer-controller

Step 7: Apply Your Ingress/Service
kubectl apply -f full_stack_lb.yaml
************************************************************************************************************************************************************************************************
