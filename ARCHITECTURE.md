# Architecture Overview

## Logical tiers

### Presentation tier
Static HTML, CSS, JavaScript, and images are stored in a private S3 bucket and delivered through CloudFront. The browser calls the PHP application through the application hostname.

### Application tier
The public Application Load Balancer forwards requests to PHP/Apache EC2 instances in private application subnets. The instances are managed by an Auto Scaling Group using a Launch Template and target tracking based on average CPU utilization.

### Data tier
Amazon RDS for MySQL is deployed as a Multi-AZ DB instance. The primary handles reads and writes. A synchronous standby in another AZ is maintained for automatic failover.

## Network layout

VPC CIDR: `10.0.0.0/16`

| Subnet | AZ A | AZ B |
|---|---|---|
| Public | 10.0.1.0/24 | 10.0.2.0/24 |
| Application private | 10.0.11.0/24 | 10.0.12.0/24 |
| Database private | 10.0.21.0/24 | 10.0.22.0/24 |

Public subnets host the internet-facing ALB and NAT Gateways. Application subnets host EC2 instances. Database subnets host RDS. The database subnets have no default route to the internet.

## Security boundaries

- ALB security group accepts HTTP from the internet in the starter configuration.
- Application security group accepts traffic only from the ALB security group.
- RDS security group accepts MySQL only from the application security group.
- Systems Manager is used for administration; do not open SSH to the internet.
- WAF is associated with the ALB and enables the AWS Managed Core Rule Set.
- Private S3 access is provided through CloudFront Origin Access Control.

## Operations

CloudWatch alarms monitor EC2 CPU and ALB unhealthy hosts. Alarm actions publish to SNS. Systems Manager Session Manager provides shell access without inbound SSH.
