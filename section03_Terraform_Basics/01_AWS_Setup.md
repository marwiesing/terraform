### Spinning Up an AWS EC2 Instance Using Terraform

#### Step 1: Opening an AWS Account
To get started with Terraform on AWS, you first need an AWS account. You can create one by visiting [aws.amazon.com](https://aws.amazon.com). Amazon offers a **free tier** that allows you to use certain resources free of charge for 12 months. For instance:
- You can run a **T2 Micro** instance (a small EC2 instance type) for **750 hours per month** for free within the first year.

After creating your account, always remember to **shut down your instances** after use to avoid incurring unexpected charges.

#### Step 2: Setting Up an IAM User
Once your AWS account is ready, the next step is to set up an IAM (Identity and Access Management) user. Terraform requires programmatic access to AWS resources, so you need to create an admin user with appropriate permissions:

1. **Navigate to IAM**: In the AWS Management Console, search for "IAM" and select **Manage access to AWS resources**.
2. **Create a New User**:
   - Click **Users** > **Add User**.
   - Enter a username (e.g., `TerraformUser`).
   - Select **Programmatic Access** as the access type. This generates an **Access Key ID** and a **Secret Access Key** for API access.
3. **Assign the User to a Group**:
   - If you don’t already have one, create a group (e.g., `Terraform Administrators`) and attach the **AdministratorAccess** policy to it. This grants full access to AWS resources.
4. **Save Access Keys**: After creating the user, securely store the Access Key ID and Secret Access Key. You’ll configure these in Terraform later.

#### Step 3: Understanding the EC2 Dashboard
The **Amazon EC2 Dashboard** is where you manage your instances. To access it, navigate to **Compute > EC2** in the AWS Management Console. Key points to remember:
- Ensure you are in the **correct region** (e.g., `EU-West-1` for Ireland).
- If launching in a different region, update the **region** setting in your Terraform configuration file.
- Each AWS region is isolated, so resources like instances and security groups are region-specific.

#### Step 4: Default Security Groups
When launching instances, AWS assigns them to a **default security group**. Security groups act as virtual firewalls to control inbound and outbound traffic. For initial setups:
- Filter for the **default security group** in your EC2 dashboard.
- Check **inbound rules** to ensure your IP address is allowed:
  1. Go to **Inbound Rules** > **Edit Inbound Rules**.
  2. Add a rule for **My IP** to allow traffic from your machine.
  3. If you have a dynamic IP, you may need to update this rule frequently.

For initial labs and demos, the **default security group** will suffice. However, as you progress to configuring **VPCs (Virtual Private Clouds)**, you’ll define custom security groups in Terraform.

#### Step 5: Writing Your First Terraform Configuration File
To launch a T2 Micro instance, you’ll create a Terraform configuration file (e.g., `main.tf`). A basic configuration looks like this:

```hcl
provider "aws" {
  region = "eu-west-1" # Replace with your desired region
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0" # Replace with a valid AMI ID for your region
  instance_type = "t2.micro"

  tags = {
    Name = "TerraformExampleInstance"
  }
}
```

- **Provider Block**: Specifies the AWS region to deploy resources.
- **Resource Block**: Defines an EC2 instance with an AMI ID and instance type. Replace the AMI ID with one available in your selected region.

#### Step 6: Applying the Configuration
Once your configuration file is ready, follow these steps to deploy the instance:
1. **Initialize Terraform**: Run `terraform init` in the directory containing your configuration file to download necessary provider plugins.
2. **Review Changes**: Execute `terraform plan` to preview the resources Terraform will create.
3. **Apply Configuration**: Use `terraform apply` to create the instance. You’ll be prompted to confirm the changes.
   - Terraform will provision the T2 Micro instance as per the configuration.

#### Additional Notes
- **Shutting Down Instances**: Always terminate unused instances to avoid charges. You can do this via the AWS console or using `terraform destroy`.
- **AWS Free Tier**: Regularly monitor your usage at [aws.amazon.com/free](https://aws.amazon.com/free) to ensure you stay within the free tier limits.

#### What’s Next?
In the next lecture, we’ll delve into managing state files and launching additional AWS resources using Terraform. This foundational knowledge will help you scale and automate your infrastructure effectively.

