# Terraform-Jenkins Infrastructure Pipeline

Automates AWS EC2 instance provisioning using **Terraform** for infrastructure-as-code and **Jenkins** for CI/CD orchestration.

---

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Jenkins Pipeline](#jenkins-pipeline)
- [Troubleshooting](#troubleshooting)

---

## Overview

Automates AWS EC2 infrastructure deployment through a Git-driven CI/CD pipeline. Terraform manages infrastructure, Jenkins orchestrates the workflow, and approval gates ensure safety.

**Key Benefits:**
- Infrastructure as Code with version control
- Automated deployment and destruction
- Manual approval gates for production safety
- Complete audit trail through Git

---

## Features

- ✅ Automated EC2 instance provisioning
- ✅ Customizable instance types, AMIs, and regions
- ✅ Jenkins CI/CD integration with approval workflows
- ✅ Plan review before applying changes
- ✅ Toggle between apply and destroy actions
- ✅ Auto-approval option for automation
- ✅ Automatic resource tagging

---

## Prerequisites

- **Terraform** >= 1.0
- **Jenkins** >= 2.300 (with Pipeline, Git, AWS Credentials plugins)
- **Git** for version control
- **AWS Account** with credentials and EC2 permissions
- **Jenkins Credentials**: Create `aws-access-key-id` and `aws-secret-access-key` in Jenkins

---

## Installation

### 1. Clone the Repository
```bash
git clone https://github.com/shekahassan/Terraform-Jenkins-Infra-Pipeline.git
cd terraform-jenkins-infrastucture-pipeline
```

### 2. Install Terraform
```bash
# Linux
wget https://releases.hashicorp.com/terraform/1.5.0/terraform_1.5.0_linux_amd64.zip
unzip terraform_1.5.0_linux_amd64.zip && sudo mv terraform /usr/local/bin/

# macOS
brew install terraform
```

### 3. Configure AWS Credentials
```bash
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="us-east-1"
```

### 4. Setup Jenkins
1. Create AWS credentials in Jenkins (Manage Jenkins → Manage Credentials)
2. Create a new Pipeline job pointing to this repository
3. Specify `Jenkinsfile` as the pipeline definition

---

## Configuration

### Project Files

| File | Purpose |
|------|---------|
| **Jenkinsfile** | 9-stage pipeline: init, validate, format, plan, approve, apply/destroy, outputs |
| **main.tf** | AWS EC2 instance resource |
| **provider.tf** | AWS provider configuration |
| **variables.tf** | Configurable input variables |
| **output.tf** | Instance public IP and ID outputs |

### Customize Variables

Create `terraform.tfvars`:
```hcl
aws_region    = "us-east-1"
instance_type = "t2.small"
name_tag      = "MyServer"
```

Or use command line:
```bash
terraform apply -var="instance_type=t2.small" -var="name_tag=MyServer"
```

---

## Usage

### Local Testing
```bash
terraform init          # Initialize Terraform
terraform validate      # Check configuration
terraform plan          # Preview changes
terraform apply         # Create infrastructure
terraform destroy       # Remove infrastructure
```

### Jenkins Pipeline

#### Build Parameters
- **autoApprove**: Skip manual approval if checked
- **action**: Choose `apply` (create) or `destroy` (remove)

#### Pipeline Stages
1. **Checkout** - Clone repository
2. **Init** - Initialize Terraform
3. **Validate** - Check syntax
4. **Format** - Apply formatting
5. **Plan** - Generate plan
6. **Approval** - Manual review (if autoApprove=false)
7. **Apply/Destroy** - Execute action
8. **Output Results** - Display IPs and instance IDs

---

## Jenkins Pipeline

### Complete Workflow
```
Checkout → Init → Validate → Format → Plan → Approval* → Apply/Destroy → Outputs
```
*Only if autoApprove=false

### Parameter Combinations

| autoApprove | action  | Behavior |
|-------------|---------|----------|
| false       | apply   | Requires manual approval before creating infrastructure |
| true        | apply   | Automatically creates infrastructure without approval |
| false       | destroy | Shows plan, then auto-destroys (safe) |
| true        | destroy | Automatically destroys without approval |

### Trigger Pipeline
1. Click "Build with Parameters" on Jenkins job
2. Set **autoApprove** (checked/unchecked)
3. Select **action** (apply/destroy)
4. Click "Build"

### Monitor Execution
- Open Jenkins console output
- Each stage shows ✓ status indicators
- Review plan details in "Terraform Plan" stage
- Approve when prompted in "Manual Review" stage

### Outputs
After successful apply:
```bash
public_ip = "1.2.3.4"
instance_id = "i-1234567890abcdef0"
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "No valid credential sources found" | Verify AWS credentials in environment or ~/.aws/credentials |
| "Invalid AMI ID" | Update AMI for your region in variables.tf |
| "Terraform init fails" | Ensure Terraform is installed and internet is available |
| "Jenkins credential error" | Verify credential IDs match in Jenkinsfile |
| "Instance fails to launch" | Check AWS account limits, IAM permissions, and security groups |

**Debug Mode:**
```bash
export TF_LOG=DEBUG
terraform plan
```

---

## Best Practices

1. Always review plans before applying in production
2. Use terraform.tfstate backend security (S3 with encryption)
3. Require manual approval for production changes
4. Keep infrastructure code in Git with meaningful commits
5. Use consistent resource tagging
6. Backup state files regularly
7. Implement least-privilege IAM policies
8. Monitor all infrastructure changes through CI/CD

---

## Support & Resources

- [Terraform Documentation](https://www.terraform.io/docs)
- [AWS Terraform Provider](https://registry.terraform.io/providers/hashicorp/aws/latest)
- [Jenkins Documentation](https://www.jenkins.io/doc)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2)

---

## Contributing

1. Fork the repository
2. Create feature branch: `git checkout -b feature/your-feature`
3. Commit changes: `git commit -m "Add your feature"`
4. Push: `git push origin feature/your-feature`
5. Submit a Pull Request

---

**Author:** Sheka Hassan  
**Last Updated:** 2026-07-24
