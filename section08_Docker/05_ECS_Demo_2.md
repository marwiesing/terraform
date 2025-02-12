# **Automating Docker Deployment on AWS ECS with Jenkins and Terraform**  
*Building, pushing, and deploying Docker images with a fully automated CI/CD pipeline*  

## **Resources**  
- [Jenkins Official Documentation](https://www.jenkins.io/doc/)  
- [Amazon Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/)  
- [Amazon Elastic Container Service (ECS)](https://aws.amazon.com/ecs/)  
- [Terraform AWS ECS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ecs_cluster)  
- [AWS CLI Documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)  

---

## **1. Overview of the CI/CD Pipeline**  

### **Objective**  
This demo demonstrates **a fully automated pipeline** for:  
✅ **Building a Docker image** in Jenkins.  
✅ **Pushing the image to Amazon ECR** for storage.  
✅ **Deploying the image on ECS** using Terraform.  
✅ **Triggering deployments automatically** based on Git commits.  

### **Workflow**  

1. **Jenkins polls the Git repository** for new changes.  
2. **Jenkins builds a Docker image**, tagging it with the latest **Git commit hash**.  
3. **Jenkins pushes the image** to **Amazon ECR**.  
4. **Jenkins triggers Terraform** to deploy the image on **ECS**.  
5. **Terraform updates ECS** with the latest container version.  
6. **ECS pulls the new image from ECR** and restarts the service.  

---

## **2. Setting Up the CI/CD Pipeline**  

### **Step 1: Infrastructure Setup with Terraform**  

All Terraform configuration files are located in **Docker Demo 3** and define:  
- **ECR Repository (`ecr.tf`)** – Stores Docker images.  
- **ECS Cluster (`ecs.tf`)** – Manages containerized applications.  
- **IAM Policies (`iam.tf`)** – Grants ECS instances access to ECR and logging services.  
- **Security Groups (`securitygroup.tf`)** – Controls access to ECS and Jenkins.  
- **Jenkins Setup (`jenkins.tf`)** – Deploys Jenkins with necessary plugins.  
- **S3 Remote State (`s3.tf`)** – Stores Terraform state files.  

### **Step 2: Provisioning Jenkins with Terraform**  

1. **Initialize Terraform**  
   ```sh
   terraform init
   ```
2. **Apply Terraform Configuration**  
   ```sh
   terraform apply -auto-approve
   ```
   - This sets up **Jenkins, ECR, ECS, IAM roles, security groups, and networking**.  
   - The **Jenkins instance IP** is displayed in the Terraform output.  

3. **Access Jenkins**  
   ```sh
   ssh -i mykey.pem ec2-user@<JENKINS_INSTANCE_IP>
   ```
   - Retrieve the initial **Jenkins admin password**:  
     ```sh
     sudo cat /var/lib/jenkins/secrets/initialAdminPassword
     ```
   - Open **Jenkins Web UI**: `http://<JENKINS_INSTANCE_IP>:8080`  

---

## **3. Configuring Jenkins for Docker Builds**  

### **Step 1: Installing Plugins**  
In **Jenkins**, install the following plugins:  
1. **Git Plugin** – To pull source code from GitHub.  
2. **Parameterized Trigger Plugin** – To pass parameters between jobs.  

### **Step 2: Creating the Build Job**  

#### **1️⃣ Build Job: "Docker Build & Push"**
1. **Create a New Freestyle Project** → Name it `"Docker Build & Push"`.  
2. **Configure Source Code Management (SCM)** → Use Git, enter repository URL.  
3. **Add Build Step → Execute Shell Script**  
   ```sh
   # Define ECR Repository
   REPO_URL="<AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/myapp"

   # Build Docker Image with Git Commit Hash
   GIT_COMMIT=$(git rev-parse --short HEAD)
   docker build -t $REPO_URL:$GIT_COMMIT .

   # Authenticate & Push Image to ECR
   aws ecr get-login-password --region <AWS_REGION> | docker login --username AWS --password-stdin $REPO_URL
   docker push $REPO_URL:$GIT_COMMIT
   ```
4. **Save & Run the Job**  

---

## **4. Automating Deployment to ECS with Jenkins**  

### **Step 1: Creating the Deployment Job**  

#### **2️⃣ Deployment Job: "Docker Deploy to ECS"**  
1. **Create a New Freestyle Project** → Name it `"Docker Deploy to ECS"`.  
2. **Mark the Project as Parameterized** → Add a **String Parameter**:  
   - **Name:** `APP_VERSION`  
   - **Default Value:** `latest`  
3. **Add Build Step → Execute Shell Script**  
   ```sh
   # Change Directory to Terraform Configuration
   cd ~/terraform-course/docker-demo-3

   # Initialize Terraform State from S3
   ./scripts/configure-remote-state.sh

   # Deploy to ECS
   terraform apply -auto-approve -var "MYAPP_VERSION=$APP_VERSION" -var "MYAPP_SERVICE_ENABLE=1"
   ```
4. **Save & Run the Job**  

---

## **5. Automating the Deployment Process**  

### **Step 1: Configure Build Triggers**  

1. **Go to "Docker Build & Push" Job** → **Post-build Actions**  
2. **Add "Trigger parameterized build on other projects"**  
3. **Set Target Project to `"Docker Deploy to ECS"`**  
4. **Add a Predefined Parameter:**  
   - **Name:** `APP_VERSION`  
   - **Value:** `${GIT_COMMIT}`  

### **Step 2: Test the Pipeline**  

1. **Push a change to GitHub**  
2. **Jenkins automatically triggers** the **Docker Build & Push** job  
3. **Jenkins then triggers** the **Docker Deploy to ECS** job  
4. **Terraform updates ECS** with the new Docker image  

---

## **6. Terraform Code Breakdown**  

### **ECS Task Definition (`myapp.tf`)**
```hcl
resource "aws_ecs_task_definition" "myapp-task-definition" {
  family                = "myapp"
  container_definitions = templatefile("templates/app.json.tpl", {
    REPOSITORY_URL = replace(aws_ecr_repository.myapp.repository_url, "https://", "")
    APP_VERSION    = var.MYAPP_VERSION
  })
}
```
- Defines the **container image, memory, and CPU limits**.  
- Uses the **Jenkins-passed Git commit hash** as the image version.  

### **ECS Service Definition (`myapp.tf`)**
```hcl
resource "aws_ecs_service" "myapp-service" {
  count           = var.MYAPP_SERVICE_ENABLE
  name            = "myapp"
  cluster         = aws_ecs_cluster.example-cluster.id
  task_definition = aws_ecs_task_definition.myapp-task-definition.arn
  desired_count   = 1

  load_balancer {
    elb_name       = aws_elb.myapp-elb.name
    container_name = "myapp"
    container_port = 3000
  }
}
```
- ECS service **launches the new container version** when triggered.  

---

## **7. Accessing the Application**  

1. **Retrieve Load Balancer DNS**  
   ```sh
   terraform output elb
   ```
2. **Test the Deployed Application**  
   ```sh
   curl http://<ELB_DNS_NAME>
   ```
   - Expected Output: **`Hello, World!`**  

---

## **8. Summary and Next Steps**  

### **Key Takeaways**  
✅ **Jenkins automates Docker image building and versioning**.  
✅ **AWS ECR stores versioned images** for deployment.  
✅ **Terraform automates ECS deployments** using Jenkins.  
✅ **CI/CD ensures fast and repeatable deployments**.  

