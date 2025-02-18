# Introduction to CDK for Terraform using Golang

## Resources
- [CDK for Terraform Documentation](https://developer.hashicorp.com/terraform/cdktf)
- [Golang Official Documentation](https://golang.org/doc/)
- [Terraform CLI Documentation](https://developer.hashicorp.com/terraform/docs/cli)
- [Homebrew for MacOS](https://brew.sh/)
- [Chocolatey for Windows](https://chocolatey.org/)
- [Node.js Package Manager (NPM)](https://www.npmjs.com/)

---

## Overview
In this demo, I will introduce CDK for Terraform (CDKTF) using Golang. The **CDKTF-GO** project is part of my Terraform course. Inside the **CDKTF-GO** directory, you will find a `README.md` file and Golang source files.

I will be using **VS Code** as my preferred editor, though you can also use other IDEs. Throughout the course, I primarily use **Vi**, but for real-world development, VS Code is a more practical choice as it supports Terraform extensions.

## Setting Up the Environment
To replicate this lab, you need to install the following dependencies:

- **Golang**
- **Terraform**
- **CDKTF** (CDK for Terraform)

### Installation on MacOS
If you are on MacOS, you can install all required tools using Homebrew:
```sh
brew install go cdktf terraform
```

### Installation on Windows
On Windows, I recommend using **Chocolatey** as a package manager:
```sh
choco install golang terraform cdktf
```
Alternatively, you can install Node.js manually and use `npm` to install CDKTF:
```sh
npm install -g cdktf-cli
```

### Using a Preconfigured Vagrant Box
If you are using my **DevOps Vagrant Box**, it already includes a script to install all necessary tools. If you already have the repository cloned, update it:
```sh
git pull
```
Otherwise, clone and start the Vagrant box:
```sh
git clone https://github.com/example/devops-box.git
cd devops-box
vagrant up
vagrant ssh
./install-cdktf.sh
```
This script installs Golang, NPM, and CDKTF.

## Initializing a New CDKTF Project
Once the environment is set up, navigate to the **CDKTF-GO** project and initialize the dependencies:
```sh
cd cdktf-go
cdktf get
go mod tidy
```
These commands:
- `cdktf get`: Downloads Terraform provider bindings and generates necessary Golang files.
- `go mod tidy`: Ensures all Go modules are correctly installed.

**Note:** `cdktf get` may take a long time to complete, so be patient.

## Writing Infrastructure as Code in Golang
Instead of using Terraform's **HCL**, CDKTF allows us to define infrastructure using Golang.

### Main Function (`main.go`)
The entry point for the CDKTF application is the `main` function. Here’s what happens:
1. **Initialize the app:**
   ```go
   app := cdktf.NewApp(nil)
   ```
2. **Define a new stack:**
   ```go
   stack := cdktf.NewTerraformStack(app, cdktf.String("MyStack"))
   ```
3. **Set up an AWS provider:**
   ```go
   awsProvider := aws.NewAwsProvider(stack, cdktf.String("AWS"), &aws.AwsProviderConfig{
       Region: cdktf.String("eu-west-1"),
   })
   ```

### Defining an AWS EC2 Instance
To create an EC2 instance, we use:
```go
instance := ec2.NewInstance(stack, cdktf.String("MyInstance"), &ec2.InstanceConfig{
   Ami:           cdktf.String(getUbuntuAmi(stack)),
   InstanceType:  cdktf.String("t2.micro"),
   Provider:      awsProvider,
})
```
The `getUbuntuAmi()` function dynamically retrieves an AMI instead of hardcoding one:
```go
func getUbuntuAmi(stack cdktf.TerraformStack) string {
   amiData := ec2.NewDataAwsAmi(stack, cdktf.String("UbuntuAmi"), &ec2.DataAwsAmiConfig{
       MostRecent: cdktf.Bool(true),
       Owners:     cdktf.List(cdktf.String("099720109477")),
   })
   return *amiData.Id()
}
```

### Deploying the Infrastructure
Once the code is set up, run:
```sh
cdktf deploy
```
This command:
- Compiles the Golang code.
- Converts it into a Terraform configuration.
- Executes `terraform apply` to deploy resources to AWS.

### Outputting the Public IP
To retrieve the instance’s public IP:
```go
cdktf.NewTerraformOutput(stack, cdktf.String("PublicIP"), &cdktf.TerraformOutputConfig{
   Value: instance.PublicIp(),
})
```

## Understanding CDKTF Output Files
When `cdktf deploy` runs, it generates:
- `cdktf.out/stack.json` - Equivalent to Terraform configuration files.
- `cdktf.out/stacks/MyStack/plan.json` - The Terraform execution plan.

CDKTF translates Golang code into **JSON-based Terraform configurations**, which Terraform then processes.

## Destroying the Infrastructure
To clean up resources:
```sh
cdktf destroy
```
This works similarly to `terraform destroy`, removing all provisioned infrastructure.

## Key Takeaways
- **CDKTF** allows writing Terraform code in **Golang**.
- You still need **Terraform CLI**, **providers**, and **state management**.
- CDKTF makes it easier to integrate **infrastructure-as-code** with software applications.
- If you’re from an **Ops/DevOps background**, you may prefer **HCL** over CDKTF.
- Developers who are already using **Golang, TypeScript, Python, Java, or C#** can benefit from writing infrastructure in their preferred language.

## Additional Resources
- [Terraform Cloud Development Kit (CDKTF) GitHub](https://github.com/hashicorp/terraform-cdk)
- [AWS Provider for Terraform](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Getting Started with Golang](https://golang.org/doc/tutorial/)


