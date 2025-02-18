# AWS CodePipeline for Continuous Delivery

## References
- [AWS CodePipeline Documentation](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html)
- [AWS CodeCommit Documentation](https://docs.aws.amazon.com/codecommit/latest/userguide/welcome.html)
- [AWS CodeBuild Documentation](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html)
- [AWS CodeDeploy Documentation](https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html)
- [AWS Elastic Container Service (ECS)](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [AWS CodeStar Connections](https://docs.aws.amazon.com/dtconsole/latest/userguide/connections-create-github.html)
- [GitHub Repository for Demo](https://github.com/your-repo/docker-demo-codepipeline)

## Overview![overview](image.png)
AWS CodePipeline is a fully managed continuous delivery (CD) service that automates the process of building, testing, and deploying applications. It integrates seamlessly with AWS CodeCommit, CodeBuild, and CodeDeploy, as well as external services like GitHub and Bitbucket.

## Key Features
- **Build Support**: Supports various build tools such as npm (Node.js), Maven, Gradle (Java), and Docker.
- **Deployment Options**: Deploy applications on **EC2**, **on-premises**, **Lambda**, or **ECS**.
- **CI/CD Automation**: Automates the entire pipeline from source code commit to deployment.
- **Integration with Jenkins**: Allows CodePipeline to integrate with an existing Jenkins CI/CD setup.

## Detailed Explanation of Components

### 1. **Source Stage** (CodeCommit / GitHub / Bitbucket)
AWS CodePipeline can be triggered by a source control repository. The most common choices include:
- **AWS CodeCommit** (Amazon's Git-based repository service)
- **GitHub** or **GitHub Enterprise**
- **Bitbucket**
- **Custom S3-based solutions**

### 2. **Build Stage** (AWS CodeBuild)
- CodeBuild launches an **EC2 instance** or a **containerized build environment**.
- It runs unit tests, static code analysis, and other validation checks.
- It can also perform **Docker builds** and push images to Amazon Elastic Container Registry (ECR) or DockerHub.
- Build artifacts, such as deployment packages or container images, are stored in **Amazon S3**.
- Supports alternative build tools like Jenkins or self-hosted solutions.

### 3. **Deploy Stage** (AWS CodeDeploy / ECS / Lambda)
AWS CodeDeploy automates deployments, ensuring zero downtime via strategies like **blue-green deployment** and **rolling updates**.
- **ECS Integration**: CodeDeploy registers new **ECS task definitions** and ensures a smooth deployment.
- **Lambda Integration**: CodeDeploy can also manage function deployments with traffic shifting.
- **EC2 Deployment**: Applications can be deployed directly to EC2 instances or Auto Scaling Groups.

## Deployment Architecture
The architecture involves three major components:
1. **Source Repository**: Stores application source code, including the Dockerfile and build specification.
2. **CodeBuild Execution**: Reads the `buildspec.yml` file and executes `docker build` and `docker push` to store the image in ECR.
3. **CodeDeploy Execution**: Reads `appspec.yaml` to deploy the application in **ECS**.

### **Deployment Workflow**
1. **Developer pushes code to CodeCommit / GitHub** → Triggers AWS CodePipeline.
2. **CodePipeline invokes CodeBuild** → Runs build and tests, and pushes artifacts to S3 and ECR.
3. **CodeDeploy updates ECS Task Definition** → Registers a new version of the application.
4. **Blue-Green Deployment**
   - Initially, all traffic is routed to the **blue** target group.
   - A new **green** target group is created and validated.
   - If successful, traffic is switched from **blue** to **green**.
   - If issues occur, rollback is triggered.
5. **Traffic Routing via Load Balancer** → Ensures seamless switching between old and new deployments.

## Deployment Strategies
- **Blue-Green Deployment** (default strategy): Minimizes downtime by deploying a new version alongside the old version before switching traffic.
- **Canary Deployment**: Gradually shifts traffic to the new version (e.g., 10% initially, then 100%).
- **Rolling Update**: Updates a subset of instances gradually instead of all at once.

## Handling External Git Repositories
AWS CodePipeline allows integration with external repositories through **AWS CodeStar Connections**:
- **GitHub Integration**: Using CodeStar, you can switch between CodeCommit and GitHub as a source.
- **Configuration Steps**:
  1. Go to AWS CodePipeline settings.
  2. Select **Connections** → Create a new connection for GitHub.
  3. Approve GitHub OAuth request.
  4. Ensure status is **Available**.
  5. Use this connection in the pipeline configuration.

## Summary
AWS CodePipeline provides a flexible, automated, and scalable way to manage CI/CD pipelines. By leveraging **CodeBuild, CodeDeploy, and CodeCommit**, developers can efficiently deploy applications using best practices such as **blue-green deployments** and **canary releases**. Additionally, integrations with **GitHub, Jenkins, and Bitbucket** provide enhanced flexibility for various workflows.

---

## Next Steps
In the next lecture, we will perform a hands-on **demo** where we:
- Fork the provided GitHub repository.
- Set up a full CI/CD pipeline in AWS.
- Deploy a containerized Node.js application using CodePipeline.
- Monitor deployment status via AWS Console.

Stay tuned!

