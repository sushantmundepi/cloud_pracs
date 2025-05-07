# **Automating EC2 Deployment with Terraform**

This tutorial walks you through automating the launch and management of an Amazon EC2 instance using Terraform, an Infrastructure as Code (IaC) tool. You’ll explore the entire process—from setting up Terraform and configuring AWS access, to writing infrastructure code and deploying a fully functional EC2 instance with minimal manual intervention.


---

## **Prerequisites**

* **AWS Account** – Required to access AWS services.
* **IAM User** Must have programmatic access enabled and appropriate permissions (e.g., EC2, VPC).
* **Access Key ID** and **Secret Access Key** – Used by Terraform to authenticate with AWS.
* **Terraform** Install it from the official source: (https://developer.hashicorp.com/terraform/downloads)
* **AWS CLI** installed (optional, but recommended) Useful for verifying credentials and managing resources: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

---

## 🔧 **Step 1: Create Terraform Configuration**

### `main.tf`

```hcl
provider "aws" {
  region     = var.aws_region
  access_key = var.aws_access_key
  secret_key = var.aws_secret_key
}

resource "aws_instance" "ec2_example" {
  ami           = var.ami_id
  instance_type = var.instance_type
  key_name      = var.key_name

  tags = {
    Name = "Terraform-EC2"
  }
}
```

---

### `variables.tf`

```hcl
variable "aws_region" {
  default = "us-east-1"
}

variable "aws_access_key" {
  type      = string
  sensitive = true
}

variable "aws_secret_key" {
  type      = string
  sensitive = true
}

variable "ami_id" {
  default = "ami-0c94855ba95c71c99"  # Ubuntu 20.04 in us-east-1
}

variable "instance_type" {
  default = "t2.micro"
}

variable "key_name" {
  description = "Existing AWS key pair name"
}
```

---

### `outputs.tf`

```hcl
output "instance_id" {
  value = aws_instance.ec2_example.id
}

output "public_ip" {
  value = aws_instance.ec2_example.public_ip
}
```

---

### `terraform.tfvars` (Optional: or input manually on plan/apply)

```hcl
aws_access_key = "YOUR_ACCESS_KEY"
aws_secret_key = "YOUR_SECRET_KEY"
key_name       = "your-existing-keypair"
```

---

## **Step 2: Initialize & Apply**

Run these Terraform CLI commands in your project directory:

```bash
# Initialize Terraform providers and state
terraform init

# Review the resources Terraform will create
terraform plan

# Apply the configuration to create EC2
terraform apply
```

Type `yes` to approve the deployment.

---

## **Expected Output**

Terraform will output your EC2 instance ID and its public IP:

```bash
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:
instance_id = "i-0123456789abcdef0"
public_ip   = "3.92.165.123"
```

---

## **Destroy Resources**

To tear down the EC2 instance:

```bash
terraform destroy
```

## Links

* https://registry.terraform.io/providers/hashicorp/aws/latest/docs
* https://docs.aws.amazon.com/ec2/index.html

---

