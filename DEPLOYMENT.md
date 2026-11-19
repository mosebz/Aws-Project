# Deployment Guide

## 1. Prepare your AWS account

1. Choose an AWS Region with at least two Availability Zones.
2. Configure AWS CLI credentials using an IAM role or SSO profile.
3. Create a budget alert in AWS Budgets.
4. Ensure your account has sufficient quotas for EC2, VPC Elastic IPs, NAT Gateways, RDS, and ALB.
5. If using DNS, create a Route 53 hosted zone and obtain an ACM certificate in the ALB Region.

## 2. Configure Terraform

```bash
cd deployment/terraform
cp terraform.tfvars.example terraform.tfvars
```

Set a unique project name, region, notification email, database password, and optional DNS values. Do not commit `terraform.tfvars`.

For production, prefer storing Terraform state in a dedicated encrypted S3 backend with versioning and a lock mechanism. The starter uses local state for ease of learning.

## 3. Validate and deploy

```bash
terraform init
terraform fmt -recursive
terraform validate
terraform plan
terraform apply
```

Read the plan carefully. Confirm the region, CIDR ranges, instance sizing, and estimated costs before approving.

## 4. Confirm the stack

- Open the ALB DNS name from Terraform outputs.
- Confirm the target group reports healthy instances.
- Test `/health.php`.
- Confirm EC2 instances appear as managed nodes in Systems Manager.
- Confirm the RDS instance is Multi-AZ.
- Confirm CloudWatch alarms exist and the SNS subscription email is confirmed.

## 5. DNS, CloudFront, and HTTPS

The Terraform starter creates the ALB, WAF, S3 bucket, and CloudFront distribution. Provide a Route 53 hosted zone ID and hostnames to create aliases:
- `app.example.com` -> ALB alias.
- `static.example.com` -> CloudFront alias.

For CloudFront custom domains, the ACM certificate must be in `us-east-1`. The current starter uses the default CloudFront domain unless you extend the variables and certificate configuration.

For ALB HTTPS, request/import an ACM certificate in the same Region as the ALB, add an HTTPS listener on port 443, and redirect port 80 to 443. Do not expose the HTTP-only starter to production.

## 6. Destroy resources

```bash
terraform destroy
```

RDS deletion protection is disabled in this learning starter. Back up any data you need before destroying the stack.
