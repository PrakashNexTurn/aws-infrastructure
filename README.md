# AWS Infrastructure

This repository contains GitHub Actions workflows and infrastructure configurations for provisioning AWS resources using Terraform modules from the [terraform-scripts](https://github.com/PrakashNexTurn/terraform-scripts) repository.

## 🚀 Quick Start

### Environment Setup Workflow

The Environment Setup workflow provisions complete environments with EKS clusters and ECR repositories:

1. Navigate to **Actions** → **Environment Setup**
2. Click **Run workflow**
3. Configure parameters:
   - **Environment**: `dev`, `staging`, or `prod`
   - **ECR Services**: Comma-separated service names
   - **Action**: `plan`, `apply`, or `destroy`
   - **AWS Region**: Target region (defaults to `us-west-2`)

### Prerequisites

Configure the following repository secrets:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

## 📋 Available Workflows

| Workflow | Description | File |
|----------|-------------|------|
| **Environment Setup** | Provision EKS clusters and ECR repositories | [environment-setup.yml](.github/workflows/environment-setup.yml) |

## 🏗️ Infrastructure Components

### Provisioned Resources

- **EKS Cluster**: Managed Kubernetes clusters with auto-scaling node groups
- **ECR Repositories**: Container registries for each specified service
- **Backend**: S3 bucket and DynamoDB table for Terraform state management

### Resource Naming

- **EKS Cluster**: `nextops-<environment>-eks`
- **ECR Repositories**: `nextops-<environment>-<service-name>`
- **S3 Backend**: `nextops-terraform-state-<environment>`
- **DynamoDB Lock**: `nextops-terraform-state-<environment>`

## 📚 Documentation

- [Environment Setup Guide](docs/ENVIRONMENT_SETUP.md) - Comprehensive workflow documentation
- [Terraform Modules](https://github.com/PrakashNexTurn/terraform-scripts) - Source modules repository

## 🛡️ Security Features

- ✅ KMS encryption for all resources
- ✅ ECR vulnerability scanning
- ✅ S3 bucket versioning and public access blocking
- ✅ Terraform state locking
- ✅ Comprehensive resource tagging

## 🔧 Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   S3 Backend    │───▶│      ECR        │───▶│      EKS        │
│ (State Store)   │    │  (Registries)   │    │   (Cluster)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                    DynamoDB State Locking                      │
└─────────────────────────────────────────────────────────────────┘
```

## 🎯 Usage Examples

### Deploy Development Environment
```yaml
environment: dev
ecr_services: frontend,backend,api
action: apply
aws_region: us-west-2
```

### Plan Production Changes
```yaml
environment: prod
ecr_services: web-app,auth-service,payment-api
action: plan
aws_region: us-east-1
```

## 🤝 Contributing

1. Create a feature branch: `git checkout -b feature/DEVOPS-XXX`
2. Make your changes
3. Test thoroughly
4. Submit a pull request with descriptive commit messages starting with the Jira ID

## 📞 Support

- **Issues**: Create GitHub issues for bugs or feature requests
- **Documentation**: Check the [docs](docs/) directory for detailed guides
- **Modules**: Visit [terraform-scripts](https://github.com/PrakashNexTurn/terraform-scripts) for module documentation

---

**Maintained by**: Platform Engineering Team  
**Last Updated**: October 2024