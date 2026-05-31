# AWS Deep-Dive Guide — For Platform Engineers (Azure Background)

> You know Azure well. This guide maps AWS concepts to your Azure knowledge
> and covers the AWS services REA Group uses: EKS, IAM, VPC, CloudWatch, Route 53, ECR, etc.

---

## TABLE OF CONTENTS

1. [AWS vs Azure — Quick Mapping](#1-mapping)
2. [IAM — Identity & Access Management (Deep)](#2-iam)
3. [VPC — Networking (Deep)](#3-vpc)
4. [EKS — Elastic Kubernetes Service](#4-eks)
5. [ECR — Container Registry](#5-ecr)
6. [CloudWatch — Monitoring & Logging](#6-cloudwatch)
7. [Route 53 — DNS](#7-route53)
8. [ALB / NLB — Load Balancers](#8-lb)
9. [S3 — Object Storage](#9-s3)
10. [Secrets Manager & Parameter Store](#10-secrets)
11. [Lambda — Serverless](#11-lambda)
12. [CloudFormation & CDK (IaC)](#12-iac)
13. [Security Services](#13-security)
14. [Cost Optimization](#14-cost)
15. [Interview Questions & Answers](#15-qa)

---

## 1. AWS vs AZURE — QUICK MAPPING <a name="1-mapping"></a>

| Category | Azure | AWS |
|---|---|---|
| **Kubernetes** | AKS | EKS |
| **Container Registry** | ACR | ECR |
| **VM** | Virtual Machine | EC2 |
| **Serverless** | Azure Functions | Lambda |
| **Object Storage** | Blob Storage | S3 |
| **DNS** | Azure DNS | Route 53 |
| **Load Balancer (L7)** | Application Gateway | ALB |
| **Load Balancer (L4)** | Azure Load Balancer | NLB |
| **CDN** | Azure CDN / Front Door | CloudFront |
| **VPN** | VPN Gateway | VPN Gateway / Transit Gateway |
| **Monitoring** | Azure Monitor | CloudWatch |
| **Logs** | Log Analytics (KQL) | CloudWatch Logs (Insights) |
| **Tracing** | Application Insights | X-Ray |
| **Secrets** | Key Vault | Secrets Manager / Parameter Store |
| **IAM** | Entra ID (AAD) + RBAC | IAM Users/Roles/Policies |
| **IaC** | ARM/Bicep | CloudFormation / CDK |
| **CI/CD** | Azure DevOps Pipelines | CodePipeline / CodeBuild |
| **SQL DB** | Azure SQL | RDS |
| **NoSQL** | Cosmos DB | DynamoDB |
| **Message Queue** | Service Bus | SQS |
| **Event Streaming** | Event Hubs | Kinesis / MSK |
| **WAF** | Azure WAF | AWS WAF |
| **Resource Groups** | Resource Groups (mandatory) | Resource Groups (optional, tags preferred) |
| **Regions** | Region / Availability Zone | Region / Availability Zone |
| **Managed Identity** | Managed Identity | IAM Roles for Services / IRSA for EKS |

---

## 2. IAM — IDENTITY & ACCESS MANAGEMENT <a name="2-iam"></a>

### Core Concepts

```
AWS Account
├── Root User (NEVER use for daily work)
├── IAM Users (people — avoid; use SSO instead)
├── IAM Groups (collections of users)
├── IAM Roles (assumed by services, not users)
│   ├── EC2 Instance Role
│   ├── EKS Pod Role (via IRSA)
│   ├── Lambda Execution Role
│   └── Cross-Account Role
└── IAM Policies (JSON documents defining permissions)
    ├── AWS Managed Policies (pre-built by AWS)
    ├── Customer Managed Policies (you create)
    └── Inline Policies (attached directly to user/role)
```

### IAM Policy Structure

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowS3ReadOnly",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::my-bucket",
                "arn:aws:s3:::my-bucket/*"
            ],
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": "10.0.0.0/8"
                }
            }
        }
    ]
}
```

### Key IAM Concepts

**Principal**: Who is making the request (user, role, service)
**Action**: What API call (`s3:GetObject`, `ec2:StartInstances`)
**Resource**: What AWS resource (ARN)
**Condition**: When does this apply (IP, time, MFA, tags)

**Policy Evaluation Logic:**
1. Default: DENY everything
2. Check all policies attached to the principal
3. If ANY policy has explicit DENY → DENIED
4. If ANY policy has ALLOW → ALLOWED
5. If no ALLOW found → DENIED (implicit deny)

### IRSA — IAM Roles for Service Accounts (EKS)

This is **Azure Managed Identity equivalent for EKS pods**.

```
Instead of: Pod → uses node's IAM role (too permissive)
With IRSA:  Pod → K8s ServiceAccount → IAM Role (least privilege)
```

```yaml
# 1. Create Kubernetes ServiceAccount with IAM role annotation
apiVersion: v1
kind: ServiceAccount
metadata:
  name: property-api-sa
  namespace: production
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/PropertyAPIRole
```

```bash
# 2. Create IAM Role with trust policy for the ServiceAccount
# Trust policy allows ONLY this specific ServiceAccount to assume the role
{
    "Version": "2012-10-17",
    "Statement": [{
        "Effect": "Allow",
        "Principal": {
            "Federated": "arn:aws:iam::123456789012:oidc-provider/oidc.eks.ap-southeast-2.amazonaws.com/id/ABCDEF"
        },
        "Action": "sts:AssumeRoleWithWebIdentity",
        "Condition": {
            "StringEquals": {
                "oidc.eks.ap-southeast-2.amazonaws.com/id/ABCDEF:sub": "system:serviceaccount:production:property-api-sa"
            }
        }
    }]
}
```

```yaml
# 3. Use ServiceAccount in Pod/Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: property-api
spec:
  template:
    spec:
      serviceAccountName: property-api-sa  # ← This gives the pod IAM permissions
      containers:
      - name: api
        image: 123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/property-api:v1
```

### AWS CLI IAM Commands

```bash
# List users
aws iam list-users

# List roles
aws iam list-roles | jq '.Roles[].RoleName'

# Get current identity
aws sts get-caller-identity

# Assume a role (get temporary credentials)
aws sts assume-role --role-arn arn:aws:iam::123456789012:role/AdminRole --role-session-name mysession

# List policies attached to a role
aws iam list-attached-role-policies --role-name MyRole

# Create a policy
aws iam create-policy --policy-name S3ReadOnly --policy-document file://policy.json

# Attach policy to role
aws iam attach-role-policy --role-name MyRole --policy-arn arn:aws:iam::123456789012:policy/S3ReadOnly
```

---

## 3. VPC — NETWORKING <a name="3-vpc"></a>

### VPC Architecture

```
┌─────────────────────── VPC (10.0.0.0/16) ──────────────────────┐
│                                                                  │
│  ┌─── AZ-a ──────────────────┐  ┌─── AZ-b ──────────────────┐ │
│  │                             │  │                             │ │
│  │  Public Subnet (10.0.1.0/24)│  │  Public Subnet (10.0.2.0/24)│ │
│  │  ┌─────┐  ┌─────┐          │  │  ┌─────┐  ┌─────┐          │ │
│  │  │ NAT │  │ ALB │          │  │  │ NAT │  │ ALB │          │ │
│  │  └─────┘  └─────┘          │  │  └─────┘  └─────┘          │ │
│  │                             │  │                             │ │
│  │  Private Subnet(10.0.3.0/24)│  │  Private Subnet(10.0.4.0/24)│ │
│  │  ┌──────────────────┐      │  │  ┌──────────────────┐      │ │
│  │  │ EKS Worker Nodes │      │  │  │ EKS Worker Nodes │      │ │
│  │  │ App Servers       │      │  │  │ App Servers       │      │ │
│  │  └──────────────────┘      │  │  └──────────────────┘      │ │
│  │                             │  │                             │ │
│  │  DB Subnet (10.0.5.0/24)   │  │  DB Subnet (10.0.6.0/24)   │ │
│  │  ┌──────────────────┐      │  │  ┌──────────────────┐      │ │
│  │  │ RDS / ElastiCache │      │  │  │ RDS Standby       │      │ │
│  │  └──────────────────┘      │  │  └──────────────────┘      │ │
│  └─────────────────────────────┘  └─────────────────────────────┘ │
│                                                                  │
│  Internet Gateway ──── Route Table ──── NAT Gateway              │
└──────────────────────────────────────────────────────────────────┘
```

### Key Networking Concepts

| Concept | Azure Equivalent | Purpose |
|---|---|---|
| VPC | VNet | Isolated network |
| Subnet | Subnet | Network segment |
| Internet Gateway | Built-in | Connect to internet |
| NAT Gateway | NAT Gateway | Outbound internet for private subnets |
| Security Group | NSG | Stateful firewall (per instance) |
| NACL | NSG (subnet-level) | Stateless firewall (per subnet) |
| Route Table | Route Table | Routing rules |
| VPC Peering | VNet Peering | Connect two VPCs |
| Transit Gateway | Virtual WAN | Hub-and-spoke multi-VPC |
| PrivateLink / VPC Endpoint | Private Endpoint | Access AWS services privately |

### Security Groups vs NACLs

| Feature | Security Group | NACL |
|---|---|---|
| Level | Instance (ENI) | Subnet |
| Stateful | Yes (return traffic auto-allowed) | No (must allow both inbound & outbound) |
| Rules | Allow only | Allow AND Deny |
| Evaluation | All rules evaluated | Rules evaluated in order (lowest number first) |
| Default | Deny all inbound, allow all outbound | Allow all |

```bash
# Security Group example
aws ec2 create-security-group --group-name web-sg --description "Web Server SG" --vpc-id vpc-abc123

# Allow HTTP inbound
aws ec2 authorize-security-group-ingress --group-id sg-123 --protocol tcp --port 80 --cidr 0.0.0.0/0

# Allow HTTPS inbound
aws ec2 authorize-security-group-ingress --group-id sg-123 --protocol tcp --port 443 --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress --group-id sg-123 --protocol tcp --port 22 --cidr 10.0.0.0/8
```

---

## 4. EKS — ELASTIC KUBERNETES SERVICE <a name="4-eks"></a>

### EKS vs AKS

| Feature | AKS (Azure) | EKS (AWS) |
|---|---|---|
| Control Plane Cost | Free | ~$73/month per cluster |
| Networking | Azure CNI / Kubenet | VPC CNI (native VPC IPs) |
| Ingress | AGIC / Nginx | AWS ALB Ingress Controller |
| Identity for Pods | Managed Identity / Workload Identity | IRSA (IAM Roles for Service Accounts) |
| Node Auto-scaling | AKS Cluster Autoscaler | Karpenter / Cluster Autoscaler |
| Managed Add-ons | AKS Add-ons | EKS Add-ons |
| Container Registry | ACR | ECR |
| Logging | Azure Monitor / Container Insights | CloudWatch Container Insights / Fluent Bit |
| Service Mesh | OSM / Istio | AWS App Mesh / Istio |

### EKS Architecture

```
┌─────────────── AWS Region ──────────────────┐
│                                               │
│  ┌──── EKS Control Plane (AWS Managed) ────┐ │
│  │  API Server                               │ │
│  │  etcd                                     │ │
│  │  Controller Manager                       │ │
│  │  Scheduler                                │ │
│  └───────────────────────────────────────────┘ │
│                    │                           │
│  ┌──── VPC ────────┼────────────────────────┐ │
│  │                 │                         │ │
│  │  ┌─ AZ-a ──────┼─┐  ┌─ AZ-b ─────────┐ │ │
│  │  │ Worker Node 1 │  │ Worker Node 2    │ │ │
│  │  │ ┌───────────┐ │  │ ┌───────────┐   │ │ │
│  │  │ │ Pod: App  │ │  │ │ Pod: App  │   │ │ │
│  │  │ │ Pod: API  │ │  │ │ Pod: API  │   │ │ │
│  │  │ └───────────┘ │  │ └───────────┘   │ │ │
│  │  └────────────────┘  └────────────────┘ │ │
│  └──────────────────────────────────────────┘ │
└───────────────────────────────────────────────┘
```

### EKS CLI Commands

```bash
# Create cluster
eksctl create cluster \
    --name rea-platform \
    --region ap-southeast-2 \
    --version 1.29 \
    --nodegroup-name workers \
    --node-type t3.medium \
    --nodes 3 \
    --nodes-min 2 \
    --nodes-max 5 \
    --managed

# Get kubeconfig
aws eks update-kubeconfig --name rea-platform --region ap-southeast-2

# List clusters
aws eks list-clusters --region ap-southeast-2

# Describe cluster
aws eks describe-cluster --name rea-platform --region ap-southeast-2

# List nodegroups
aws eks list-nodegroups --cluster-name rea-platform

# Update nodegroup (scale)
aws eks update-nodegroup-config \
    --cluster-name rea-platform \
    --nodegroup-name workers \
    --scaling-config minSize=3,maxSize=10,desiredSize=5
```

### EKS Add-ons

```bash
# List available add-ons
aws eks describe-addon-versions --kubernetes-version 1.29 | jq '.addons[].addonName'

# Core add-ons:
# vpc-cni         → VPC networking for pods
# coredns          → DNS for services
# kube-proxy       → Network proxy
# aws-ebs-csi-driver → EBS volumes for PVCs

# Install add-on
aws eks create-addon --cluster-name rea-platform --addon-name vpc-cni --addon-version v1.16.0

# AWS Load Balancer Controller (for ALB Ingress)
# Installed via Helm:
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=rea-platform \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller
```

### EKS + ALB Ingress

```yaml
# ALB Ingress (AWS Load Balancer Controller)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: property-api-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-southeast-2:123456789012:certificate/abc-123
    alb.ingress.kubernetes.io/healthcheck-path: /healthz
spec:
  rules:
  - host: api.rea-property.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: property-api
            port:
              number: 80
```

---

## 5. ECR — ELASTIC CONTAINER REGISTRY <a name="5-ecr"></a>

```bash
# Create repository
aws ecr create-repository --repository-name property-api --region ap-southeast-2

# Login to ECR (like docker login to ACR)
aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-southeast-2.amazonaws.com

# Tag and push image
docker build -t property-api:v1 .
docker tag property-api:v1 123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/property-api:v1
docker push 123456789012.dkr.ecr.ap-southeast-2.amazonaws.com/property-api:v1

# List images
aws ecr list-images --repository-name property-api

# Lifecycle policy (auto-clean old images — like ACR tasks)
aws ecr put-lifecycle-policy --repository-name property-api --lifecycle-policy-text '{
  "rules": [{
    "rulePriority": 1,
    "description": "Keep last 10 images",
    "selection": {
      "tagStatus": "any",
      "countType": "imageCountMoreThan",
      "countNumber": 10
    },
    "action": {
      "type": "expire"
    }
  }]
}'

# Image scanning
aws ecr start-image-scan --repository-name property-api --image-id imageTag=v1
aws ecr describe-image-scan-findings --repository-name property-api --image-id imageTag=v1
```

---

## 6. CLOUDWATCH — MONITORING & LOGGING <a name="6-cloudwatch"></a>

### CloudWatch vs Azure Monitor

| Azure | AWS CloudWatch |
|---|---|
| Azure Monitor Metrics | CloudWatch Metrics |
| Log Analytics (KQL) | CloudWatch Logs Insights |
| Alerts | CloudWatch Alarms |
| Application Insights | X-Ray + CloudWatch |
| Dashboards | CloudWatch Dashboards |
| Container Insights (AKS) | Container Insights (EKS) |

### CloudWatch Metrics

```bash
# List available metrics for a service
aws cloudwatch list-metrics --namespace AWS/EC2

# Get CPU utilization for an EC2 instance
aws cloudwatch get-metric-statistics \
    --namespace AWS/EC2 \
    --metric-name CPUUtilization \
    --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
    --start-time 2024-01-01T00:00:00 \
    --end-time 2024-01-01T01:00:00 \
    --period 300 \
    --statistics Average

# Custom metrics
aws cloudwatch put-metric-data \
    --namespace "REA/PropertyAPI" \
    --metric-name "SearchLatency" \
    --value 45.2 \
    --unit Milliseconds \
    --dimensions Service=property-search,Environment=production
```

### CloudWatch Logs

```bash
# Create log group
aws logs create-log-group --log-group-name /rea/property-api

# View log streams
aws logs describe-log-streams --log-group-name /rea/property-api --order-by LastEventTime --descending

# Get log events
aws logs get-log-events --log-group-name /rea/property-api --log-stream-name stream1 --limit 50

# Tail logs in real-time
aws logs tail /rea/property-api --follow
```

### CloudWatch Logs Insights (Query Language)

```sql
-- Find errors in the last 1 hour
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 50

-- Count errors by type
fields @message
| filter @message like /ERROR/
| stats count(*) by @message
| sort count desc
| limit 10

-- Latency percentiles
filter @type = "API"
| stats avg(latency) as avg_lat,
        pct(latency, 50) as p50,
        pct(latency, 95) as p95,
        pct(latency, 99) as p99
  by bin(5m)

-- Top 10 slowest requests
fields @timestamp, @message, latency, path
| filter latency > 1000
| sort latency desc
| limit 10

-- 5xx error rate over time
filter status >= 500
| stats count(*) as errors by bin(5m)

-- Container Insights — pod CPU usage
stats avg(pod_cpu_utilization) as avg_cpu by PodName
| sort avg_cpu desc
| limit 20
```

### CloudWatch Alarms

```bash
# Create alarm — CPU > 80%
aws cloudwatch put-metric-alarm \
    --alarm-name "HighCPU-PropertyAPI" \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --statistic Average \
    --period 300 \
    --threshold 80 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 2 \
    --alarm-actions arn:aws:sns:ap-southeast-2:123456789012:ops-alerts \
    --dimensions Name=InstanceId,Value=i-1234567890abcdef0

# List alarms
aws cloudwatch describe-alarms --state-value ALARM
```

---

## 7. ROUTE 53 — DNS <a name="7-route53"></a>

```bash
# List hosted zones
aws route53 list-hosted-zones

# List records in a zone
aws route53 list-resource-record-sets --hosted-zone-id Z1234567890

# Create/Update record (UPSERT)
aws route53 change-resource-record-sets --hosted-zone-id Z1234567890 --change-batch '{
  "Changes": [{
    "Action": "UPSERT",
    "ResourceRecordSet": {
      "Name": "api.rea-property.com",
      "Type": "A",
      "AliasTarget": {
        "HostedZoneId": "Z1234567890",
        "DNSName": "alb-123.ap-southeast-2.elb.amazonaws.com",
        "EvaluateTargetHealth": true
      }
    }
  }]
}'
```

### Route 53 Routing Policies

| Policy | Use Case |
|---|---|
| Simple | Single resource (default) |
| Weighted | A/B testing (80% v1, 20% v2) |
| Latency | Route to lowest-latency region |
| Failover | Primary/secondary (DR) |
| Geolocation | Route based on user location |
| Multi-value | Multiple healthy IPs (basic LB) |

### Health Checks

```bash
aws route53 create-health-check --caller-reference "property-api-$(date +%s)" --health-check-config '{
    "IPAddress": "1.2.3.4",
    "Port": 443,
    "Type": "HTTPS",
    "ResourcePath": "/healthz",
    "RequestInterval": 30,
    "FailureThreshold": 3
}'
```

---

## 8. ALB / NLB — LOAD BALANCERS <a name="8-lb"></a>

| Feature | ALB (Application LB) | NLB (Network LB) |
|---|---|---|
| Layer | 7 (HTTP/HTTPS) | 4 (TCP/UDP) |
| Azure Equiv | Application Gateway | Azure Load Balancer |
| Routing | Path-based, host-based, headers | Port-based |
| SSL Termination | Yes | Yes (TLS) |
| WebSocket | Yes | Yes |
| Static IP | No (use Global Accelerator) | Yes |
| Performance | Good | Ultra-low latency |
| Use Case | Web apps, APIs, microservices | Databases, game servers, IoT |
| K8s Integration | AWS LB Controller Ingress | Service type: LoadBalancer |

---

## 9. S3 — OBJECT STORAGE <a name="9-s3"></a>

```bash
# Create bucket
aws s3 mb s3://rea-property-assets --region ap-southeast-2

# Upload file
aws s3 cp image.jpg s3://rea-property-assets/images/

# List objects
aws s3 ls s3://rea-property-assets/images/

# Download
aws s3 cp s3://rea-property-assets/images/image.jpg ./

# Sync directory
aws s3 sync ./build/ s3://rea-property-frontend/ --delete

# Presigned URL (temporary access to private object)
aws s3 presign s3://rea-property-assets/images/private.jpg --expires-in 3600
```

### S3 Bucket Policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadForWebsite",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::rea-property-frontend/*"
        }
    ]
}
```

### Storage Classes

| Class | Use Case | Cost |
|---|---|---|
| S3 Standard | Frequently accessed | Highest |
| S3 Intelligent-Tiering | Unknown access patterns | Auto-moves |
| S3 Standard-IA | Infrequent access (>30 days) | Lower |
| S3 Glacier Instant Retrieval | Archive with instant access | Much lower |
| S3 Glacier Flexible Retrieval | Archive (minutes to hours) | Very low |
| S3 Glacier Deep Archive | Long-term archive (12+ hours) | Lowest |

---

## 10. SECRETS MANAGER & PARAMETER STORE <a name="10-secrets"></a>

### Secrets Manager vs Parameter Store

| Feature | Secrets Manager | Systems Manager Parameter Store |
|---|---|---|
| Azure Equiv | Key Vault Secrets | Key Vault (sort of) |
| Cost | $0.40/secret/month | Free (standard), $0.05/advanced |
| Rotation | Built-in auto-rotation | Manual |
| Cross-account | Yes | Limited |
| Best for | DB passwords, API keys | Config values, feature flags |

```bash
# Secrets Manager
aws secretsmanager create-secret --name rea/property-api/db-password --secret-string "MyS3cureP@ss"
aws secretsmanager get-secret-value --secret-id rea/property-api/db-password

# Parameter Store
aws ssm put-parameter --name "/rea/production/db-host" --type String --value "db.rea.internal"
aws ssm put-parameter --name "/rea/production/api-key" --type SecureString --value "abc123"
aws ssm get-parameter --name "/rea/production/db-host" --with-decryption
aws ssm get-parameters-by-path --path "/rea/production/"
```

### In Kubernetes (External Secrets Operator)

```yaml
# ExternalSecret — syncs AWS secrets into K8s Secrets
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: property-api-secrets
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: property-api-secrets
  data:
  - secretKey: DB_PASSWORD
    remoteRef:
      key: rea/property-api/db-password
```

---

## 11. LAMBDA — SERVERLESS <a name="11-lambda"></a>

```bash
# Create function
aws lambda create-function \
    --function-name property-image-resize \
    --runtime python3.12 \
    --handler lambda_function.lambda_handler \
    --role arn:aws:iam::123456789012:role/LambdaExecutionRole \
    --zip-file fileb://function.zip

# Invoke
aws lambda invoke --function-name property-image-resize --payload '{"key": "image.jpg"}' output.json

# View logs
aws logs tail /aws/lambda/property-image-resize --follow
```

```python
# lambda_function.py — simple example
import json
import boto3

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    # Process event
    return {
        'statusCode': 200,
        'body': json.dumps({'message': 'Processed successfully'})
    }
```

### Lambda Triggers
- API Gateway (HTTP endpoints)
- S3 (file uploads)
- SQS (message queue)
- CloudWatch Events / EventBridge (scheduled/event-driven)
- DynamoDB Streams (database changes)
- SNS (notifications)

---

## 12. CLOUDFORMATION & CDK <a name="12-iac"></a>

### CloudFormation (like ARM templates)

```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: REA Property API Infrastructure

Parameters:
  Environment:
    Type: String
    Default: production
    AllowedValues: [production, staging, development]

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: !Sub "${Environment}-vpc"

  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Web Server Security Group
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0

Outputs:
  VPCId:
    Value: !Ref VPC
    Export:
      Name: !Sub "${Environment}-VPCId"
```

```bash
# Deploy
aws cloudformation deploy \
    --template-file template.yaml \
    --stack-name rea-platform-prod \
    --parameter-overrides Environment=production \
    --capabilities CAPABILITY_IAM

# List stacks
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE

# Delete stack
aws cloudformation delete-stack --stack-name rea-platform-prod
```

### Terraform (you already know this)

REA likely uses Terraform. Your Azure Terraform experience transfers directly.
Key AWS provider difference:

```hcl
provider "aws" {
  region = "ap-southeast-2"  # Sydney (REA is Australian)
}

resource "aws_eks_cluster" "main" {
  name     = "rea-platform"
  role_arn = aws_iam_role.eks.arn
  version  = "1.29"

  vpc_config {
    subnet_ids = var.subnet_ids
  }
}
```

---

## 13. SECURITY SERVICES <a name="13-security"></a>

| Service | Purpose | Azure Equivalent |
|---|---|---|
| GuardDuty | Threat detection (AI-based) | Defender for Cloud |
| Security Hub | Security posture dashboard | Security Center |
| Inspector | Vulnerability scanning (EC2/ECR) | Defender for Servers |
| WAF | Web Application Firewall | Azure WAF |
| Shield | DDoS protection | Azure DDoS Protection |
| CloudTrail | API audit log (who did what) | Activity Log |
| Config | Resource configuration audit | Azure Policy |
| KMS | Key management | Key Vault Keys |

```bash
# CloudTrail — find who deleted a resource
aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=EventName,AttributeValue=DeleteBucket \
    --max-results 10

# GuardDuty — list findings
aws guardduty list-findings --detector-id abc123
```

---

## 14. COST OPTIMIZATION <a name="14-cost"></a>

```bash
# Cost Explorer — monthly costs by service
aws ce get-cost-and-usage \
    --time-period Start=2024-01-01,End=2024-02-01 \
    --granularity MONTHLY \
    --metrics "BlendedCost" \
    --group-by Type=DIMENSION,Key=SERVICE

# Reserved Instances — check utilization
aws ce get-reservation-utilization \
    --time-period Start=2024-01-01,End=2024-02-01

# Savings Plans recommendations
aws ce get-savings-plans-purchase-recommendation \
    --savings-plans-type COMPUTE_SP \
    --term-in-years ONE_YEAR \
    --payment-option NO_UPFRONT \
    --lookback-period-in-days THIRTY_DAYS
```

### Cost Saving Strategies (mention in interview)

| Strategy | Savings |
|---|---|
| Reserved Instances / Savings Plans | 30-60% |
| Spot Instances (for non-critical workloads) | Up to 90% |
| Right-sizing (CloudWatch metrics → smaller instances) | 20-40% |
| S3 Lifecycle policies (move to Glacier) | 50-90% |
| EKS with Karpenter (auto right-size nodes) | 20-30% |
| Delete unused EBS volumes, old snapshots | Variable |
| NAT Gateway optimization (VPC endpoints instead) | Significant |

---

## 15. INTERVIEW QUESTIONS & ANSWERS <a name="15-qa"></a>

### Q1: How does networking work in EKS?

**Answer**: "EKS uses the Amazon VPC CNI plugin, which gives each pod a real IP address from the VPC subnet — unlike AKS kubenet which uses an overlay network. This means:
- Pods can communicate directly with other AWS services without NAT
- Security Groups can be applied at the pod level
- But you need enough IP addresses in your subnets (we use /19 or larger for large clusters)
- For IP address conservation, you can enable prefix delegation or use custom networking to assign pods IPs from a different subnet."

### Q2: How do you give an EKS pod access to AWS services securely?

**Answer**: "We use IRSA — IAM Roles for Service Accounts. The flow is:
1. Create an IAM role with the specific permissions needed (least privilege)
2. Create a trust policy that allows only a specific Kubernetes ServiceAccount to assume the role
3. Annotate the ServiceAccount with the IAM role ARN
4. Pods using that ServiceAccount automatically get temporary AWS credentials via the OIDC provider
This is like Azure Workload Identity for AKS — each pod gets its own identity instead of using the node's permissions."

### Q3: How would you set up a multi-region DR strategy on AWS?

**Answer**: "For a platform like REA's property search:
1. **Data layer**: RDS with cross-region read replicas (auto-promoted on failover), S3 cross-region replication
2. **Compute**: EKS clusters in both regions, same GitOps config (ArgoCD)
3. **DNS**: Route 53 with failover routing policy — health checks detect primary failure, auto-routes to secondary
4. **CDN**: CloudFront serves static content from edge, shields origin
5. **RPO/RTO targets**: Depends on the service tier — critical services <5 min RPO, <15 min RTO
6. **Testing**: Regular DR drills, chaos engineering to verify failover works"

### Q4: How is AWS IAM different from Azure RBAC?

**Answer**: "Key differences:
- Azure: Entra ID (AAD) is the identity provider, RBAC roles are assigned at subscription/RG/resource scope
- AWS: IAM is per-account, policies are JSON documents attached to users/roles
- Azure has built-in roles (Contributor, Reader). AWS has managed policies but most teams write custom policies
- AWS doesn't have a concept like Azure resource groups for access control — they use tags and resource-level ARNs in policies
- For K8s: Azure uses AAD integration for AKS RBAC. AWS uses IRSA + aws-auth ConfigMap for EKS RBAC
- Cross-account in AWS uses STS AssumeRole; Azure uses lighthouse or cross-tenant access"

### Q5: How do you troubleshoot a pod that can't reach an AWS service (like S3)?

**Answer**: "Systematic approach:
1. **IRSA configured?** Check ServiceAccount annotation, check if pod has `AWS_ROLE_ARN` and `AWS_WEB_IDENTITY_TOKEN_FILE` env vars
2. **IAM role trust policy?** The OIDC provider and ServiceAccount must match exactly
3. **IAM permissions?** Check the role's policy allows the specific S3 action on the specific bucket ARN
4. **VPC networking?** Is the pod in a private subnet? Does it have a NAT Gateway for internet access? Or better, use a VPC endpoint for S3 (stays within AWS network)
5. **Security Groups?** Check if egress is allowed on port 443 (S3 uses HTTPS)
6. **DNS?** Can the pod resolve `s3.ap-southeast-2.amazonaws.com`? Check CoreDNS
I'd use `kubectl exec` into the pod and run `aws sts get-caller-identity` to verify the assumed role, then `aws s3 ls s3://bucket/` to test access."
