# AWS CodePipeline Demo - Part 3

## Deployment Using Terraform

### Initializing and Applying Terraform Configuration
- Before deploying the infrastructure, we need to initialize Terraform:
  ```sh
  terraform init
  ```
- Once initialized, apply the Terraform configuration to create all required AWS resources:
  ```sh
  terraform apply
  ```
- Ensure that the **AWS region** is correctly set in `vars.tf` before proceeding.
- Terraform will create:
  - Git repository (AWS CodeCommit)
  - AWS CodeBuild
  - AWS CodePipeline
  - AWS VPC
  - AWS ECS Cluster
  - Load Balancer
  - IAM Policies and Roles
  - S3 Buckets for artifacts
  - CodeDeploy setup

### Avoiding S3 Bucket Name Conflicts
- A **random string** is appended to the S3 bucket name to prevent naming conflicts.
- Terraform dynamically generates a unique S3 bucket name to ensure compatibility across multiple deployments.

## Pushing Code to AWS CodeCommit
- Once the infrastructure is deployed, the next step is to push application code to AWS CodeCommit.
- A sample project (`docker-demo`) is used for deployment, which:
  - Listens on **port 3000**.
  - Has a simple **Node.js application** returning `Hello World`.
  - Contains a **Dockerfile** using `Node 12`, running `npm install` and `npm start`.
- The repository must include essential files:
  - `buildspec.yml`: Defines CodeBuild steps.
  - `appspec.yml`: Specifies deployment instructions for CodeDeploy.
  - `create-new-task-def.sh`: Generates a new ECS task definition.

### Cloning and Configuring CodeCommit
- Clone the repository using Git:
  ```sh
  git clone <repository-url>
  ```
- If using a local Git repository, remove the default Git remote:
  ```sh
  git remote remove origin
  ```
- Add AWS CodeCommit as the new remote repository:
  ```sh
  git remote add origin ssh://git-codecommit.<region>.amazonaws.com/v1/repos/demo
  ```
- If using **SSH authentication**, ensure the public key is uploaded to IAM:
  - Navigate to **IAM → Users → Security Credentials**.
  - Upload the SSH **public key**.
  - Use the key ID as the SSH login for CodeCommit.

### Pushing Code to CodeCommit
- Add and commit necessary files:
  ```sh
  git add appspec.yml buildspec.yml create-new-task-def.sh
  git commit -m "Initial commit for CodePipeline"
  ```
- Push the changes to CodeCommit:
  ```sh
  git push -u origin master
  ```

## Triggering CodePipeline
- Once the repository is updated, AWS CodePipeline will trigger automatically.
- **First execution might fail** if the `master` branch does not exist, but pushing to `master` resolves the issue.
- CodePipeline stages:
  1. **Source**: Fetches code from AWS CodeCommit.
  2. **Build**: Executes CodeBuild to build the Docker image and push it to AWS ECR.
  3. **Deploy**: Deploys the image to AWS ECS using CodeDeploy.

## Build Process in AWS CodeBuild
- The build phase involves:
  - Running `docker build`.
  - Pushing the Docker image to **Amazon ECR**.
  - Generating **task definition JSON** for ECS deployment.
  - Uploading artifacts (`appspec.yml` and `taskdef.json`) to S3.
- Build logs can be monitored in AWS CodeBuild.

## Deployment to AWS ECS
- CodeDeploy initiates a **blue-green deployment**:
  - A **replacement task set** is created in ECS.
  - **Traffic routing switches** from `blue` to `green`.
  - A **waiting period of five minutes** is enforced before terminating the old task.
- ECS details:
  - Initial deployments fail until the first image is available in ECR.
  - Once the build is complete, a new ECS task is created and runs the latest container.
  - Health checks validate the container before switching traffic.

## Verifying Deployment
- Check the active target group in ECS to confirm the switch from `blue` to `green`.
- Navigate to the application URL to confirm deployment:
  ```sh
  curl http://<load-balancer-url>
  ```
- If successful, **"Hello World"** should be displayed.

## Additional Deployment Strategies
- Instead of blue-green deployments, other strategies can be configured:
  - **Canary Deployments**: Gradually shifting traffic (e.g., 10% → 20% → 50%).
  - **Rolling Updates**: Incrementally replacing old tasks with new ones.

## Conclusion
- This setup demonstrates an **end-to-end CI/CD pipeline** using **AWS CodePipeline, CodeBuild, CodeDeploy, and ECS Fargate**.
- Terraform simplifies infrastructure provisioning.
- CodePipeline automates application deployment.
- Blue-green deployments reduce downtime.
- For production, consider additional security measures such as **private subnets and NAT Gateways**.

### Next Steps
- **Enhance security** by restricting public access.
- **Optimize CI/CD process** by improving test automation.
- **Monitor performance** using AWS CloudWatch.

If you encounter issues, feel free to ask questions in the Q&A section.

