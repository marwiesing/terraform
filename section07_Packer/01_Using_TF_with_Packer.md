# Using Terraform with Packer

## Resources
- [Packer Documentation](https://developer.hashicorp.com/packer/docs)
- [Terraform Documentation](https://developer.hashicorp.com/terraform/docs)
- [AWS AMI Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)
- [Consul by HashiCorp](https://www.consul.io/)
- [Docker Documentation](https://docs.docker.com/)

## Introduction
In this lecture, we explore how to use **Packer** in conjunction with **Terraform** to build **Amazon Machine Images (AMIs)** efficiently. 

### What is Packer?
Packer is a **command-line tool** that automates the creation of machine images for multiple platforms, including AWS, Azure, and Google Cloud. 
Instead of provisioning software **after** an instance boots up (e.g., using shell scripts or configuration management tools like Chef), Packer allows you to pre-package instances into **custom AMIs**. 

This significantly **reduces instance boot time** by ensuring that required software and configurations are already present in the AMI.

### Why Use Packer with Terraform?
Previously, we used Terraform to configure instances dynamically with:
- **Shell commands** (`provisioner "remote-exec"`)
- **Configuration management tools** (e.g., Chef, Ansible, Puppet)

However, this approach can introduce **longer boot times** and configuration drift. Packer solves these issues by:
- Creating **preconfigured AMIs** with all necessary software baked in.
- Speeding up autoscaling operations in AWS.
- Ensuring consistency across multiple deployments.

## Common Use Cases
Packer is particularly useful when working with **horizontally scalable applications** or **clusters**. Instead of configuring instances on startup, you can launch a predefined AMI multiple times:
- **Consul Clusters** – Boot instances with pre-installed Consul agents.
- **Application Servers** – Include the necessary runtime and application code in the AMI.
- **CI/CD Pipelines** – Automate the creation of development/testing environments.

## Alternative Approaches
Another approach is to use **default AMIs** and install software dynamically at runtime. While this is more flexible, it is slower compared to pre-baked AMIs. Additionally, **Docker** can be used as an alternative:
- Deploy a **lightweight base AMI** with Docker pre-installed.
- Use Terraform to launch instances and **run Docker containers** instead of custom AMIs.
- Benefit from **container orchestration** (e.g., Kubernetes, ECS, Nomad) for even more flexibility.

For those interested in **Docker-based AMI strategies**, refer to the bonus lecture and the Docker course.

## Understanding the Packer Template
A **Packer template** defines how an AMI is built. It consists of several key sections:

### 1. Variables
Variables are used to **store credentials and configurations**:
```json
{
  "variables": {
    "aws_access_key": "{{env `AWS_ACCESS_KEY_ID`}}",
    "aws_secret_key": "{{env `AWS_SECRET_ACCESS_KEY`}}"
  }
}
```
Using environment variables for credentials (`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`) ensures security and avoids hardcoding sensitive information.

### 2. Builders
Builders define the **cloud provider and instance type** used to create the AMI. In AWS, the most common builder is `amazon-ebs`:
```json
{
  "builders": [
    {
      "type": "amazon-ebs",
      "region": "us-east-1",
      "source_ami": "ami-0c55b159cbfafe1f0",
      "instance_type": "t2.micro",
      "ssh_username": "ubuntu",
      "ami_name": "my-custom-ami-{{timestamp}}"
    }
  ]
}
```
- `region`: Defines where the instance is launched.
- `source_ami`: The base image (e.g., Ubuntu AMI ID).
- `instance_type`: Specifies the machine type.
- `ssh_username`: The default user for SSH connections.
- `ami_name`: The final AMI name (using `{{timestamp}}` for uniqueness).

### 3. Provisioners
Provisioners define **installation scripts** that configure the instance before it is converted into an AMI. The most common provisioner is **shell**:
```json
{
  "provisioners": [
    {
      "type": "shell",
      "scripts": ["scripts/install_software.sh"],
      "execute_command": "{{ .Vars }} sudo -E sh '{{ .Path }}'",
      "pause_before": "10s"
    }
  ]
}
```
- This installs and enables **NGINX**, **Docker**, **Vim**, and **LVM2** on the instance before saving it as an AMI.
- Additional **custom applications** can be installed the same way.

## Demo: Using Terraform with Packer
In this demo, we will:
1. Use **Packer** to create a custom AMI.
2. Extract the AMI ID and store it in a Terraform variable (`amivar.tf`).
3. Use **Terraform** to launch an instance with the new AMI.
4. Verify that installed software (NGINX, Docker) is present.

### Steps:
1. Run the build script:
   ```sh
   ./build-and-launch.sh
   ```
2. The script executes Packer, extracts the AMI ID, and stores it in `amivar.tf`.
3. Terraform applies the changes:
   ```sh
   terraform apply
   ```
4. Verify the instance has the necessary software:
   ```sh
   ssh ubuntu@instance-ip
   docker --version
   nginx -v
   curl localhost
   ```
   The output should display "Welcome to Nginx!" confirming the setup.

### Key Takeaways:
✅ **Packer automates AMI creation**, reducing manual effort.
✅ **Terraform seamlessly integrates with Packer**, allowing infrastructure automation.
✅ **Using AMIs ensures consistency across deployments**.
✅ **This approach aligns with immutable infrastructure principles**, improving maintainability.

The next demo will expand on this by integrating **Jenkins for CI/CD**, automating Packer builds, and applying Terraform configurations dynamically.

