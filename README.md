# ☁️ AWS for DevOps — Complete Cheat Sheet

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazon-aws)
![EC2](https://img.shields.io/badge/EC2-Compute-yellow?logo=amazon-aws)
![S3](https://img.shields.io/badge/S3-Storage-green?logo=amazon-aws)
![EKS](https://img.shields.io/badge/EKS-Kubernetes-blue?logo=kubernetes)
![Terraform](https://img.shields.io/badge/Terraform-IaC-purple?logo=terraform)
![DevOps](https://img.shields.io/badge/DevOps-Engineer-red)

A complete AWS reference for DevOps Engineers — EC2, S3, VPC, IAM, EKS, ECR, Lambda, RDS, CloudFront, Route 53, CloudWatch, and more.

---

## 📑 Table of Contents

1. [AWS CLI Setup](#1-aws-cli-setup)
2. [IAM — Identity & Access Management](#2-iam--identity--access-management)
3. [EC2 — Elastic Compute Cloud](#3-ec2--elastic-compute-cloud)
4. [S3 — Simple Storage Service](#4-s3--simple-storage-service)
5. [VPC — Virtual Private Cloud](#5-vpc--virtual-private-cloud)
6. [Security Groups & NACLs](#6-security-groups--nacls)
7. [RDS — Relational Database Service](#7-rds--relational-database-service)
8. [EKS — Elastic Kubernetes Service](#8-eks--elastic-kubernetes-service)
9. [ECR — Elastic Container Registry](#9-ecr--elastic-container-registry)
10. [Lambda — Serverless](#10-lambda--serverless)
11. [API Gateway](#11-api-gateway)
12. [CloudFront — CDN](#12-cloudfront--cdn)
13. [Route 53 — DNS](#13-route-53--dns)
14. [CloudWatch — Monitoring](#14-cloudwatch--monitoring)
15. [CloudFormation — IaC](#15-cloudformation--iac)
16. [ELB — Elastic Load Balancer](#16-elb--elastic-load-balancer)
17. [Auto Scaling](#17-auto-scaling)
18. [SNS & SQS — Messaging](#18-sns--sqs--messaging)
19. [CodePipeline & CodeBuild — CI/CD](#19-codepipeline--codebuild--cicd)
20. [Secrets Manager & SSM](#20-secrets-manager--ssm)

---

## 1. AWS CLI Setup

### Install AWS CLI

```bash
# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Verify
aws --version
```

### Configure AWS CLI

```bash
aws configure
# AWS Access Key ID: YOUR_ACCESS_KEY
# AWS Secret Access Key: YOUR_SECRET_KEY
# Default region name: us-east-1
# Default output format: json
```

### Multiple Profiles

```bash
# Configure named profile
aws configure --profile dev
aws configure --profile prod

# Use a specific profile
aws s3 ls --profile dev
export AWS_PROFILE=prod
```

### Essential CLI Commands

| Command | Description |
|---------|-------------|
| `aws configure` | Configure AWS credentials |
| `aws configure list` | Show current configuration |
| `aws sts get-caller-identity` | Show current account and user |
| `aws configure list-profiles` | List all configured profiles |
| `aws --version` | Show AWS CLI version |

---

## 2. IAM — Identity & Access Management

### Key Concepts

| Concept | Description |
|---------|-------------|
| **User** | Individual identity with credentials |
| **Group** | Collection of users with shared permissions |
| **Role** | Identity assumed by services or users temporarily |
| **Policy** | JSON document defining permissions |
| **ARN** | Amazon Resource Name — unique identifier |

### IAM Users

```bash
# List all users
aws iam list-users

# Create user
aws iam create-user --user-name manish-deore

# Delete user
aws iam delete-user --user-name manish-deore

# Create access key for user
aws iam create-access-key --user-name manish-deore

# List access keys
aws iam list-access-keys --user-name manish-deore

# Delete access key
aws iam delete-access-key \
  --user-name manish-deore \
  --access-key-id AKIAIOSFODNN7EXAMPLE
```

### IAM Groups

```bash
# Create group
aws iam create-group --group-name DevOps-Team

# Add user to group
aws iam add-user-to-group \
  --user-name manish-deore \
  --group-name DevOps-Team

# List groups
aws iam list-groups

# List users in group
aws iam get-group --group-name DevOps-Team

# Remove user from group
aws iam remove-user-from-group \
  --user-name manish-deore \
  --group-name DevOps-Team
```

### IAM Roles

```bash
# List all roles
aws iam list-roles

# Create role (from trust policy file)
aws iam create-role \
  --role-name EC2-S3-Role \
  --assume-role-policy-document file://trust-policy.json

# Attach managed policy to role
aws iam attach-role-policy \
  --role-name EC2-S3-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

# List policies attached to role
aws iam list-attached-role-policies --role-name EC2-S3-Role
```

### IAM Policies

```bash
# List all policies
aws iam list-policies --scope Local

# Create custom policy
aws iam create-policy \
  --policy-name MyDevOpsPolicy \
  --policy-document file://policy.json

# Get policy details
aws iam get-policy \
  --policy-arn arn:aws:iam::123456789:policy/MyDevOpsPolicy

# Attach policy to user
aws iam attach-user-policy \
  --user-name manish-deore \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
```

### Sample IAM Policy (Least Privilege)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

---

## 3. EC2 — Elastic Compute Cloud

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Instance** | Virtual server in the cloud |
| **AMI** | Amazon Machine Image — OS template |
| **Instance Type** | Hardware configuration (t2.micro, m5.large) |
| **Key Pair** | SSH key for secure access |
| **Security Group** | Virtual firewall for instances |
| **EBS** | Elastic Block Store — persistent storage |
| **Elastic IP** | Static public IP address |
| **Spot Instance** | Unused capacity at up to 90% discount |
| **Reserved Instance** | 1-3 year commitment for up to 75% discount |

### EC2 Instance Management

```bash
# List all instances
aws ec2 describe-instances

# List running instances only
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running"

# Start instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Reboot instance
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0

# Terminate instance
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Get instance public IP
aws ec2 describe-instances \
  --instance-ids i-1234567890abcdef0 \
  --query 'Reservations[0].Instances[0].PublicIpAddress'
```

### Launch EC2 Instance

```bash
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type t2.micro \
  --key-name my-key-pair \
  --security-group-ids sg-12345678 \
  --subnet-id subnet-12345678 \
  --count 1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyServer}]'
```

### Key Pairs

```bash
# Create key pair
aws ec2 create-key-pair \
  --key-name my-key-pair \
  --query 'KeyMaterial' \
  --output text > my-key-pair.pem

chmod 400 my-key-pair.pem

# List key pairs
aws ec2 describe-key-pairs

# Delete key pair
aws ec2 delete-key-pair --key-name my-key-pair
```

### EBS Volumes

```bash
# List volumes
aws ec2 describe-volumes

# Create volume
aws ec2 create-volume \
  --size 20 \
  --volume-type gp3 \
  --availability-zone us-east-1a

# Attach volume to instance
aws ec2 attach-volume \
  --volume-id vol-1234567890abcdef0 \
  --instance-id i-1234567890abcdef0 \
  --device /dev/sdf

# Detach volume
aws ec2 detach-volume --volume-id vol-1234567890abcdef0

# Delete volume
aws ec2 delete-volume --volume-id vol-1234567890abcdef0
```

### Elastic IPs

```bash
# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

# Associate with instance
aws ec2 associate-address \
  --instance-id i-1234567890abcdef0 \
  --allocation-id eipalloc-12345678

# Release Elastic IP
aws ec2 release-address --allocation-id eipalloc-12345678
```

### EC2 Instance Types Quick Reference

| Family | Type | Use Case |
|--------|------|---------|
| General | t3, t2, m5 | Web servers, dev/test |
| Compute | c5, c6g | High CPU — batch processing |
| Memory | r5, x1 | Databases, in-memory cache |
| Storage | i3, d2 | High I/O — NoSQL databases |
| GPU | p3, g4 | ML, video rendering |

---

## 4. S3 — Simple Storage Service

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Bucket** | Container for objects (like a folder) |
| **Object** | File stored in S3 (up to 5TB) |
| **Key** | Unique identifier for an object |
| **Versioning** | Keep multiple versions of an object |
| **Lifecycle Policy** | Automatically transition or delete objects |
| **ACL** | Access Control List |
| **Pre-signed URL** | Temporary URL for private objects |

### Bucket Operations

```bash
# List all buckets
aws s3 ls

# Create bucket
aws s3 mb s3://my-bucket-name

# Create bucket in specific region
aws s3 mb s3://my-bucket-name --region ap-south-1

# Delete empty bucket
aws s3 rb s3://my-bucket-name

# Delete bucket with all contents
aws s3 rb s3://my-bucket-name --force
```

### File Operations

```bash
# Upload file
aws s3 cp file.txt s3://my-bucket/

# Upload with custom storage class
aws s3 cp file.txt s3://my-bucket/ --storage-class GLACIER

# Download file
aws s3 cp s3://my-bucket/file.txt ./

# Move file
aws s3 mv file.txt s3://my-bucket/

# Delete file
aws s3 rm s3://my-bucket/file.txt

# List objects in bucket
aws s3 ls s3://my-bucket/

# List with sizes
aws s3 ls s3://my-bucket/ --human-readable --summarize
```

### Sync Operations

```bash
# Sync local folder to S3
aws s3 sync ./local-folder s3://my-bucket/

# Sync S3 to local folder
aws s3 sync s3://my-bucket/ ./local-folder

# Sync with delete (removes files not in source)
aws s3 sync ./local-folder s3://my-bucket/ --delete

# Sync with exclusions
aws s3 sync ./local-folder s3://my-bucket/ \
  --exclude "*.tmp" \
  --exclude ".git/*"
```

### S3 Storage Classes

| Class | Use Case | Retrieval |
|-------|---------|-----------|
| **Standard** | Frequently accessed data | Instant |
| **Intelligent-Tiering** | Unknown access patterns | Instant |
| **Standard-IA** | Infrequently accessed | Instant |
| **One Zone-IA** | Infrequent, non-critical | Instant |
| **Glacier Instant** | Archive with instant access | Instant |
| **Glacier Flexible** | Archive, 1-5 min retrieval | Minutes |
| **Glacier Deep Archive** | Long-term archive | Hours |

### S3 Bucket Policy Example

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

---

## 5. VPC — Virtual Private Cloud

### Key Concepts

| Concept | Description |
|---------|-------------|
| **VPC** | Isolated virtual network in AWS |
| **Subnet** | Range of IP addresses within a VPC |
| **Public Subnet** | Has route to Internet Gateway |
| **Private Subnet** | No direct internet access |
| **Internet Gateway (IGW)** | Enables VPC internet access |
| **NAT Gateway** | Allows private subnet outbound internet access |
| **Route Table** | Rules directing network traffic |
| **VPC Peering** | Connect two VPCs |
| **VPN Gateway** | Connect on-prem network to VPC |

### VPC Operations

```bash
# List VPCs
aws ec2 describe-vpcs

# Create VPC
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Create subnet
aws ec2 create-subnet \
  --vpc-id vpc-12345678 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a

# Create Internet Gateway
aws ec2 create-internet-gateway

# Attach IGW to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-12345678 \
  --vpc-id vpc-12345678

# Create NAT Gateway
aws ec2 create-nat-gateway \
  --subnet-id subnet-12345678 \
  --allocation-id eipalloc-12345678

# List Route Tables
aws ec2 describe-route-tables

# Add route to route table
aws ec2 create-route \
  --route-table-id rtb-12345678 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-12345678
```

### Typical VPC Architecture

```
VPC: 10.0.0.0/16
│
├── Public Subnet: 10.0.1.0/24  (us-east-1a)
│     └── EC2 Web Server ←→ Internet Gateway ←→ Internet
│
├── Public Subnet: 10.0.2.0/24  (us-east-1b)
│     └── EC2 Web Server (HA)
│
├── Private Subnet: 10.0.3.0/24 (us-east-1a)
│     └── EC2 App Server → NAT Gateway → Internet
│
└── Private Subnet: 10.0.4.0/24 (us-east-1b)
      └── RDS Database (no internet access)
```

---

## 6. Security Groups & NACLs

### Security Groups vs NACLs

| Feature | Security Group | NACL |
|---------|---------------|------|
| Level | Instance level | Subnet level |
| State | Stateful | Stateless |
| Rules | Allow only | Allow AND Deny |
| Default Inbound | Deny all | Allow all |
| Default Outbound | Allow all | Allow all |

### Security Group Commands

```bash
# List security groups
aws ec2 describe-security-groups

# Create security group
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web Server Security Group" \
  --vpc-id vpc-12345678

# Allow HTTP (port 80)
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow HTTPS (port 443)
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from my IP only
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 22 \
  --cidr MY-IP/32

# Revoke a rule
aws ec2 revoke-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Delete security group
aws ec2 delete-security-group --group-id sg-12345678
```

---

## 7. RDS — Relational Database Service

### Key Concepts

| Concept | Description |
|---------|-------------|
| **DB Instance** | Isolated database environment |
| **DB Subnet Group** | Set of subnets for RDS (multi-AZ) |
| **Multi-AZ** | Automatic failover to standby instance |
| **Read Replica** | Read-only copy for performance |
| **Snapshot** | Manual backup of DB instance |
| **Parameter Group** | DB engine configuration settings |

### RDS Commands

```bash
# List DB instances
aws rds describe-db-instances

# Create MySQL RDS instance
aws rds create-db-instance \
  --db-instance-identifier mydb \
  --db-instance-class db.t3.micro \
  --engine mysql \
  --engine-version 8.0 \
  --master-username admin \
  --master-user-password MyPassword123 \
  --allocated-storage 20 \
  --vpc-security-group-ids sg-12345678 \
  --db-subnet-group-name my-subnet-group \
  --no-publicly-accessible

# Start DB instance
aws rds start-db-instance --db-instance-identifier mydb

# Stop DB instance
aws rds stop-db-instance --db-instance-identifier mydb

# Delete DB instance
aws rds delete-db-instance \
  --db-instance-identifier mydb \
  --skip-final-snapshot

# Create snapshot
aws rds create-db-snapshot \
  --db-instance-identifier mydb \
  --db-snapshot-identifier mydb-snapshot-2025

# List snapshots
aws rds describe-db-snapshots

# Create read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier mydb-replica \
  --source-db-instance-identifier mydb
```

### Supported RDS Engines

| Engine | Versions |
|--------|---------|
| MySQL | 5.7, 8.0 |
| PostgreSQL | 13, 14, 15 |
| MariaDB | 10.5, 10.6 |
| Oracle | SE2, EE |
| SQL Server | Express, Web, Standard, Enterprise |
| Aurora MySQL | Compatible with MySQL 5.7, 8.0 |
| Aurora PostgreSQL | Compatible with PostgreSQL 13, 14 |

---

## 8. EKS — Elastic Kubernetes Service

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Cluster** | Control plane + worker nodes |
| **Node Group** | Group of EC2 instances as worker nodes |
| **Fargate Profile** | Serverless compute for pods |
| **OIDC Provider** | For IAM roles for service accounts |
| **kubeconfig** | Configuration file for kubectl |

### EKS Commands

```bash
# List clusters
aws eks list-clusters

# Create cluster
aws eks create-cluster \
  --name my-cluster \
  --kubernetes-version 1.29 \
  --role-arn arn:aws:iam::123456789:role/EKSClusterRole \
  --resources-vpc-config \
    subnetIds=subnet-111,subnet-222,\
    securityGroupIds=sg-12345678

# Describe cluster
aws eks describe-cluster --name my-cluster

# Update kubeconfig (connect kubectl to EKS)
aws eks update-kubeconfig \
  --name my-cluster \
  --region us-east-1

# Create node group
aws eks create-nodegroup \
  --cluster-name my-cluster \
  --nodegroup-name my-nodes \
  --node-role arn:aws:iam::123456789:role/EKSNodeRole \
  --subnets subnet-111 subnet-222 \
  --instance-types t3.medium \
  --scaling-config minSize=2,maxSize=5,desiredSize=3

# List node groups
aws eks list-nodegroups --cluster-name my-cluster

# Delete node group
aws eks delete-nodegroup \
  --cluster-name my-cluster \
  --nodegroup-name my-nodes

# Delete cluster
aws eks delete-cluster --name my-cluster
```

### EKS + kubectl Workflow

```bash
# Connect kubectl to EKS
aws eks update-kubeconfig --name my-cluster --region us-east-1

# Verify connection
kubectl get nodes
kubectl get pods -A

# Deploy application
kubectl apply -f deployment.yaml

# Check deployment
kubectl get deployments
kubectl get pods
kubectl get svc
```

---

## 9. ECR — Elastic Container Registry

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Repository** | Stores Docker images |
| **Image** | Container image stored in ECR |
| **Image Tag** | Version label for image (e.g. latest, v1.0) |
| **Lifecycle Policy** | Auto-delete old images |

### ECR Commands

```bash
# List repositories
aws ecr describe-repositories

# Create repository
aws ecr create-repository \
  --repository-name my-app \
  --image-scanning-configuration scanOnPush=true

# Delete repository
aws ecr delete-repository \
  --repository-name my-app \
  --force

# List images in repo
aws ecr list-images --repository-name my-app

# Login to ECR (authenticate Docker)
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS \
  --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com

# Build and push Docker image to ECR
ECR_URI=123456789.dkr.ecr.us-east-1.amazonaws.com/my-app

docker build -t my-app .
docker tag my-app:latest $ECR_URI:latest
docker push $ECR_URI:latest

# Pull image from ECR
docker pull $ECR_URI:latest

# Delete image
aws ecr batch-delete-image \
  --repository-name my-app \
  --image-ids imageTag=latest
```

---

## 10. Lambda — Serverless

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Function** | Code that runs in response to events |
| **Trigger** | Event that invokes the function |
| **Layer** | Shared libraries/dependencies for functions |
| **Concurrency** | Number of simultaneous executions |
| **Cold Start** | First invocation delay |
| **Timeout** | Max execution time (up to 15 min) |

### Lambda Commands

```bash
# List functions
aws lambda list-functions

# Create function
aws lambda create-function \
  --function-name my-function \
  --runtime python3.11 \
  --role arn:aws:iam::123456789:role/LambdaRole \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://function.zip

# Invoke function
aws lambda invoke \
  --function-name my-function \
  --payload '{"key": "value"}' \
  output.json

# Invoke with log
aws lambda invoke \
  --function-name my-function \
  --log-type Tail \
  output.json

# Update function code
aws lambda update-function-code \
  --function-name my-function \
  --zip-file fileb://function.zip

# Update environment variables
aws lambda update-function-configuration \
  --function-name my-function \
  --environment Variables={KEY=value,KEY2=value2}

# Delete function
aws lambda delete-function --function-name my-function

# Get function logs
aws logs tail /aws/lambda/my-function --follow
```

### Lambda Supported Runtimes

| Language | Runtime |
|----------|---------|
| Python | python3.11, python3.12 |
| Node.js | nodejs18.x, nodejs20.x |
| Java | java11, java17, java21 |
| Go | provided.al2 |
| Ruby | ruby3.2 |
| .NET | dotnet6, dotnet8 |

---

## 11. API Gateway

### Key Concepts

| Concept | Description |
|---------|-------------|
| **REST API** | Traditional HTTP API |
| **HTTP API** | Lighter, faster, cheaper than REST |
| **WebSocket API** | Real-time two-way communication |
| **Stage** | Deployment environment (dev, prod) |
| **Resource** | URL path in the API |
| **Method** | HTTP method (GET, POST, etc.) |
| **Integration** | Backend connection (Lambda, HTTP, etc.) |

### API Gateway Commands

```bash
# List REST APIs
aws apigateway get-rest-apis

# List HTTP APIs
aws apigatewayv2 get-apis

# Create REST API
aws apigateway create-rest-api \
  --name my-api \
  --description "My DevOps API"

# Get resources
aws apigateway get-resources --rest-api-id abc123

# Create deployment
aws apigateway create-deployment \
  --rest-api-id abc123 \
  --stage-name prod

# Get stages
aws apigateway get-stages --rest-api-id abc123

# Delete REST API
aws apigateway delete-rest-api --rest-api-id abc123
```

---

## 12. CloudFront — CDN

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Distribution** | A CloudFront deployment |
| **Origin** | Source of content (S3, EC2, ALB) |
| **Edge Location** | AWS data center closest to user |
| **TTL** | How long content is cached at edge |
| **Invalidation** | Force fresh content from origin |
| **OAC** | Origin Access Control — restricts S3 to CloudFront only |
| **Behaviors** | Cache rules for different URL patterns |

### CloudFront Commands

```bash
# List distributions
aws cloudfront list-distributions

# Get distribution details
aws cloudfront get-distribution --id EDFDVBD6EXAMPLE

# Create invalidation (clear cache)
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/*"

# Invalidate specific files
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/index.html" "/css/*"

# List invalidations
aws cloudfront list-invalidations \
  --distribution-id EDFDVBD6EXAMPLE

# Enable distribution
aws cloudfront update-distribution \
  --id EDFDVBD6EXAMPLE \
  --if-match ETVPDKIKX0DER \
  --distribution-config file://dist-config.json
```

---

## 13. Route 53 — DNS

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Hosted Zone** | Container for DNS records for a domain |
| **Record Set** | DNS record (A, CNAME, MX, etc.) |
| **Alias Record** | AWS-specific CNAME pointing to AWS resources |
| **Health Check** | Monitor endpoint health |
| **Routing Policy** | How to route traffic |

### Routing Policies

| Policy | Description | Use Case |
|--------|-------------|---------|
| **Simple** | Single resource | Basic routing |
| **Weighted** | Split traffic by % | A/B testing |
| **Latency** | Route to lowest latency region | Global apps |
| **Failover** | Primary/backup | High availability |
| **Geolocation** | Route by user's location | Region-specific content |
| **Multivalue** | Multiple IPs returned | Simple load balancing |

### Route 53 Commands

```bash
# List hosted zones
aws route53 list-hosted-zones

# Create hosted zone
aws route53 create-hosted-zone \
  --name example.com \
  --caller-reference unique-string-2025

# List records in a zone
aws route53 list-resource-record-sets \
  --hosted-zone-id Z1234567890

# Create/Update DNS record (A record)
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234567890 \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "app.example.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{"Value": "1.2.3.4"}]
      }
    }]
  }'

# Delete hosted zone
aws route53 delete-hosted-zone --id Z1234567890

# List health checks
aws route53 list-health-checks
```

---

## 14. CloudWatch — Monitoring

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Metrics** | Time-series data from AWS services |
| **Alarms** | Triggers actions based on metric thresholds |
| **Logs** | Collect and store log data |
| **Log Group** | Container for log streams |
| **Log Stream** | Sequence of log events |
| **Dashboard** | Visual display of metrics |
| **Events/EventBridge** | Trigger actions based on events |

### CloudWatch Commands

```bash
# List metrics
aws cloudwatch list-metrics --namespace AWS/EC2

# Get metric statistics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time 2025-01-01T00:00:00Z \
  --end-time 2025-01-02T00:00:00Z \
  --period 3600 \
  --statistics Average

# List alarms
aws cloudwatch describe-alarms

# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu \
  --alarm-description "CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789:my-topic

# Delete alarm
aws cloudwatch delete-alarms --alarm-names high-cpu

# List log groups
aws logs describe-log-groups

# Create log group
aws logs create-log-group --log-group-name /myapp/logs

# Tail logs (follow live)
aws logs tail /aws/lambda/my-function --follow

# Get log events
aws logs get-log-events \
  --log-group-name /myapp/logs \
  --log-stream-name my-stream
```

---

## 15. CloudFormation — IaC

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Stack** | Collection of AWS resources defined in a template |
| **Template** | JSON/YAML file defining resources |
| **Change Set** | Preview changes before applying |
| **Drift** | Difference between template and actual resources |
| **Nested Stack** | Stack within a stack |

### CloudFormation Commands

```bash
# List stacks
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE

# Create stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=EnvName,ParameterValue=prod \
  --capabilities CAPABILITY_IAM

# Update stack
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_IAM

# Describe stack
aws cloudformation describe-stacks --stack-name my-stack

# Create change set (preview changes)
aws cloudformation create-change-set \
  --stack-name my-stack \
  --change-set-name my-changes \
  --template-body file://template.yaml

# Execute change set
aws cloudformation execute-change-set \
  --stack-name my-stack \
  --change-set-name my-changes

# Delete stack
aws cloudformation delete-stack --stack-name my-stack

# Validate template
aws cloudformation validate-template \
  --template-body file://template.yaml
```

### Simple CloudFormation Template

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Simple EC2 instance'

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c02fb55956c7d316
      InstanceType: !Ref InstanceType
      Tags:
        - Key: Name
          Value: MyServer

Outputs:
  InstanceId:
    Value: !Ref MyEC2Instance
```

---

## 16. ELB — Elastic Load Balancer

### Load Balancer Types

| Type | Layer | Use Case |
|------|-------|---------|
| **ALB** (Application) | Layer 7 | HTTP/HTTPS, path-based routing, microservices |
| **NLB** (Network) | Layer 4 | TCP/UDP, ultra-low latency, gaming |
| **CLB** (Classic) | Layer 4 & 7 | Legacy — avoid for new projects |
| **GWLB** (Gateway) | Layer 3 | Firewalls, intrusion detection |

### ELB Commands

```bash
# List load balancers
aws elbv2 describe-load-balancers

# Create ALB
aws elbv2 create-load-balancer \
  --name my-alb \
  --type application \
  --subnets subnet-111 subnet-222 \
  --security-groups sg-12345678

# Create target group
aws elbv2 create-target-group \
  --name my-targets \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-12345678 \
  --health-check-path /health

# Register targets
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --targets Id=i-1234567890abcdef0

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:... \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=arn:...

# Describe target health
aws elbv2 describe-target-health \
  --target-group-arn arn:aws:elasticloadbalancing:...

# Delete load balancer
aws elbv2 delete-load-balancer \
  --load-balancer-arn arn:aws:elasticloadbalancing:...
```

---

## 17. Auto Scaling

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Launch Template** | EC2 configuration for scaling |
| **Auto Scaling Group** | Group of EC2 instances managed together |
| **Scaling Policy** | Rules for when to scale in/out |
| **Cooldown** | Wait period between scaling actions |
| **Target Tracking** | Scale to maintain a metric target |

### Auto Scaling Commands

```bash
# List Auto Scaling Groups
aws autoscaling describe-auto-scaling-groups

# Create Auto Scaling Group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-template,Version='$Latest' \
  --min-size 1 \
  --max-size 5 \
  --desired-capacity 2 \
  --vpc-zone-identifier "subnet-111,subnet-222" \
  --target-group-arns arn:aws:elasticloadbalancing:...

# Update desired capacity
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --desired-capacity 3

# Set scaling policy (target tracking)
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 50.0
  }'

# Delete Auto Scaling Group
aws autoscaling delete-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --force-delete
```

---

## 18. SNS & SQS — Messaging

### SNS — Simple Notification Service

```bash
# List topics
aws sns list-topics

# Create topic
aws sns create-topic --name my-topic

# Subscribe email to topic
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789:my-topic \
  --protocol email \
  --notification-endpoint myemail@example.com

# Publish message
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789:my-topic \
  --message "Hello from SNS!" \
  --subject "Test Notification"

# Delete topic
aws sns delete-topic \
  --topic-arn arn:aws:sns:us-east-1:123456789:my-topic
```

### SQS — Simple Queue Service

```bash
# List queues
aws sqs list-queues

# Create queue
aws sqs create-queue --queue-name my-queue

# Create FIFO queue
aws sqs create-queue \
  --queue-name my-queue.fifo \
  --attributes FifoQueue=true

# Send message
aws sqs send-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789/my-queue \
  --message-body "Hello from SQS!"

# Receive message
aws sqs receive-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789/my-queue

# Delete message
aws sqs delete-message \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789/my-queue \
  --receipt-handle HANDLE

# Delete queue
aws sqs delete-queue \
  --queue-url https://sqs.us-east-1.amazonaws.com/123456789/my-queue
```

---

## 19. CodePipeline & CodeBuild — CI/CD

### CodePipeline Commands

```bash
# List pipelines
aws codepipeline list-pipelines

# Get pipeline details
aws codepipeline get-pipeline --name my-pipeline

# Start pipeline execution
aws codepipeline start-pipeline-execution --name my-pipeline

# Get pipeline state
aws codepipeline get-pipeline-state --name my-pipeline

# List pipeline executions
aws codepipeline list-pipeline-executions --pipeline-name my-pipeline

# Delete pipeline
aws codepipeline delete-pipeline --name my-pipeline
```

### CodeBuild Commands

```bash
# List projects
aws codebuild list-projects

# Start build
aws codebuild start-build --project-name my-project

# Get build status
aws codebuild batch-get-builds --ids build-id

# List builds for project
aws codebuild list-builds-for-project --project-name my-project

# Stop build
aws codebuild stop-build --id build-id
```

### Sample buildspec.yml

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
    commands:
      - npm install

  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password | docker login --username AWS \
          --password-stdin $ECR_URI

  build:
    commands:
      - echo Building Docker image...
      - docker build -t $IMAGE_TAG .
      - docker tag $IMAGE_TAG $ECR_URI:$IMAGE_TAG

  post_build:
    commands:
      - echo Pushing image to ECR...
      - docker push $ECR_URI:$IMAGE_TAG

artifacts:
  files:
    - imageDefinitions.json
```

---

## 20. Secrets Manager & SSM

### Secrets Manager

```bash
# List secrets
aws secretsmanager list-secrets

# Create secret
aws secretsmanager create-secret \
  --name my-db-password \
  --description "Database password" \
  --secret-string "MySecretPassword123"

# Create secret as JSON
aws secretsmanager create-secret \
  --name my-db-creds \
  --secret-string '{"username":"admin","password":"MyPass123"}'

# Get secret value
aws secretsmanager get-secret-value \
  --secret-id my-db-password

# Update secret
aws secretsmanager update-secret \
  --secret-id my-db-password \
  --secret-string "NewPassword456"

# Delete secret
aws secretsmanager delete-secret \
  --secret-id my-db-password \
  --recovery-window-in-days 7
```

### SSM Parameter Store

```bash
# List parameters
aws ssm describe-parameters

# Put parameter (String)
aws ssm put-parameter \
  --name "/myapp/db-host" \
  --value "db.example.com" \
  --type String

# Put parameter (SecureString - encrypted)
aws ssm put-parameter \
  --name "/myapp/db-password" \
  --value "MyPassword123" \
  --type SecureString

# Get parameter
aws ssm get-parameter --name "/myapp/db-host"

# Get decrypted SecureString
aws ssm get-parameter \
  --name "/myapp/db-password" \
  --with-decryption

# Get multiple parameters
aws ssm get-parameters \
  --names "/myapp/db-host" "/myapp/db-password" \
  --with-decryption

# Delete parameter
aws ssm delete-parameter --name "/myapp/db-host"
```

---

## 👨‍💻 Author

**Manish Deore**

DevOps Engineer | AWS | Docker | Kubernetes | Terraform | Linux
---
⭐ **Star this repository if it helped you!**
