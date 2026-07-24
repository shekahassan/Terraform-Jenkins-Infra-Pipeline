# Terraform-Jenkins Infrastructure Pipeline

A comprehensive Infrastructure-as-Code (IaC) solution that combines **Terraform** and **Jenkins** to automate AWS EC2 instance provisioning and management. This project enables automated infrastructure deployment, scaling, and destruction through a declarative configuration management approach with Jenkins CI/CD orchestration.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Installation & Setup](#installation--setup)
- [Configuration](#configuration)
- [Usage](#usage)
- [Jenkins Pipeline Workflow](#jenkins-pipeline-workflow)
- [Terraform Commands](#terraform-commands)
- [Outputs](#outputs)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

---

## Overview

This project automates the deployment and management of AWS EC2 instances using **Terraform** for infrastructure provisioning and **Jenkins** for CI/CD orchestration. It provides a repeatable, version-controlled, and auditable way to manage cloud infrastructure.

### Key Benefits:
- **Infrastructure as Code**: Define infrastructure in version-controlled Terraform files
- **Automation**: Automated provisioning and destruction of AWS resources
- **CI/CD Integration**: Seamless integration with Jenkins for workflow orchestration
- **Cost Control**: Easy infrastructure scaling and destruction
- **Auditability**: Full history of infrastructure changes through Git
- **Human Approval**: Manual approval steps before applying changes

---

## Features

✅ **Automated EC2 Provisioning**: Automatically launch Ubuntu EC2 instances on AWS
✅ **Flexible Configuration**: Customizable instance types, AMI IDs, and region settings
✅ **Jenkins Integration**: Full CI/CD pipeline with automated approval workflows
✅ **Plan Review**: Human-readable Terraform plans for review before applying
✅ **Apply/Destroy Actions**: Toggle between infrastructure creation and destruction
✅ **Auto-Approval Option**: Skip approval stage for automated deployments
✅ **AWS Region Support**: Easily switch AWS regions
✅ **Instance Tagging**: Automatic resource tagging for better organization

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Git Repository                          │
│              (Terraform & Jenkins Code)                     │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
        ┌────────────────────────────┐
        │       Jenkins Server       │
        │  - Triggers Pipeline       │
        │  - Approves Changes        │
        │  - Executes Terraform      │
        └────────┬───────────────────┘
                 │
        ┌────────┴───────────────┐
        │   Terraform Workflow   │
        │  - Init                │
        │  - Plan                │
        │  - Apply/Destroy       │
        └────────┬───────────────┘
                 │
                 ▼
        ┌────────────────────────────┐
        │      AWS Provider          │
        │  - AWS API Calls           │
        │  - Credential Management   │
        └────────┬───────────────────┘
                 │
                 ▼
        ┌────────────────────────────┐
        │    AWS Resources           │
        │  - EC2 Instance            │
        │  - Instance Tags           │
        └────────────────────────────┘
```

---

## Prerequisites

Before you begin, ensure you have the following:

### Software Requirements:
- **Terraform** >= 1.0 (recommended: latest version)
- **Jenkins** >= 2.300+ with the following plugins:
  - Pipeline
  - Git
  - AWS Credentials
- **Git** for version control
- **AWS CLI** (optional, for manual operations)

### AWS Requirements:
- Active AWS account with appropriate permissions
- AWS Access Key ID and Secret Key
- IAM permissions for EC2, VPC, and related services
- Valid AWS region (default: `us-east-1`)

### Jenkins Configuration:
- Jenkins credentials named `aws-access-key-id` and `aws-secret-access-key`
- Credentials should contain your AWS access keys
- Jenkins must have internet access to GitHub and AWS APIs

---

## Project Structure

```
terraform-jenkins-infrastucture-pipeline/
├── Jenkinsfile                 # Jenkins CI/CD Pipeline Definition
├── main.tf                     # Primary Terraform Configuration
├── provider.tf                 # AWS Provider Configuration
├── variables.tf                # Input Variables & Defaults
├── output.tf                   # Output Values (IPs, Instance IDs)
└── README.md                   # Documentation (this file)
```

### File Descriptions:

| File | Purpose |
|------|---------|
| **Jenkinsfile** | Defines the Jenkins pipeline with stages for checkout, init, plan, and apply/destroy |
| **main.tf** | Creates the AWS EC2 instance resource |
| **provider.tf** | Configures the AWS provider with API credentials and region |
| **variables.tf** | Defines input variables with defaults for flexibility |
| **output.tf** | Exports instance public IP and instance ID |

---

## Installation & Setup

### Step 1: Clone the Repository

```bash
git clone https://github.com/shekahassan/Terraform-Jenkins-Infra-Pipeline.git
cd terraform-jenkins-infrastucture-pipeline
```

### Step 2: Install Terraform

**On Linux:**
```bash
wget https://releases.hashicorp.com/terraform/1.5.0/terraform_1.5.0_linux_amd64.zip
unzip terraform_1.5.0_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform --version
```

**On macOS:**
```bash
brew install terraform
terraform --version
```

### Step 3: Configure AWS Credentials

Export your AWS credentials:
```bash
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"
```

Or create an AWS credentials file at `~/.aws/credentials`:
```
[default]
aws_access_key_id = your-access-key
aws_secret_access_key = your-secret-key
```

### Step 4: Configure Jenkins

1. Install required Jenkins plugins
2. Create credentials in Jenkins:
   - Navigate to **Manage Jenkins** → **Manage Credentials**
   - Add credential type: **Secret text**
   - Credential ID: `aws-access-key-id`
   - Secret: Your AWS Access Key ID
3. Repeat for `aws-secret-access-key`
4. Create a new **Pipeline** job
5. Point it to this repository and specify the `Jenkinsfile`

---

## Configuration

### Variables (variables.tf)

The project uses the following configurable variables:

```hcl
variable "aws_access_key" {
  description = "AWS access key"
  type        = string
  default     = ""
}

variable "aws_secret_key" {
  description = "AWS secret key"
  type        = string
  default     = ""
}

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"  # Change to your preferred region
}

variable "ami" {
  type        = string
  description = "Ubuntu AMI ID"
  default     = "ami-0f5ee92e2d63afc18"  # us-east-1 Ubuntu 22.04 LTS
}

variable "instance_type" {
  type        = string
  description = "Instance type"
  default     = "t2.micro"  # Free tier eligible
}

variable "name_tag" {
  type        = string
  description = "Name of the EC2 instance"
  default     = "My EC2 Instance"
}
```

### Customizing Variables:

**Option 1: Terraform Variables File (terraform.tfvars)**
```hcl
aws_region    = "us-east-1"
instance_type = "t2.small"
name_tag      = "Production-Server"
```

**Option 2: Command Line Override**
```bash
terraform apply -var="instance_type=t3.medium" -var="name_tag=WebServer"
```

**Option 3: Environment Variables**
```bash
export TF_VAR_instance_type="t2.small"
export TF_VAR_name_tag="MyServer"
```

---

## Usage

### Local Development (Manual Terraform)

#### Initialize Terraform:
```bash
terraform init
```

#### Validate Configuration:
```bash
terraform validate
```

#### Preview Changes:
```bash
terraform plan
```

#### Apply Infrastructure:
```bash
terraform apply
```

#### Destroy Infrastructure:
```bash
terraform destroy
```

### Jenkins Pipeline Usage

#### Trigger the Pipeline:

1. Open Jenkins Dashboard
2. Select your pipeline job
3. Click **"Build with Parameters"**
4. Configure parameters:
   - **autoApprove**: Check to skip manual approval
   - **action**: Select `apply` or `destroy`
5. Click **Build**

#### Pipeline Flow:

```
Checkout → Init → Plan → Manual Approval (if needed) → Apply/Destroy
```

---

## Jenkins Pipeline Workflow

The `Jenkinsfile` orchestrates the following workflow:

### Pipeline Stages:

| Stage | Description |
|-------|-------------|
| **Checkout** | Clones the repository |
| **Terraform init** | Initializes Terraform working directory |
| **Plan** | Generates and displays Terraform plan |
| **Apply / Destroy** | Executes apply or destroy action with optional approval |

### Pipeline Parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `autoApprove` | Boolean | false | Skip manual approval if true |
| `action` | Choice | apply | Choose between `apply` or `destroy` |

### Environment Variables:

The pipeline sets up AWS credentials from Jenkins secrets:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_DEFAULT_REGION`: us-east-1

---

## Terraform Commands

### Essential Commands:

```bash
# Initialize Terraform working directory
terraform init

# Validate configuration syntax
terraform validate

# Format code to standard conventions
terraform fmt -recursive

# Show resource plan
terraform plan

# Apply changes
terraform apply

# Destroy all infrastructure
terraform destroy

# Show current state
terraform show

# Output values from state
terraform output

# Refresh state from AWS
terraform refresh

# Target specific resources
terraform apply -target=aws_instance.public_instance
```

### State Management:

```bash
# List resources in state
terraform state list

# Show specific resource details
terraform state show aws_instance.public_instance

# Remove resource from state (without destroying)
terraform state rm aws_instance.public_instance
```

---

## Outputs

After successful deployment, Terraform outputs the following values:

### Generated Outputs:

```
Outputs:

public_ip = "<instance-public-ip>"
instance_id = "i-<instance-id>"
```

### Accessing Outputs:

```bash
# Get all outputs
terraform output

# Get specific output
terraform output public_ip
terraform output instance_id

# Output in JSON format
terraform output -json
```

### Use Cases for Outputs:

- SSH into the instance: `ssh -i key.pem ubuntu@<public_ip>`
- Configure DNS records
- Set up security group rules
- Application configuration

---

## Troubleshooting

### Common Issues & Solutions:

#### Issue: "No valid credential sources found"
```
Solution:
- Verify AWS credentials in environment or ~/.aws/credentials
- Check Jenkins credentials configuration
- Ensure IAM user has necessary permissions
```

#### Issue: "Invalid AMI ID"
```
Solution:
- Update AMI ID for your region using AWS CLI:
  aws ec2 describe-images --owners amazon \
    --filters "Name=root-device-type,Values=ebs" \
    "Name=name,Values=amzn2-ami-hvm-*" \
    --query 'sort_by(Images, &CreationDate)[-1]'
```

#### Issue: "Terraform init fails"
```
Solution:
- Ensure Terraform is installed: terraform version
- Check internet connectivity
- Verify .terraform directory permissions
```

#### Issue: "Jenkins build fails with credential error"
```
Solution:
- Verify credential IDs match in Jenkinsfile
- Check Jenkins has permissions to access credentials
- Test credentials with: aws sts get-caller-identity
```

#### Issue: "Instance fails to launch"
```
Solution:
- Verify AWS account limits (vCPU, instances)
- Check IAM permissions for EC2 operations
- Verify instance type is available in region
- Check security group rules
```

### Debug Mode:

Enable verbose Terraform output:
```bash
export TF_LOG=DEBUG
terraform plan
```

Enable Jenkins debug logging:
```
Manage Jenkins → System Log → Add log recorder
Set levels to DEBUG for terraform/aws packages
```

---

## Best Practices

1. **Always Review Plans**: Never skip the plan phase in production
2. **Use State Files Securely**: Store state files in secure backends (S3 with encryption)
3. **Implement Approval Gates**: Require human approval for production changes
4. **Version Control**: Keep all IaC code in Git with meaningful commits
5. **Tag Resources**: Use consistent tagging for cost tracking and organization
6. **Backup State**: Regularly backup Terraform state files
7. **Limit Permissions**: Use least-privilege IAM policies
8. **Monitor Changes**: Track all infrastructure changes through CI/CD
9. **Document Changes**: Use commit messages to document infrastructure decisions
10. **Test Locally**: Test Terraform configurations locally before committing

---

## Contributing

To contribute to this project:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "Add your feature"`
4. Push to the branch: `git push origin feature/your-feature`
5. Submit a Pull Request

---


## Support & Resources

- [Terraform Documentation](https://www.terraform.io/docs)
- [AWS Terraform Provider](https://registry.terraform.io/providers/hashicorp/aws/latest)
- [Jenkins Documentation](https://www.jenkins.io/doc)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2)

---

## Author

Created by Sheka Hassan

For questions or issues, please open an issue on the GitHub repository.

---

**Last Updated**: 2026-07-24
