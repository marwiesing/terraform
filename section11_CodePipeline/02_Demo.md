# AWS CodePipeline Demo with Terraform

## Overview
This Terraform-based AWS CodePipeline demo automates the entire CI/CD workflow for deploying a containerized application on AWS ECS using Fargate. It leverages AWS CodeCommit (or GitHub via CodeStar), CodeBuild, CodeDeploy, and ECR to create a seamless deployment pipeline. The deployment strategy follows a **Blue-Green** approach, ensuring zero downtime.

```sh
terraform-course/codepipeline-demo/
├── codepipeline.tf         # Defines the AWS CodePipeline setup
├── codecommit.tf           # Sets up the CodeCommit repository (optional for GitHub users)
├── codebuild.tf            # Configures AWS CodeBuild for containerized builds
├── codedeploy.tf           # Specifies CodeDeploy configurations for ECS deployment
├── ecs.tf                  # Defines ECS cluster, services, and task definitions
├── s3.tf                   # Manages S3 storage for pipeline artifacts
├── kms.tf                  # Sets up encryption for artifact storage
├── iam-codebuild.tf        # IAM roles and permissions for CodeBuild
├── iam-codedeploy.tf       # IAM roles and permissions for CodeDeploy
├── iam-codepipeline.tf     # IAM roles and permissions for CodePipeline
├── vpc.tf                  # Defines the network infrastructure
├── lb.tf                   # Load Balancer configuration
├── ecr.tf                  # Amazon Elastic Container Registry configuration
└── provider.tf             # Defines AWS provider and region settings
```


## Key Features
- **Automated Source Integration:** Retrieves application source code from **AWS CodeCommit** or **GitHub**.
- **Build & Packaging:** Uses **AWS CodeBuild** to create a Docker image and push it to **ECR**.
- **Artifact Storage:** Stores deployment artifacts (e.g., task definitions) in an **S3 bucket**.
- **Deployment & Traffic Routing:** Deploys new versions on **ECS Fargate** and handles traffic routing via an **Application Load Balancer (ALB)**.
- **Security & IAM Roles:** Implements role-based access control with **AWS IAM**.

## Pipeline Workflow
### 1. Source Stage
- Retrieves source code from **GitHub** (via CodeStar) or **AWS CodeCommit**.
- Uses Terraform (`codecommit.tf` or `codestar.tf`) to define the repository.

### 2. Build Stage
- Executes AWS CodeBuild using **buildspec.yml**.
- Runs `docker build` and pushes the image to **ECR**.
- Generates ECS task definition (`taskdef.json`).

### 3. Deploy Stage
- Uses **AWS CodeDeploy** to deploy the new ECS task.
- Implements **Blue-Green Deployment** for traffic switching.
- Configures ALB target groups to manage active and inactive environments.

## Infrastructure Breakdown
### Terraform Modules & Resources
- **`codepipeline.tf`** – Defines the pipeline with source, build, and deploy stages.
- **`codebuild.tf`** – Configures the build environment and ECR integration.
- **`codedeploy.tf`** – Manages deployment settings and ECS service updates.
- **`ecs.tf` & `fargate-service.tf`** – Sets up the ECS cluster, services, and task definitions.
- **`lb.tf`** – Configures ALB for traffic management.
- **`s3.tf`** – Creates S3 buckets for storing build artifacts.
- **`iam-*.tf`** – Defines IAM roles and permissions.

## Deployment Steps
1. **Apply Terraform Configuration:**
   ```bash
   terraform init
   terraform apply -auto-approve
   ```
2. **Push Code to Source Repository:**
   ```bash
   git push origin main
   ```
3. **Monitor CodePipeline Execution:**
   - Check AWS Console → CodePipeline for build and deployment status.
4. **Verify Deployment on ECS:**
   ```bash
   aws ecs list-tasks --cluster demo
   ```
5. **Validate Traffic Switch:**
   - Access the **Application Load Balancer (ALB)** URL.
   - Confirm the Blue-Green deployment transition.

## Summary
This Terraform-based AWS CodePipeline automates CI/CD workflows, enabling a seamless, scalable, and secure deployment pipeline for ECS Fargate applications. By following a **Blue-Green Deployment** model, it ensures zero downtime and rollback capabilities, enhancing the reliability of cloud-native applications.

