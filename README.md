# AWS Three-Tier PHP/MySQL Web Application

An AWS-Solutions-style reference repository for a highly available PHP/MySQL web application. It is organized into `source/`, `deployment/`, `docs/`, and `.github/` so the application, infrastructure, operating procedures, and automation are easy to navigate.

> **Status:** educational reference implementation. Review the Terraform, costs, quotas, and security settings before deploying. It is not an AWS-certified solution or a production-ready turnkey product.

## Architecture

```text
                         Route 53
                    /                 \
 app.example.com (ALB alias)      static.example.com (CloudFront alias)
              |                              |
        AWS WAF on ALB                 CloudFront CDN
              |                              |
     Public Application ALB              Private S3 origin
              |
       Private EC2 Auto Scaling
        PHP + Apache application
              |
       RDS for MySQL Multi-AZ
       Primary + synchronous standby

Operations: Systems Manager Session Manager | CloudWatch | SNS
Network: VPC, 2 AZs, public/private/database subnets, NAT Gateways,
         Security Groups and custom Network ACLs
```

The application hostname points to the ALB to meet the assignment's Route 53 alias requirement. A separate static hostname points to CloudFront, which caches CSS, JavaScript, and images from a private S3 bucket. WAF is associated with the ALB and uses AWS Managed Rules.

## Assignment coverage

| Requirement | Implementation |
|---|---|
| VPC, public/private subnets | Terraform VPC across two AZs |
| NAT Gateway | One NAT Gateway per AZ |
| Security Groups | ALB, application, and RDS security groups |
| NACLs | Custom application/database subnet NACLs |
| EC2 + Launch Template | PHP/Apache EC2 launch template |
| Auto Scaling | Multi-AZ EC2 Auto Scaling Group |
| Target tracking | Average CPU utilization policy |
| ALB / Layer 7 | HTTP listener and `/health.php` health check; HTTPS instructions in docs |
| WAF / OWASP | AWS Managed Core Rule Set attached to ALB |
| CloudFront caching | Distribution with private S3 origin and cache policy |
| RDS Multi-AZ | MySQL Multi-AZ DB instance |
| Route 53 alias | App alias to ALB and static alias to CloudFront |
| Route 53 health checks | Optional health check for application endpoint |
| Systems Manager | EC2 instance role and SSM agent |
| CloudWatch | CPU and ALB alarms |
| SNS | Email subscription for alarm notifications |

## Repository layout

```text
.
├── .github/workflows/       # CI validation
├── deployment/
│   ├── terraform/           # AWS infrastructure as code
│   └── scripts/             # helper scripts
├── docs/                    # architecture, deployment, security, operations
├── source/
│   └── app/                 # PHP application and static assets
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── LICENSE
├── NOTICE
├── SECURITY.md
└── README.md
```

## Prerequisites

- AWS account and an IAM role/user with permissions for the resources in `deployment/terraform`
- AWS CLI v2
- Terraform >= 1.6
- A Route 53 hosted zone and domain (optional; DNS can be added after the core stack)
- A verified email address for SNS
- An ACM certificate in the ALB's region before enabling HTTPS

## Deploy

Read [`docs/DEPLOYMENT.md`](docs/DEPLOYMENT.md) before applying Terraform. Start with a sandbox account and a budget alert.

```bash
cd deployment/terraform
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars. Never commit terraform.tfvars.
terraform init
terraform fmt -recursive
terraform validate
terraform plan
terraform apply
```

The Terraform stack creates billable resources. In particular, NAT Gateways, the ALB, RDS Multi-AZ, EC2, CloudFront requests, and data transfer can incur charges.

## Run the app locally

Requirements: PHP 8.2+ with `pdo_mysql`.

```bash
cd source/app
cp .env.example .env
# Edit .env with local MySQL connection settings.
php -S 127.0.0.1:8080 -t public
```

Open `http://127.0.0.1:8080`. The `/health.php` endpoint is intended for ALB health checks.

## Important design notes

1. An RDS Multi-AZ standby is for failover; it is not a read replica and does not serve read traffic.
2. The app alias points to the ALB, while the static alias points to CloudFront. This avoids a DNS-path contradiction while satisfying both requirements.
3. The CloudFront S3 origin is private. Use Origin Access Control (OAC); do not enable public bucket access.
4. The starter ALB listener uses HTTP to avoid requiring a certificate during initial testing. Add an HTTPS listener with ACM before production and redirect HTTP to HTTPS.
5. The Terraform code creates infrastructure. Review the user-data bootstrap and app deployment approach before production; for production, bake an immutable AMI or use a controlled artifact pipeline.
6. Terraform state may contain sensitive infrastructure metadata. Use a secured remote backend with encryption, versioning, and tightly scoped access for team use.
7. This project intentionally does not include real AWS credentials, domain names, or database passwords.

## Cleanup

```bash
cd deployment/terraform
terraform destroy
```

Check the AWS console after destroy for retained resources such as S3 objects, log groups, snapshots, or resources created manually.

## License

Apache-2.0. See [`LICENSE`](LICENSE).
