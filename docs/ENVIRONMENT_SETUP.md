# Environment Setup Workflow

This document provides comprehensive guidance for using the Environment Setup GitHub Actions workflow to provision AWS infrastructure using Terraform.

## Overview

The Environment Setup workflow automates the provisioning of environment-level shared infrastructure including:

- **Amazon EKS Cluster** with managed node groups
- **ECR Repositories** for container image storage
- **Terraform Backend** (S3 + DynamoDB) for state management

## Quick Start

### 1. Navigate to Actions Tab
Go to the GitHub Actions tab in the [aws-infrastructure](https://github.com/PrakashNexTurn/aws-infrastructure) repository.

### 2. Select Environment Setup Workflow
Find and click on "Environment Setup" workflow.

### 3. Run Workflow
Click "Run workflow" and configure the parameters:

| Parameter | Description | Example | Required |
|-----------|-------------|---------|----------|
| **environment** | Target environment | `dev`, `staging`, `prod` | ✅ Yes |
| **ecr_services** | Comma-separated service list | `frontend,backend,api` | ✅ Yes |
| **action** | Terraform action | `plan`, `apply`, `destroy` | ✅ Yes |
| **aws_region** | AWS deployment region | `us-west-2` | ❌ No (default: us-west-2) |

### 4. Monitor Execution
Watch the workflow progress through each stage and review the deployment summary.

## Workflow Stages

### Stage 1: Input Validation
- Validates environment name (dev/staging/prod)
- Validates ECR service names (lowercase, alphanumeric, hyphens, underscores)
- Handles AWS region constraints (e.g., EKS us-east-1e limitation)
- Sets up pipeline variables

### Stage 2: Backend Setup
- **S3 Bucket Creation**: `nextops-terraform-state-<environment>`
  - Enables versioning for state history
  - Configures server-side encryption (AES256)
  - Blocks all public access
- **DynamoDB Table Creation**: `nextops-terraform-state-<environment>`
  - Provides state locking mechanism
  - Prevents concurrent modifications

### Stage 3: Infrastructure Provisioning
- **EKS Cluster Deployment**:
  - Kubernetes version 1.33
  - Managed node groups with auto-scaling
  - KMS encryption for secrets
  - Comprehensive control plane logging
  - Proper subnet configuration (filters out us-east-1e if needed)
- **ECR Repository Creation**:
  - Per-service container registries
  - Vulnerability scanning enabled
  - KMS encryption
  - Lifecycle policies for image management

### Stage 4: Summary Generation
- Deployment status and configuration
- Infrastructure component details
- Output values for integration

## Infrastructure Components

### EKS Cluster Configuration

| Environment | Node Size | Min Nodes | Max Nodes | Desired Nodes | Instance Type |
|-------------|-----------|-----------|-----------|---------------|---------------|
| **dev** | Small | 1 | 3 | 2 | t3.medium |
| **staging** | Small | 1 | 3 | 2 | t3.medium |
| **prod** | Large | 2 | 5 | 3 | t3.large |

**Features Enabled:**
- ✅ KMS encryption for secrets
- ✅ Control plane logging (API, audit, authenticator, controller manager, scheduler)
- ✅ Managed node groups with auto-scaling
- ✅ VPC integration with default VPC and subnets
- ✅ Proper availability zone handling (excludes us-east-1e for EKS compatibility)

### ECR Repository Configuration

Each service gets its own ECR repository with:

| Feature | dev/staging | prod |
|---------|-------------|------|
| **Image Scanning** | ✅ Enabled | ✅ Enabled |
| **KMS Encryption** | ✅ Enabled | ✅ Enabled |
| **Force Delete** | ✅ Yes | ❌ No |
| **Max Images** | 10 | 20 |
| **Retention (Untagged)** | 7 days | 14 days |

**Protected Tags:** `latest`, `main`, `master`, `<environment>`, `stable`

## AWS Regions Support

| Region | Status | Notes |
|--------|--------|-------|
| **us-east-1** | ✅ Supported | us-east-1e AZ automatically excluded for EKS |
| **us-west-1** | ✅ Supported | Full AZ support |
| **us-west-2** | ✅ Supported | Default region |
| **eu-west-1** | ✅ Supported | Full AZ support |
| **ap-southeast-1** | ✅ Supported | Full AZ support |

## Required Secrets

Ensure these GitHub Action secrets are configured in your repository:

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `AWS_ACCESS_KEY_ID` | AWS Access Key ID | `AKIAIOSFODNN7EXAMPLE` |
| `AWS_SECRET_ACCESS_KEY` | AWS Secret Access Key | `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` |

## Usage Examples

### Example 1: Deploy Development Environment
```yaml
environment: dev
ecr_services: frontend,backend,api,worker
action: plan
aws_region: us-west-2
```

### Example 2: Production Deployment
```yaml
environment: prod
ecr_services: web-app,api-gateway,auth-service,notification-service
action: apply
aws_region: us-east-1
```

### Example 3: Infrastructure Cleanup
```yaml
environment: staging
ecr_services: frontend,backend
action: destroy
aws_region: eu-west-1
```

## Output Values

After successful deployment, the workflow provides these outputs:

### EKS Cluster Outputs
- `eks_cluster_id` - The ID of the EKS cluster
- `eks_cluster_arn` - The ARN of the cluster
- `eks_cluster_endpoint` - Kubernetes API server endpoint
- `eks_cluster_version` - Kubernetes version
- `eks_node_group_arn` - Node group ARN
- `eks_node_group_status` - Node group status

### ECR Repository Outputs (per service)
- `ecr_<service>_repository_url` - Repository URL
- `ecr_<service>_repository_arn` - Repository ARN
- `ecr_<service>_repository_info` - Complete repository information

### Infrastructure Details
- `vpc_id` - VPC ID used
- `subnet_ids` - List of subnet IDs used

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: EKS Node Group Subnet Configuration Error
**Error**: `Attribute subnet_ids requires 1 item minimum, but config has only 0 declared`

**Solution**: ✅ **FIXED** - The workflow now properly passes `node_group_subnet_ids` parameter to the EKS module.

#### Issue 2: ECR Output Deprecation Warnings
**Error**: `The attribute "name" is deprecated. Refer to the provider documentation`

**Solution**: ✅ **FIXED** - Updated to use `repository_info` output which provides comprehensive repository details without deprecated attributes.

#### Issue 3: us-east-1e Availability Zone Error
**Error**: EKS does not support creating control plane instances in us-east-1e

**Solution**: ✅ **FIXED** - Automatic filtering excludes us-east-1e when deploying in us-east-1 region.

#### Issue 4: S3 Bucket Already Exists (Different Region)
**Error**: `BucketAlreadyOwnedByYou` or cross-region bucket issues

**Solution**: 
1. Check if bucket exists in different region
2. Delete existing bucket if safe to do so
3. Or modify the bucket naming pattern in the workflow

#### Issue 5: DynamoDB Table Already Exists
**Error**: `ResourceInUseException` when creating DynamoDB table

**Solution**: 
1. Check existing table configuration
2. Ensure table has correct schema (LockID as hash key)
3. Delete table if migration is needed

#### Issue 6: Insufficient AWS Permissions
**Error**: Various `AccessDenied` errors

**Required Permissions**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "eks:*",
        "ecr:*",
        "s3:*",
        "dynamodb:*",
        "iam:*",
        "kms:*",
        "ec2:*"
      ],
      "Resource": "*"
    }
  ]
}
```

### Debugging Steps

1. **Check Workflow Logs**: Review each job's detailed logs
2. **Validate Inputs**: Ensure all parameters meet validation requirements
3. **Terraform State**: Check S3 bucket for state file issues
4. **AWS Console**: Verify resources in AWS console
5. **Re-run with Plan**: Use `plan` action first to preview changes

### Getting Help

1. **Check workflow summary** for deployment details
2. **Review job logs** for specific error messages
3. **Validate AWS permissions** for the service account
4. **Check AWS quotas** for EKS and ECR in your region
5. **Verify network configuration** (VPC, subnets, security groups)

## Best Practices

### Security
- ✅ Use least-privilege IAM policies
- ✅ Enable encryption for all resources
- ✅ Regularly rotate AWS access keys
- ✅ Monitor resource access and usage

### Cost Optimization
- 🔄 Use `destroy` action to clean up dev/staging environments when not needed
- 📊 Monitor EKS node group scaling
- 📈 Review ECR storage costs and lifecycle policies
- 💰 Consider spot instances for non-production workloads

### Operational Excellence
- 📋 Always run `plan` before `apply` in production
- 🔄 Maintain environment parity where possible
- 📝 Document any manual changes outside Terraform
- 🚀 Use proper tagging for resource management

---

**Updated**: Latest version includes critical bug fixes for EKS node group configuration and ECR deprecation warnings.