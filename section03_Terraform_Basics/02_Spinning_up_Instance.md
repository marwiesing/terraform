### Launching a T2 or T3 Micro Instance with Terraform

#### Overview
In this demo, we will:
1. Create a Terraform file to spin up an EC2 instance (T2 or T3 Micro).
2. Run `terraform apply` to provision the infrastructure on AWS.
3. Discuss how to manage credentials securely and understand Terraform’s workflow.

Both **T2 Micro** and **T3 Micro** instances are available under AWS’s free tier. However, note that T3 Micro instances may enable an **unlimited mode** by default, potentially incurring charges. To avoid this, start with a T2 Micro instance and explore T3 Micro for advanced use cases.

#### Step 1: Cloning the Terraform Course Repository
To get the necessary files, clone the Terraform course repository:

```bash
git clone <repository_url>
```
Alternatively, you can download the ZIP file. Cloning provides the advantage of pulling updates (e.g., bug fixes or version upgrades) directly via `git pull`.

The repository includes:
- Terraform demo files.
- A README mapping demos to specific course chapters.
- Regular updates (e.g., module version bumps).

#### Step 2: Preparing Your Development Environment
Choose an editor to modify Terraform files. Recommended options:
- **VS Code**: A user-friendly IDE suitable for beginners and developers.
- **Vim** or **Vim Improved (VI IMproved)**: Lightweight and efficient for terminal-based editing.

For this demo, we’ll use **VS Code**. Download and install it for your operating system. Open the cloned repository directory in VS Code to explore the demo files (e.g., `instance.tf`).

#### Step 3: Configuring AWS Credentials
Terraform requires AWS credentials for API access. There are two methods to configure credentials securely:

1. **Using the AWS CLI**:
   - Run `aws configure`:
     ```bash
     aws configure
     ```
   - Enter your Access Key ID, Secret Access Key, default region, and output format.
   - Credentials will be stored in `~/.aws/credentials`.

2. **Using Environment Variables**:
   - Export credentials in your terminal:
     ```bash
     export AWS_ACCESS_KEY_ID=<your_access_key>
     export AWS_SECRET_ACCESS_KEY=<your_secret_key>
     ```
   - This approach avoids hardcoding sensitive information in Terraform files.

#### Step 4: Writing the Terraform Configuration File
Below is an example of a Terraform file (`instance.tf`) to launch a T2 Micro instance:

```hcl
provider "aws" {
  region = "us-east-1" # Specify your desired AWS region
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0" # Replace with a valid AMI ID
  instance_type = "t2.micro"

  tags = {
    Name = "TerraformExample"
  }
}
```

- **AMI ID**: Locate the appropriate AMI for your region using [Ubuntu EC2 Locator](https://cloud-images.ubuntu.com/locator/).
  - Example: In `us-east-1`, you might use the AMD64 AMI for Ubuntu (e.g., `ami-0c55b159cbfafe1f0`).
  - Avoid using ARM64 AMIs unless required; they’re typically not part of the free tier.

#### Step 5: Initializing Terraform
Before applying your configuration, initialize Terraform:

```bash
terraform init
```
This downloads necessary provider plugins (e.g., `hashicorp/aws`) and creates a `.terraform` directory and a `.terraform.lock.hcl` file to manage provider versions.

#### Step 6: Running Terraform Plan and Apply
1. **Preview Changes**:
   ```bash
   terraform plan
   ```
   This shows the resources Terraform will create or modify without applying changes.

2. **Apply Changes**:
   ```bash
   terraform apply
   ```
   Terraform will first generate a plan, display it, and prompt for confirmation. Type `yes` to proceed. Example output:
   - **1 to add**: A new instance will be created.
   - **Parameters**:
     - AMI: The ID of the image to use.
     - Instance Type: `t2.micro`.

#### Step 7: Verifying the Instance
Once the instance is created, navigate to the EC2 dashboard in the AWS Management Console. Ensure:
- The instance is running in the correct region (e.g., `us-east-1`).
- The default security group is assigned (allowing basic traffic rules).

#### Step 8: Modifying the Configuration
To update the instance (e.g., changing `t2.micro` to `t3.micro`):
1. Edit the `instance_type` in the `instance.tf` file.
2. Apply the changes:
   ```bash
   terraform apply
   ```
   Terraform attempts **in-place updates** (e.g., stopping and restarting the instance with the new configuration) where possible.

#### Step 9: Cleaning Up Resources
To remove resources when they’re no longer needed, use:

```bash
terraform destroy
```
Terraform will:
- Identify and delete resources (e.g., the EC2 instance).
- Prevent orphaned resources and ensure your account is clean.

#### Best Practices
1. **Avoid Hardcoding Credentials**:
   - Use environment variables or the AWS CLI for secure credential management.
2. **Monitor State Files**:
   - Terraform maintains a state file (`terraform.tfstate`) to track resources. Keep it secure and version-controlled.
3. **Split Configurations**:
   - Use multiple `.tf` files for modularity and better organization.

#### What’s Next?
In the next chapter, we’ll explore adding SSH keys, IAM roles, and advanced parameters to the instance, enabling secure and convenient access.

