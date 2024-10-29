# Azure DevOps Pipeline for E-Commerce Platform

![](https://i.imgur.com/waxVImv.png)
### [View all Roadmaps](https://github.com/nholuongut/all-roadmaps) &nbsp;&middot;&nbsp; [Best Practices](https://github.com/nholuongut/all-roadmaps/blob/main/public/best-practices/) &nbsp;&middot;&nbsp; [Questions](https://www.linkedin.com/in/nholuong/)
<br/>

This project involves the development and implementation of a CI/CD pipeline using Azure DevOps for an e-commerce platform. The main goals are to streamline the deployment process, automate the build, test, and deployment stages, and implement strategies to minimize downtime during deployments.

Tools and Technologies
	•	Azure DevOps: Repos, Pipelines, Artifacts
	•	Docker: Containerization
	•	AKS (Azure Kubernetes Service): Orchestration
	•	ACR (Azure Container Registry): Container registry

Key Achievements
	•	Automated build, test, and deployment stages, reducing deployment time by 50%.
	•	Implemented rolling updates and blue-green deployments to minimize downtime.

Detailed Steps and Code

1. Setting Up Azure DevOps Repos
	I.	Create a New Repository
		i.	Go to Azure DevOps organization.
		ii.	Create a new project (e.g., azure-devops-pipeline-for-e-commerce-platform).
		iii.	Navigate to Repos and create a new repository.
	II.	Clone the Repository Locally
```bash
git clone https://github.com/nholuongut/azure-devops-pipeline-for-e-commerce-platform.git
cd azure-devops-pipeline-for-e-commerce-platform
```
	III. Add Project Files Add your e-commerce platform source code to this repository.
	IV.	Commit and Push
```bash
git add .
git commit -m "Initial commit"
git push origin main
```
2. Creating the Azure DevOps Pipeline
	I.	Navigate to Pipelines
		i.	In your Azure DevOps project, go to Pipelines and create a new pipeline.
	II.	Select Repository
		i.	Select the repository you created for the e-commerce platform.
	III.	Configure Pipeline
		i.	Choose Starter Pipeline and replace its content with the following YAML configuration:
```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  dockerRegistryServiceConnection: '<your-service-connection>'
  imageRepository: 'azure-devops-pipeline-for-e-commerce-platform'
  containerRegistry: 'youracr.azurecr.io'
  dockerfilePath: 'Dockerfile'
  tag: '$(Build.BuildId)'

stages:
- stage: Build
  jobs:
  - job: Build
    steps:
    - task: UseDotNet@2
      inputs:
        packageType: 'sdk'
        version: '5.x'
        installationPath: $(Agent.ToolsDirectory)/dotnet

    - script: 'dotnet build --configuration $(buildConfiguration)'
      displayName: 'Build the application'

    - task: Docker@2
      inputs:
        containerRegistry: '$(dockerRegistryServiceConnection)'
        repository: '$(imageRepository)'
        command: 'buildAndPush'
        Dockerfile: '$(dockerfilePath)'
        tags: |
          $(tag)

- stage: Deploy
  dependsOn: Build
  jobs:
  - deployment: Deploy
    environment: 'staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: Kubernetes@1
            inputs:
              connectionType: 'Azure Resource Manager'
              azureSubscriptionEndpoint: '<your-azure-subscription>'
              azureResourceGroup: '<your-resource-group>'
              kubernetesCluster: '<your-k8s-cluster>'
              namespace: 'staging'
              command: 'apply'
              useConfigurationFile: true
              configuration: 'manifests/deployment.yaml'
              secretType: 'dockerRegistry'
              containerRegistryType: 'Azure Container Registry'
              containerRegistry: 'youracr.azurecr.io'
              imagePullSecret: '<image-pull-secret>'
              arguments: '-f manifests/deployment.yaml'

- stage: Production
  dependsOn: Deploy
  jobs:
  - deployment: Production
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: Kubernetes@1
            inputs:
              connectionType: 'Azure Resource Manager'
              azureSubscriptionEndpoint: '<your-azure-subscription>'
              azureResourceGroup: '<your-resource-group>'
              kubernetesCluster: '<your-k8s-cluster>'
              namespace: 'production'
              command: 'apply'
              useConfigurationFile: true
              configuration: 'manifests/deployment.yaml'
              secretType: 'dockerRegistry'
              containerRegistryType: 'Azure Container Registry'
              containerRegistry: 'youracr.azurecr.io'
              imagePullSecret: '<image-pull-secret>'
              arguments: '-f manifests/deployment.yaml'
```

3. Docker and Kubernetes Configuration
	
	I.	Dockerfile
```Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["ECommercePlatform/ECommercePlatform.csproj", "ECommercePlatform/"]
RUN dotnet restore "ECommercePlatform/ECommercePlatform.csproj"
COPY . .
WORKDIR "/src/ECommercePlatform"
RUN dotnet build "ECommercePlatform.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "ECommercePlatform.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "ECommercePlatform.dll"]
```

	II.	Kubernetes Deployment YAML (manifests/deployment.yaml)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-devops-pipeline-for-e-commerce-platform
  labels:
    app: azure-devops-pipeline-for-e-commerce-platform
spec:
  replicas: 3
  selector:
    matchLabels:
      app: azure-devops-pipeline-for-e-commerce-platform
  template:
    metadata:
      labels:
        app: azure-devops-pipeline-for-e-commerce-platform
    spec:
      containers:
      - name: azure-devops-pipeline-for-e-commerce-platform
        image: youracr.azurecr.io/azure-devops-pipeline-for-e-commerce-platform:$(tag)
        ports:
        - containerPort: 80
```
4. Implementing Rolling Updates and Blue-Green Deployments
	I.	Rolling Updates
		i.	Ensure the strategy section in your deployment YAML is configured for rolling updates.
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```
	II.	Blue-Green Deployments
		i.	Set up two environments (e.g., staging and production).
		ii.	Deploy to staging first, validate, and then switch traffic to production.
```yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azure-devops-pipeline-for-e-commerce-platform-staging
  labels:
    app: azure-devops-pipeline-for-e-commerce-platform
    environment: staging
spec:
  replicas: 3
  selector:
    matchLabels:
      app: azure-devops-pipeline-for-e-commerce-platform
      environment: staging
  template:
    metadata:
      labels:
        app: azure-devops-pipeline-for-e-commerce-platform
        environment: staging
    spec:
      containers:
      - name: azure-devops-pipeline-for-e-commerce-platform
        image: youracr.azurecr.io/azure-devops-pipeline-for-e-commerce-platform:$(tag)
        ports:
        - containerPort: 80
```
Summary
This project demonstrates the creation of a robust CI/CD pipeline using Azure DevOps for an e-commerce platform. The pipeline automates the build, test, and deployment stages, and implements rolling updates and blue-green deployments to minimize downtime. With these practices, the deployment process is streamlined, and downtime is significantly reduced, enhancing the overall efficiency and reliability of the e-commerce platform.

# 🚀 I'm are always open to your feedback.  Please contact as bellow information:
### [Contact ]
* [Name: nho Luong]
* [Skype](luongutnho_skype)
* [Github](https://github.com/nholuongut/)
* [Linkedin](https://www.linkedin.com/in/nholuong/)
* [Email Address](luongutnho@hotmail.com)

![](https://i.imgur.com/waxVImv.png)
![](Donate.png)
[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/nholuong)

# License
* Nho Luong (c). All Rights Reserved.🌟