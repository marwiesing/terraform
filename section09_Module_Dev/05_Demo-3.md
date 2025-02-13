Here is the **enhanced and structured version** of the final part of the demo, including additional explanations, improved clarity, and references to relevant AWS and Terraform documentation.

---

# **Deploying ECS & ALB with Terraform: Final Execution and Verification**  

## **Resources**  
- [In4IT Terraform Modules Repository](https://github.com/in4it/terraform-modules) – Contains ECS and ALB Terraform modules  
- [AWS Elastic Container Service (ECS) Documentation](https://docs.aws.amazon.com/ecs/latest/developerguide/Welcome.html) – Official AWS ECS documentation  
- [AWS Elastic Load Balancing (ALB) Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) – Application Load Balancer (ALB) official guide  
- [AWS Elastic Container Registry (ECR) Documentation](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html) – Overview of AWS ECR for Docker image hosting  

---

## **1️⃣ Preparing for ALB Deployment: SSL Certificate in ACM**  

Before we can **deploy our ALB with HTTPS**, we need to **request an SSL certificate** via AWS Certificate Manager (ACM).  

### **Steps to Request an SSL Certificate**  
1️⃣ Navigate to **AWS Certificate Manager (ACM)** in the AWS console.  
2️⃣ Click **Request a Certificate** → Select **Public Certificate**.  
3️⃣ Enter the domain name (e.g., `*.ecs.newtech.academy`).  
4️⃣ Choose **DNS validation** for automatic verification.  
5️⃣ Copy the **certificate ARN** after validation is complete.  

📌 **Why Do We Need This?**  
- HTTPS listeners require an **SSL certificate** to secure traffic.  
- ACM certificates are **free**, but you need to own the domain name.  

---

## **2️⃣ Initializing and Applying Terraform Configuration**  

Once the certificate is ready, we navigate to the **Terraform module demo folder** and prepare for deployment.  

### **Update Configuration in `vars.tf`**  
Ensure that:  
✅ The **AWS region** is correctly set.  
✅ The **private key path** is defined for EC2 instances.  

If needed, generate a new SSH key:  

```sh
ssh-keygen -t rsa -b 4096 -f ~/.ssh/my-key
```

### **Initialize Terraform Modules**  
We run `terraform init` to download the required modules and dependencies:  

```sh
terraform init
```

📌 **What This Does:**  
- Fetches modules from **GitHub (`in4it/terraform-modules`)**.  
- Configures Terraform’s backend for state storage.  

### **Apply Terraform Configuration**  
Now, we run `terraform apply` to provision the infrastructure:  

```sh
terraform apply
```

📌 **Expected Output:**  
- **44 resources** are created, including:  
  ✅ **ECS Cluster & ECS Service**  
  ✅ **Application Load Balancer (ALB)**  
  ✅ **ALB Listeners & Security Groups**  
  ✅ **CloudWatch Log Groups**  

---

## **3️⃣ Pushing a Docker Image to AWS ECR**  

Once our infrastructure is deployed, we **push a Docker image to AWS ECR**.  

### **Pulling and Tagging an Image**  
First, we **pull the Nginx Docker image** locally:  

```sh
docker pull nginx
```

Next, we need to **re-tag the image** with the ECR repository URI. To find the repository URI:  
- Check the **Terraform state file** or  
- View **ECS > Repositories** in the AWS Console.  

```sh
aws ecr describe-repositories --query "repositories[].repositoryUri"
```

Re-tag the image using the **repository URI**:  

```sh
docker tag nginx <ECR_REPO_URI>:latest
```

### **Logging into AWS ECR & Pushing the Image**  
To authenticate with AWS ECR, we use the AWS CLI:  

```sh
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <ECR_REPO_URI>
```

Now, **push the re-tagged image** to ECR:  

```sh
docker push <ECR_REPO_URI>:latest
```

📌 **Why Re-Tag?**  
- AWS ECR **requires images to have its repository URI** before pushing.  

---

## **4️⃣ ECS Service Deployment & Verification**  

### **Check ECS Service Status**  
After pushing the Docker image, ECS will **fetch the image and deploy tasks**.  

To verify:  
1️⃣ Navigate to **AWS Console > ECS > Clusters**.  
2️⃣ Click on the **ECS Service** → Check **Task Status**.  
3️⃣ Refresh to see tasks moving from **pending** → **running**.  

### **Health Check Validation**  
ECS tasks become **healthy** after passing an **ALB health check**.  
- The **ALB pings ECS tasks** and verifies a `200 OK` response.  
- If successful, the **task is marked healthy** and traffic is routed.  

---

## **5️⃣ Accessing the Application via ALB**  

### **Retrieve ALB DNS Name**  
The **ALB DNS name** can be found in:  
- The **Terraform output** (`terraform output alb_dns_name`)  
- The **AWS Console > EC2 > Load Balancers**  

Example:  

```sh
terraform output alb_dns_name
```

Expected output:  
```sh
my-alb-123456789.us-east-1.elb.amazonaws.com
```

### **Test Application Access**  
Open a browser and **navigate to the ALB DNS name**:  

```sh
http://my-alb-123456789.us-east-1.elb.amazonaws.com
```

📌 **Expected Result:**  
✅ The **Nginx Welcome Page** should be displayed.  

---

## **6️⃣ Enabling HTTPS**  

### **Testing HTTPS Access**  

Even without a **custom domain**, we can test HTTPS:  

```sh
https://my-alb-123456789.us-east-1.elb.amazonaws.com
```

📌 **Expected Browser Warning:**  
⚠️ **Certificate Name Mismatch**  
- Since we are not using a **custom domain**, the browser warns us.  
- Clicking **Proceed** allows access via HTTPS.  

---

## **7️⃣ Linking a Custom Domain via Route 53 (Optional)**  

For **fully secure HTTPS access**, we must:  
1️⃣ **Create a Route 53 DNS record** pointing to the ALB.  
2️⃣ **Ensure the ACM certificate matches the domain name**.  
3️⃣ **Verify HTTPS works without warnings**.  

Example Route 53 Record:  

```sh
record_type = "A"
name        = "ecs.newtech.academy"
alias_target {
  dns_name = module.my-alb.dns_name
  hosted_zone_id = module.my-alb.zone_id
}
```

📌 **Result:**  
- Accessing **`https://ecs.newtech.academy`** now **works without warnings**.  

---

## **8️⃣ Conclusion**  

In this demo, we:  
✅ **Used Terraform to create an ECS cluster and ALB**  
✅ **Set up security groups and IAM roles**  
✅ **Pushed a Docker image to AWS ECR**  
✅ **Deployed an ECS service and verified health checks**  
✅ **Accessed the application via ALB**  
✅ **Tested HTTPS and discussed DNS configuration**  


