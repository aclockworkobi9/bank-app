# Bank Application CI/CD Pipeline

This repository contains a banking application with a complete CI/CD pipeline implementation using Jenkins. The pipeline automates building, testing, scanning, and deploying the application to an Amazon EKS Kubernetes cluster created with Terraform.

## Pipeline Overview

The CI/CD pipeline includes the following stages:

1. **Git Checkout**: Retrieves source code from the repository
2. **Compile**: Compiles the Java application using Maven
3. **Test**: Runs unit tests (currently skipped with `-DskipTests=true`)
4. **Trivy FS Scan**: Performs security scanning on the filesystem
5. **SonarQube Analysis**: Analyzes code quality
6. **Build**: Creates application package with Maven
7. **Publish to Nexus**: Publishes artifacts to Nexus repository
8. **Docker Build Image**: Creates Docker image
9. **Trivy Image Scan**: Scans Docker image for vulnerabilities
10. **Docker Push**: Pushes image to Docker Hub
11. **Kubernetes Interaction**: Deploys application to EKS cluster
12. **Verify Deployment**: Verifies successful deployment

## Infrastructure as Code with Terraform

Our EKS cluster and related infrastructure are provisioned using Terraform. The project structure includes:

- `main.tf & variable.tf`: Contains the core infrastructure configuration, including VPC and EKS cluster setup

### Terraform Configuration

The Terraform configuration manages:

- Amazon VPC with appropriate networking components
- EKS cluster with worker nodes
- IAM roles and policies required for cluster operation
- Security groups and network policies

## RBAC Configuration for EKS

We've implemented comprehensive RBAC (Role-Based Access Control) for our Jenkins integration with EKS. The RBAC configuration files include:

- `ServiceAccount.yaml`: Defines the Kubernetes service account for Jenkins
- `role.yaml`: Specifies permissions for the Jenkins service account
- `role-binding.yaml`: Binds the role to the service account

These configurations ensure that Jenkins has the necessary permissions to deploy to the EKS cluster while following security best practices and the principle of least privilege.

## Jenkins Integration with EKS

The pipeline uses the Jenkins Kubernetes plugin to interact with the EKS cluster. The authentication is handled through the AWS credentials configured in Jenkins, and the deployment process respects the RBAC permissions set up for the Jenkins service account.

## Deployment Process

1. Jenkins builds and tests the application
2. Security scanning is performed on both the code and Docker image
3. The image is built and pushed to Docker Hub
4. Jenkins authenticates with AWS and updates the Kubernetes configuration
5. The application is deployed to the EKS cluster using the configured RBAC permissions
6. Deployment verification ensures the application is running correctly

## Troubleshooting

### Docker Push Errors

- Verify Docker Hub credentials in Jenkins
- Ensure you have permission to push to the specified repository

### EKS Connection Issues

- Verify that IAM permissions are correctly configured
- Check that the aws-auth ConfigMap includes Jenkins user
- Ensure kubectl is properly configured in Jenkins

### Terraform-Related Issues

- Check Terraform state for drift or inconsistencies
- Verify AWS credentials have sufficient permissions

## Repository Structure

- `src/`: Application source code
- `Jenkinsfile`: Jenkins pipeline definition
- `ds.yml`: Kubernetes deployment manifests
- `Dockerfile`: Container image definition
- Terraform files:
  - `main.tf`: Main infrastructure definition (VPC, EKS)
  - `variables.tf`: Input variables for Terraform
  - `output.tf`: Output values after infrastructure creation
- RBAC configuration:
  - `ServiceAccount.yaml`: Jenkins service account
  - `role.yaml`: Role definition with permissions
  - `role-binding.yaml`: Role binding to service account
  - `sec.yaml`: Security configuration
