# Devops-Implementation
# Go Web App Deployment with Docker, Helm, and Argo CD
This project demonstrates how to deploy a Go web application using Docker, Helm, and Kubernetes on an Amazon EKS cluster. It includes a CI/CD pipeline using GitHub Actions and Argo CD for continuous delivery and deployment automation.
## Prerequisites
- AWS CLI installed and configured.
- kubectl installed and configured for EKS cluster.
- Docker installed and configured.
- Helm installed.
- Argo CD installed in cluster.
## Setup Instructions
# 1. Clone the Repository
      git clone https://github.com/----
      cd repo
# 2. Dockerfile
We should  write a Dockerfile for our application. The Dockerfile defines the environment for our application and specifies the steps to build the Docker image.

# 3. Create EKS Cluster
For EKS creation we can follow the AWS documentation for commands or to create an EKS cluster.

              eksctl create cluster --name my-cluster --region ap-south-1
              
(we can change the instance type as per our application requirements by using autoscaling(edit) or else we can also apply configure yaml file)

# 4. Install NGINX Ingress Controller

To manage ingress resources, we need to install the NGINX Ingress Controller in its namespace. Run the following command to deploy it:

           kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
           
To check the NGINX Ingress Controller’s status:

              kubectl get pods --namespace=ingress-nginx
   
            kubectl get svc --namespace=ingress-nginx
   
# 5. Apply Kubernetes Manifests

Deploy the application using Helm charts and Kubernetes manifests. Ensure that the Helm values file is updated with the correct image tag.


       helm upgrade --install go-web-app helm/go-web-app
 
# 6. Install Argo CD

## Install Argo CD using manifests

    kubectl create namespace argocd
    
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml


## Access the Argo CD UI (Loadbalancer service) 


      kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'


## Get the Loadbalancer service IP

         kubectl get svc argocd-server -n argocd

# 7. Continuous Integration and Deployment Automation

## Configuring GitHub Secrets

we need to configure the following secrets in GitHub:

### GitHub Token:
  
- Add a GitHub token to allow the pipeline to commit updates to the repository. This token should be added as a secret in our GitHub repository settings under the name TOKEN.

  ### DockerHub Credentials:
  
- DockerHub Username: Add DockerHub username as a secret in GitHub under the name DOCKERHUB_USERNAME.
  
- DockerHub Token: Generate a token in DockerHub and add it as a secret in GitHub under the name DOCKERHUB_TOKEN.
  
## Code Changes Trigger Pipeline: 

The CI/CD pipeline automates the deployment process as follows:

- Whenever changes are pushed to the GitHub repository, the pipeline is automatically triggered. This process includes:
- Building the Docker Image: The pipeline rebuilds the Docker image with a new tag, which is generated based on the GitHub run ID.
- Pushing the Image: The newly built image is then pushed to DockerHub with the updated tag.
- Updating Helm Chart: The pipeline updates the values.yaml file in the Helm chart with the new image tag and commits this change back to the repository.

- Handling Ignored Paths: In the GitHub Actions workflow, paths related to Helm charts, K8s manifests, and the README are ignored for triggering pipeline runs. This means that changes to these files will not trigger the pipeline. However, if changes occur in other paths, the pipeline will loop and execute the build, push, and deployment processes again.

## Accessing Application

To access deployed application:

- Resolve Ingress IP: The Ingress resource uses a DNS name assigned to the LoadBalancer service. Perform an nslookup on this DNS name to retrieve the IP addresses.

- Update /etc/hosts: Choose one of the IP addresses returned by nslookup and add it to local /etc/hosts file along withe host name:
- 
      <IP_ADDRESS> go-web-app.local
  
- Browse the Application: Navigate to http://go-web-app.local/courses in web browser to view application.
- This setup ensures that any updates to the codebase trigger a full rebuild and redeployment, keeping application up-to-date seamlessly.

## Argo CD Synchronization

After the pipeline rebuilds and redeploys the application, Argo CD will automatically detect the changes. It will sync with the latest commit ID from GitHub, ensuring that the application is updated with the latest changes. This synchronization process keeps the deployed application in the cluster aligned with the state defined in the GitHub repository.

Note: Argo CD relies on the Git repository as the source of truth for the desired state of the application. If changes are made directly in the Kubernetes cluster instead of updating the Git repository, Argo CD will not recognize or accept these changes. The cluster state will be overwritten during the next synchronization cycle to match the state defined in the repository.

