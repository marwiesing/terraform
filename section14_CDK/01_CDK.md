# CDK for Terraform (CDKTF)

## Resources
- [CDK for Terraform Official Documentation](https://developer.hashicorp.com/terraform/cdktf)
- [Terraform Registry](https://registry.terraform.io/)
- [AWS CDK vs. CDKTF - A Comparison](https://aws.amazon.com/cdk/)
- [Terraform HCL Syntax Guide](https://developer.hashicorp.com/terraform/language/syntax)

## Introduction
CDK for Terraform (CDKTF) is a cloud development kit that allows infrastructure provisioning using Terraform. Instead of writing Terraform configuration in HashiCorp Configuration Language (HCL) within `.tf` files, CDKTF enables developers to use general-purpose programming languages to define infrastructure.

CDKTF currently supports the following languages:
- **TypeScript**
- **Python**
- **Java**
- **C#**
- **Go**

## How CDKTF Works
CDKTF functions as an abstraction layer over Terraform, allowing developers to write infrastructure code in their preferred programming language. This code is then **synthesized** (translated) into Terraform configuration files before being deployed via Terraform.

### Key Workflow
1. Developers write infrastructure code using their chosen programming language.
2. CDKTF **synthesizes** the code into `.tf.json` files (Terraform JSON configuration).
3. Terraform processes these files and applies the infrastructure changes using `terraform apply`.
4. Developers can modify their infrastructure using **CDKTF commands** like `cdktf deploy` and `cdktf diff`.

This approach maintains Terraform's declarative nature while enabling imperative logic for more complex scenarios.

## Code Example: Defining an EC2 Instance in Go
Below is an example demonstrating how CDKTF can be used to define an AWS EC2 instance in Go (Golang):

```go
package main

import (
    "github.com/hashicorp/terraform-cdk-go/cdktf"
    "github.com/hashicorp/terraform-cdk-go/cdktf/providers/aws"
)

type MyStack struct {
    cdktf.TerraformStack
}

func NewMyStack(scope cdktf.App, id string) *MyStack {
    stack := cdktf.NewTerraformStack(scope, &id)

    provider := aws.NewAwsProvider(stack, cdktf.String("AWS"), &aws.AwsProviderConfig{
        Region: cdktf.String("us-west-2"),
    })

    instance := aws.NewInstance(stack, cdktf.String("EC2Instance"), &aws.InstanceConfig{
        Ami:          cdktf.String("ami-12345678"),
        InstanceType: cdktf.String("t2.micro"),
    })

    return &MyStack{TerraformStack: *stack}
}

func main() {
    app := cdktf.NewApp(nil)
    NewMyStack(*app, "MyFirstCDKTFStack")
    app.Synth()
}
```

## Comparison: HCL vs. CDKTF
| Feature | HCL (Traditional Terraform) | CDKTF |
|---------|----------------------------|-------|
| Language | HCL | TypeScript, Python, Go, Java, C# |
| Abstractions | Limited | Rich abstractions using OOP |
| Reusability | Modules | Classes and functions |
| Testing | Terraform Testing Framework | Unit tests in standard testing frameworks |
| Tooling | Terraform CLI | CDKTF CLI + Terraform |

### Example: AWS AMI Data Source in HCL vs. Go
**HCL Version:**
```hcl
provider "aws" {
  region = "us-west-2"
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-20.04-amd64-server-*"]
  }
}
```

**Go Version with CDKTF:**
```go
dataSource := aws.NewDataAwsAmi(stack, cdktf.String("UbuntuAMI"), &aws.DataAwsAmiConfig{
    MostRecent: cdktf.Bool(true),
    Owners:     []*string{cdktf.String("099720109477")},
    Filter: &aws.DataAwsAmiFilter{
        Name:   cdktf.String("name"),
        Values: []*string{cdktf.String("ubuntu/images/hvm-ssd/ubuntu-*-20.04-amd64-server-*")},
    },
})
```

## Benefits of CDKTF
- **Familiarity**: Developers can write infrastructure using standard programming languages.
- **Rich Abstractions**: Object-oriented programming (OOP) allows modularization and code reuse.
- **Better Tooling**: Integrated with IDEs like VS Code, JetBrains, and Eclipse.
- **Improved Testing**: Can leverage unit tests and integration tests for infrastructure code.
- **Flexibility**: Allows conditional logic, loops, and dynamic configurations not easily possible in HCL.

## CDKTF vs. AWS CDK
CDKTF is conceptually similar to AWS CDK but works across multiple cloud providers. While AWS CDK converts code into AWS CloudFormation templates, CDKTF converts it into Terraform configuration.

| Feature | AWS CDK | CDKTF |
|---------|--------|-------|
| Target Platform | AWS Only | Multi-Cloud (AWS, Azure, GCP, Kubernetes, etc.) |
| Backend | CloudFormation | Terraform |
| Language Support | TypeScript, Python, Java, C# | TypeScript, Python, Java, C#, Go |
| Multi-Provider Support | No | Yes |

## Current Status & Considerations
As of **February 2025**, CDKTF is stable and ready for production use. However, it is recommended to:
- Stay updated with **Terraform CDK releases**.
- Use CDKTF in combination with **best Terraform practices**.
- Evaluate whether your team benefits more from **HCL or CDKTF**, depending on complexity and familiarity.

## Conclusion
CDK for Terraform (CDKTF) bridges the gap between infrastructure and application development by allowing developers to provision infrastructure using familiar programming languages. It retains Terraform's robust state management and multi-cloud support while providing additional flexibility and abstraction capabilities.

CDKTF is an excellent choice for teams that:
- Are already familiar with **object-oriented programming**.
- Need **advanced infrastructure logic** with loops and conditionals.
- Want to **reuse code** across multiple environments efficiently.

For further exploration, visit the [CDKTF documentation](https://developer.hashicorp.com/terraform/cdktf) and try out the [examples](https://developer.hashicorp.com/terraform/cdktf/docs/examples) to get hands-on experience!

