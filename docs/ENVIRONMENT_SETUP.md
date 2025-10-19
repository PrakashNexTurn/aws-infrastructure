# Environment Setup Workflow

This document describes how to use the Environment Setup GitHub Actions workflow to provision EKS clusters and ECR repositories.

## 🎯 Overview

The Environment Setup workflow provisions AWS infrastructure using Terraform modules from the [terraform-scripts](https://github.com/PrakashNexTurn/terraform-scripts) repository. It creates:

- **EKS Cluster**: Managed Kubernetes cluster with auto-scaling node groups
- **ECR Repositories**: Container registries for each specified service
- **Backend Infrastructure**: S3 bucket and DynamoDB table for Terraform state management

## 🚀 Quick Start

### Prerequisites

1. **AWS Credentials**: Ensure the following secrets are configured in the repository:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`

2. **Permissions**: The AWS credentials must have permissions for:
   - EKS (cluster creation and management)
   - ECR (repository creation and management)
   - S3 (bucket creation and management)
   - DynamoDB (table creation and management)
   - VPC and networking (default VPC access)
   - IAM (roles and policies for EKS)

### Running the Workflow

1. Navigate to **Actions** → **Environment Setup** in the GitHub repository
2. Click **Run workflow**
3. Fill in the required parameters:
   - **Environment**: Choose `dev`, `staging`, or `prod`
   - **ECR Services**: Enter comma-separated service names (e.g., `frontend,backend,api`)
   - **Action**: Select `plan`, `apply`, or `destroy`
   - **AWS Region**: Choose target region (optional, defaults to `us-west-2`)

## 📋 Configuration Reference

### Environment-Specific Settings

| Setting | Dev | Staging | Prod |
|---------|-----|---------|------|
| **EKS Node Group** |
| Desired Size | 2 | 2 | 3 |
| Min Size | 1 | 1 | 2 |
| Max Size | 3 | 3 | 5 |
| Instance Type | t3.medium | t3.medium | t3.large |
| **ECR Settings** |
| Max Images | 10 | 10 | 20 |
| Retention (days) | 7 | 7 | 14 |
| Force Delete | true | true | false |

### Resource Naming Conventions

- **EKS Cluster**: `nextops-<environment>-eks`
- **ECR Repositories**: `nextops-<environment>-<service-name>`
- **S3 Backend**: `nextops-terraform-state-<environment>`
- **DynamoDB Lock**: `nextops-terraform-state-<environment>`

### Service Name Requirements

ECR service names must:
- Start with alphanumeric character
- Contain only lowercase letters, numbers, hyphens, underscores, and periods
- Examples: `frontend`, `auth-service`, `api_gateway`, `web.app`

## 🔧 Workflow Actions

### Plan (`plan`)
- Validates Terraform configuration
- Shows what changes will be made
- Uploads plan artifact for review
- **Use case**: Review changes before deployment

### Apply (`apply`)
- Executes the Terraform plan
- Provisions actual AWS resources
- Generates infrastructure summary
- **Use case**: Deploy infrastructure

### Destroy (`destroy`)
- Removes all provisioned resources
- Cleans up infrastructure
- **Use case**: Tear down environment

## 🏗️ Infrastructure Components

### EKS Cluster Features
- **Version**: Kubernetes 1.33
- **Security**: KMS encryption, private/public API endpoints
- **Logging**: Comprehensive control plane logs
- **Networking**: Uses default VPC and subnets
- **Auto-scaling**: Managed node groups with configurable sizing
- **Add-ons**: CoreDNS, VPC CNI, Kube Proxy

### ECR Repository Features
- **Security**: KMS encryption, vulnerability scanning
- **Lifecycle**: Automatic cleanup of old images
- **Access**: Private repositories with IAM-based access
- **Tagging**: Consistent tagging strategy
- **Protected Tags**: `latest`, `main`, `master`, `<environment>`, `stable`

### Backend Infrastructure
- **State Storage**: Versioned S3 bucket with encryption
- **State Locking**: DynamoDB table for concurrent access prevention
- **Security**: Public access blocked, server-side encryption
- **Backup**: Versioned state files for rollback capability

## 📊 Monitoring and Outputs

### Terraform Outputs

The workflow provides the following outputs:

**EKS Cluster**:
- `eks_cluster_id`: Cluster identifier
- `eks_cluster_arn`: Cluster ARN
- `eks_cluster_endpoint`: Kubernetes API endpoint
- `eks_cluster_version`: Kubernetes version
- `eks_node_group_arn`: Node group ARN
- `eks_node_group_status`: Node group status

**ECR Repositories** (per service):
- `ecr_<service>_repository_url`: Repository URL
- `ecr_<service>_repository_arn`: Repository ARN

### Workflow Summary

Each workflow run generates a comprehensive summary including:
- Configuration details
- Infrastructure components
- Backend configuration
- Deployment status
- Resource URLs and ARNs

## 🛡️ Security Considerations

### Encryption
- **EKS**: Cluster encryption enabled with KMS
- **ECR**: Repository encryption with customer-managed KMS keys
- **S3**: Server-side encryption for state files
- **DynamoDB**: Encryption at rest for lock table

### Access Control
- **EKS**: RBAC with AWS IAM integration
- **ECR**: IAM-based repository access
- **Backend**: S3 bucket policy and public access blocking
- **Network**: Security groups and VPC isolation

### Compliance
- **Scanning**: ECR vulnerability scanning enabled
- **Logging**: CloudTrail integration for audit trails
- **Tagging**: Consistent resource tagging for governance
- **Backup**: State versioning for disaster recovery

## 🚨 Important Constraints

### AWS Region Limitations
- **us-east-1e**: Not supported by EKS (workflow handles this automatically)
- **Availability Zones**: Requires at least 2 AZs in the region

### Resource Limits
- **ECR**: Repository names must be unique within AWS account
- **EKS**: One cluster per environment (managed by workflow)
- **State**: One state file per environment

### Cost Considerations
- **EKS**: Cluster costs ~$0.10/hour + node costs
- **ECR**: Storage costs for container images
- **Data Transfer**: Cross-AZ and internet transfer costs

## 🔄 Common Workflows

### Initial Environment Setup
1. Run with `action: plan` to review changes
2. Review the generated plan artifact
3. Run with `action: apply` to create infrastructure
4. Verify outputs and test connectivity

### Adding New Services
1. Update `ecr_services` parameter with new service names
2. Run with `action: plan` to see new ECR repositories
3. Run with `action: apply` to create new repositories

### Environment Updates
1. Modify Terraform modules in terraform-scripts repository
2. Run with `action: plan` to preview updates
3. Run with `action: apply` to implement changes

### Environment Cleanup
1. Ensure no critical data in ECR repositories
2. Run with `action: destroy` to remove all resources
3. Verify cleanup in AWS console

## 🐛 Troubleshooting

### Common Issues

**Invalid Service Names**
```
❌ Invalid service name: MyService. Must be lowercase alphanumeric...
```
- Solution: Use lowercase with hyphens/underscores only

**Backend Initialization Failures**
```
❌ Failed to initialize Terraform backend
```
- Check AWS credentials and permissions
- Verify S3 bucket and DynamoDB table access

**EKS Cluster Creation Failures**
```
❌ Cannot create cluster in us-east-1e
```
- Use different region or verify availability zones
- Check VPC and subnet configuration

**Terraform Plan Failures**
```
❌ Terraform plan failed with exit code: 1
```
- Review Terraform configuration syntax
- Verify module source accessibility
- Check variable values and constraints

### Getting Help

1. **Workflow Logs**: Check GitHub Actions logs for detailed error messages
2. **Terraform Output**: Review Terraform plan/apply output for specific errors
3. **AWS Console**: Verify resource states and permissions in AWS
4. **Module Documentation**: Check terraform-scripts repository for module updates

## 📚 Additional Resources

- [Terraform AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [EKS Best Practices Guide](https://aws.github.io/aws-eks-best-practices/)
- [ECR User Guide](https://docs.aws.amazon.com/AmazonECR/latest/userguide/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

---

**Workflow File**: `.github/workflows/environment-setup.yml`  
**Last Updated**: October 2024  
**Maintained By**: Platform Engineering Team