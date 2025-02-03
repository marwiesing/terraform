# **Terraform Outputs – Retrieving Important Information**
#### **Resources**
- 🔗 [Terraform AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)
- 🔗 [Terraform Outputs Documentation](https://developer.hashicorp.com/terraform/language/values/outputs)
- 🔗 [Terraform CLI Documentation](https://developer.hashicorp.com/terraform/cli)
- 🔗 [AWS Elastic IP Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)

---

## **Introduction to Terraform Outputs**
In the previous lecture, we learned how to define **input variables** to make our Terraform configuration more flexible. Now, we will explore **outputs**, which allow us to retrieve useful information from our Terraform resources after deployment.

### **What Are Terraform Outputs?**
Terraform outputs allow us to extract data from our infrastructure. These values can be used for:
- Displaying important details after deployment (e.g., Public IP of an EC2 instance).
- Passing values between Terraform configurations (e.g., passing a VPC ID to another module).
- Debugging and verifying resource attributes.

---

## **Retrieving Attributes from an AWS Instance**
A common use case for outputs is retrieving an AWS EC2 instance’s **public IP address** after deployment.

### **Step 1: Check the AWS Provider Documentation**
Before using outputs, we should verify the attributes available for our resource. Terraform's AWS provider documentation for `aws_instance` lists:
- **Arguments** (what we can define in our configuration).
- **Attributes** (what Terraform provides after creation).

#### **Finding the Public IP Attribute**
1. Open the [AWS Instance Terraform Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance).
2. Scroll down to the **Attribute Reference** section.
3. Locate the **`public_ip`** attribute.

> **Important:** If you use an **Elastic IP (`aws_eip`)**, the `public_ip` attribute won't work as expected. Instead, you must reference the `aws_eip` resource.

---

### **Step 2: Define an Output for Public IP**
To retrieve the EC2 instance’s public IP, we define an output in `outputs.tf`:

```hcl
output "public_ip" {
  description = "The public IP address of the EC2 instance"
  value       = aws_instance.example.public_ip
}
```
- The `value` uses **resource type (`aws_instance`)**, followed by **resource name (`example`)**, and then the **attribute (`public_ip`)**.
- The `description` helps document the output (optional but recommended).

---

### **Step 3: Apply Terraform and Retrieve the Output**
After defining the output, we run Terraform:

```sh
terraform apply
```

### **Expected Output:**
```hcl
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

public_ip = "54.173.99.120"
```
Terraform retrieves and displays the EC2 instance's public IP.

---

### **Understanding How Outputs Work**
Terraform outputs are useful in various scenarios:

#### **1. Accessing Output Values via CLI**
After applying Terraform, you can retrieve an output value without reapplying:
```sh
terraform output public_ip
```
Example output:
```sh
54.173.99.120
```

#### **2. Storing Outputs in a File**
To store outputs in a JSON file:
```sh
terraform output -json > outputs.json
```
This is useful for automation or integration with other tools.

---

## **Differences Between Variables, Data Sources, and Outputs**
| Feature    | Usage | Syntax Example |
|------------|------------|-------------------|
| **Variables (`var.`)** | Used to pass values into a configuration | `var.instance_type` |
| **Data Sources (`data.`)** | Used to retrieve existing resources | `data.aws_instance.existing.id` |
| **Resources (`aws_instance.`)** | Used to create new resources | `aws_instance.example.id` |
| **Outputs (`output` block)** | Used to extract and display information | `output "public_ip" { value = aws_instance.example.public_ip }` |

---

## **Using Outputs in Terraform Modules**
Outputs are particularly useful when working with **modules**. For example, if we create a reusable EC2 module, we can expose the instance's public IP as an output.

Example `outputs.tf` inside a module:
```hcl
output "instance_public_ip" {
  value = aws_instance.web.public_ip
}
```
Then, in the parent configuration:
```hcl
module "ec2" {
  source = "./modules/ec2"
}

output "ec2_public_ip" {
  value = module.ec2.instance_public_ip
}
```
This allows us to reuse the module while still retrieving the necessary values.

---

## **Cleaning Up Resources**
To avoid unnecessary AWS charges, destroy the resources when they're no longer needed:
```sh
terraform destroy
```
Terraform will delete the EC2 instance and all associated resources.

---

## **Key Takeaways**
✅ Terraform outputs allow retrieving useful resource attributes.  
✅ The `public_ip` attribute is available only if an Elastic IP is not attached.  
✅ Outputs can be accessed via `terraform output <name>`.  
✅ Outputs are useful in Terraform modules for sharing data.  
✅ Always clean up resources with `terraform destroy` when testing.

---

### **Next Steps**
In the next lecture, we will discuss **Terraform state**, which tracks resource configurations and their real-world mappings. Stay tuned! 🚀