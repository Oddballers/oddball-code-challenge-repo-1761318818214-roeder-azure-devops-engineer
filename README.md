# oddball-code-challenge-repo-1761318818214-roeder-azure-devops-engineer

# Coding Challenge

## Problem Description
Hello Gene, this coding challenge is for the Azure Devops Engineer position and is designed to assess your skills at an intermediate level. Your task is to set up an Azure DevOps pipeline that integrates with Terraform to manage a containerized application hosted on Azure.

## Requirements
1. Create a Terraform script that provisions an Azure Container Registry (ACR) and deploys a simple Docker container to Azure Kubernetes Service (AKS).
2. Write an Azure DevOps YAML pipeline that automates the build and deployment process using the Terraform script.
3. Ensure that the pipeline includes stages for building the Docker image, pushing it to ACR, and deploying it to AKS.
4. Implement logging and error handling in the pipeline.

## Technical Specifications  
- Use Terraform for provisioning Azure resources.
- The container image should be based on a simple Node.js application.
- The pipeline should use Azure DevOps YAML syntax.
- The challenge should be completable in 90 minutes.

## Starter Files

### File 1: `main.tf`
```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
}

resource "azurerm_container_registry" "example" {
  name                = "exampleacr12345"  # Change to a unique name
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  sku                 = "Basic"
  admin_enabled       = true
}

resource "azurerm_kubernetes_cluster" "example" {
  name                = "exampleaks"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  dns_prefix          = "exampleaks"

  default_node_pool {
    name       = "default"
    node_count = 1
    vm_size   = "Standard_DS2_v2"
  }

  identity {
    type = "SystemAssigned"
  }
}
```

### File 2: `azure-pipelines.yml`  
```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  containerRegistry: 'exampleacr12345'
  imageName: 'node-app'

stages:
- stage: Build
  jobs:
  - job: BuildImage
    steps:
    - script: |
        docker build -t $(containerRegistry).azurecr.io/$(imageName):$(Build.BuildId) .
        echo "Image built successfully"
      displayName: 'Build Docker Image'

- stage: Push
  jobs:
  - job: PushImage
    steps:
    - script: |
        echo $(DOCKER_PASSWORD) | docker login $(containerRegistry).azurecr.io -u $(DOCKER_USERNAME) --password-stdin
        docker push $(containerRegistry).azurecr.io/$(imageName):$(Build.BuildId)
        echo "Image pushed successfully"
      displayName: 'Push Docker Image'

- stage: Deploy
  jobs:
  - job: DeployToAKS
    steps:
    - script: |
        az aks get-credentials --resource-group example-resources --name exampleaks
        kubectl apply -f deployment.yaml
        echo "Application deployed successfully"
      displayName: 'Deploy to AKS'
```

### File 3: `deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app
    spec:
      containers:
      - name: node-app
        image: exampleacr12345.azurecr.io/node-app:latest
        ports:
        - containerPort: 3000
```

## Sample Data (if applicable)
No sample data is required for this challenge.

## Evaluation Criteria
- Correctness of the Terraform script and YAML pipeline.
- Clarity and organization of the code.
- Proper error handling and logging in the pipeline.
- Ability to run the pipeline successfully and deploy the application.

## Submission Instructions
Please submit your solution as a zip file containing the three starter files above. Ensure that your directory structure is maintained and provide any additional documentation if necessary.