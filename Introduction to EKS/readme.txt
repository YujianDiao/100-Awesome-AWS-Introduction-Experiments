# Update IAM settings in IDE
rm -vf ${HOME}/.aws/credentials
aws sts get-caller-identity --query Arn

# Update AWS CLI
sudo pip install --upgrade awscli
aws --version

# Update AWS CLI to version 2
sudo yum erase awscli
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws --update


# Install kubectl
sudo curl -o /usr/local/bin/kubectl  \
   https://amazon-eks.s3.us-west-2.amazonaws.com/1.21.2/2021-07-05/bin/linux/amd64/kubectl
sudo chmod +x /usr/local/bin/kubectl
kubectl version --client=true --short=true

# Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv -v /tmp/eksctl /usr/local/bin
eksctl version


# Create Container Image
cd ~/environment/
docker build -t test-image . 
docker images
docker run -p 8080:80 --name test-nginx test-image
docker ps
docker logs -f test-nginx
docker exec -it test-nginx /bin/bash
docker stop test-nginx
docker rm test-nginx
docker rmi test-image


# Create Amazon ECR Repository and Upload Image
aws ecr create-repository \
--repository-name game-2048 \
--image-scanning-configuration scanOnPush=true \
--region ${AWS_REGION}


# Create EKS Cluster
eksctl create cluster -f eks-demo-cluster.yaml
aws eks --region us-east-1 update-kubeconfig --name eks-demo 
eksctl create nodegroup --cluster=eks-demo

# Create microservice
kubectl apply -f game-2048.yaml
kubectl port-forward service/service-2048 8080:80 -n game-2048

# Delete EKS Cluster
eksctl delete cluster -f eks-demo-cluster.yaml 
