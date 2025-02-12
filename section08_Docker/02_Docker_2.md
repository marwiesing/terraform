Here is the enhanced version of the transcript with additional information, context updates, and references to resources.

---

# **Building and Deploying Docker Images on AWS**  
*Containerizing applications and automating deployments with Terraform*  

## **Resources**  
- [Docker Hub - Official Images](https://hub.docker.com/)  
- [Amazon EC2 Container Registry (ECR)](https://aws.amazon.com/ecr/)  
- [Amazon Elastic Container Service (ECS)](https://aws.amazon.com/ecs/)  
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)  
- [AWS CLI Documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)  

---

## **1. Building a Docker Image for a Node.js App**  

### **Introduction to Dockerizing Applications**  
In this section, we will containerize a **Node.js application** by creating a **Docker image**. This involves:  
1. Writing a **Dockerfile** that defines the image.  
2. Using `docker build` to create the image.  
3. Running and testing the container locally.  
4. Pushing the image to **Amazon ECR (Elastic Container Registry)**.  
5. Deploying the image to **ECS (Elastic Container Service)** using **Terraform**.  

---

### **Step 1: Creating the Dockerfile**  
A **Dockerfile** is a script that contains a set of instructions to build a Docker image. Below is an example for a simple Node.js application:  

#### **Dockerfile**
```dockerfile
# Use the official Node.js base image
FROM node:4.6  

# Set the working directory inside the container
WORKDIR /app  

# Copy application files into the container
ADD . /app  

# Install dependencies
RUN npm install  

# Expose port 3000 for incoming connections
EXPOSE 3000  

# Start the Node.js application
CMD ["npm", "start"]
```
#### **Understanding Each Command:**
| Command | Description |
|---------|------------|
| `FROM node:4.6` | Uses the Node.js version 4.6 base image from Docker Hub. |
| `WORKDIR /app` | Sets the working directory inside the container. |
| `ADD . /app` | Copies all application files to the `/app` directory inside the container. |
| `RUN npm install` | Installs all dependencies defined in `package.json`. |
| `EXPOSE 3000` | Opens port 3000 for incoming connections. |
| `CMD ["npm", "start"]` | Defines the default command to run when the container starts. |

---

### **Step 2: Application Files**  

1. **index.js** - A simple Node.js application using Express:  
   ```javascript
   const express = require('express');
   const app = express();

   app.get('/', (req, res) => {
       res.send('Hello, World!');
   });

   app.listen(3000, () => {
       console.log('Server running on port 3000');
   });
   ```

2. **package.json** - Defines dependencies and startup scripts:  
   ```json
   {
     "name": "myapp",
     "version": "1.0.0",
     "private": true,
     "scripts": {
       "start": "node index.js"
     },
     "dependencies": {
       "express": "^4.17.1"
     }
   }
   ```

---

### **Step 3: Building the Docker Image**  
To build the image, navigate to the project directory (`docker-demo`) and execute the following command:  

```sh
docker build -t myapp .
```
- The **dot (`.`)** represents the current directory containing the Dockerfile.  
- This command creates a Docker image named `myapp`.  

**Verifying the Built Image:**  
```sh
docker images
```
- This lists all locally available Docker images, including the newly built `myapp` image.

---

### **Step 4: Running the Docker Container Locally**  
To test the application, run the container:  

```sh
docker run -p 3000:3000 myapp
```
- The `-p 3000:3000` flag maps port **3000** of the container to **3000** on the host.  
- Open `http://localhost:3000` in your browser to verify that it works.  

---

## **2. Pushing the Docker Image to AWS ECR**  

### **Step 1: Creating an Amazon ECR Repository Using Terraform**  
Terraform can automate the creation of an ECR repository:  

#### **ecr.tf**
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_ecr_repository" "myapp" {
  name = "myapp"
}

output "ecr_repository_url" {
  value = aws_ecr_repository.myapp.repository_url
}
```
### **Deploying the Terraform Configuration**  
```sh
terraform init
terraform apply -auto-approve
```
- This **creates an ECR repository** named `myapp` and outputs its **repository URL**.

---

### **Step 2: Authenticating Docker with AWS ECR**  
To push images to ECR, first authenticate Docker with AWS:  

```sh
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com
```

---

### **Step 3: Tagging and Pushing the Image**  
1. Tag the Docker image with the ECR repository URL:  
   ```sh
   docker tag myapp:latest <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
   ```
2. Push the tagged image to ECR:  
   ```sh
   docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
   ```

---

## **3. Running the Image on AWS ECS**  

### **Step 1: Creating an ECS Cluster Using Terraform**  
Once the image is in ECR, we can define an ECS cluster and service to run it.

#### **ecs.tf**
```hcl
resource "aws_ecs_cluster" "my_cluster" {
  name = "my-cluster"
}

resource "aws_ecs_task_definition" "my_task" {
  family                   = "my-task"
  container_definitions    = jsonencode([
    {
      name      = "my-container",
      image     = "${aws_ecr_repository.myapp.repository_url}:latest",
      memory    = 512,
      cpu       = 256,
      essential = true,
    }
  ])
}

resource "aws_ecs_service" "my_service" {
  name            = "my-service"
  cluster         = aws_ecs_cluster.my_cluster.id
  task_definition = aws_ecs_task_definition.my_task.arn
  desired_count   = 2
}
```

### **Step 2: Deploying the ECS Cluster**  
```sh
terraform apply -auto-approve
```
- This **creates an ECS cluster**, **registers the container**, and **launches two instances of the container**.

---

## **4. Summary and Next Steps**  

### **Key Takeaways:**  
✅ **Docker Images** encapsulate the OS, application, and dependencies in a single package.  
✅ **ECR** stores Docker images for AWS deployments.  
✅ **ECS** provides a managed way to run Docker containers on AWS.  
✅ **Terraform** automates infrastructure setup, making deployments repeatable.  

### **Next Steps:**  
- Deploy the **ECS service** and test the running container.  
- Explore **AWS Fargate**, a **serverless** way to run containers without managing EC2 instances.  
- Set up **CI/CD pipelines** with **Jenkins, GitLab CI/CD, or AWS CodeBuild** for automated builds.  

