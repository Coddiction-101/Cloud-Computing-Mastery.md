# ☁️ Cloud Computing — Hands-On Projects Repository
### *10 Real-World Projects to Build Your Cloud Portfolio from Zero to Advanced*

> **"You don't learn cloud computing by reading about it. You learn it by breaking things, fixing them, and shipping them."**
> This document contains 10 progressively complex, real-world projects. Each project is self-contained with full architecture, step-by-step instructions, what you will learn, what can go wrong, and how to extend it further. Build all 10 and you will have a portfolio that demonstrates genuine cloud engineering capability.

---

## 📌 How to Use This Repository

Each project builds on the previous one in terms of complexity and concepts. Do them in order if you are a beginner. If you have some cloud experience, jump to the level that matches your current skill.

**Difficulty Scale:**
- 🟢 **Beginner** — No prior cloud experience needed
- 🟡 **Intermediate** — Comfortable with basic cloud services
- 🔴 **Advanced** — Confident with networking, IaC, and containers

**Cost Estimate:** Every project is designed to run within the AWS Free Tier or cost less than $5 if you clean up resources promptly. Cost estimates are provided for each project.

**Prerequisites for All Projects:**
- AWS Free Tier account (credit card required but won't be charged if within limits)
- AWS CLI installed and configured (`aws configure`)
- Git installed
- Basic terminal comfort (Linux/Mac or WSL on Windows)

---

## Project Index

| # | Project | Difficulty | Core Services | Est. Cost |
|---|---------|-----------|---------------|-----------|
| 1 | [Static Website with Custom Domain](#project-1-static-website-with-custom-domain) | 🟢 Beginner | S3, CloudFront, Route 53 | Free |
| 2 | [Serverless Contact Form API](#project-2-serverless-contact-form-api) | 🟢 Beginner | Lambda, API Gateway, SES | Free |
| 3 | [Automated EC2 Backup System](#project-3-automated-ec2-backup-system) | 🟢 Beginner | EC2, Lambda, CloudWatch, SNS | Free |
| 4 | [Three-Tier Web Application](#project-4-three-tier-web-application) | 🟡 Intermediate | VPC, EC2, RDS, ALB, Auto Scaling | ~$2 |
| 5 | [Serverless Image Processing Pipeline](#project-5-serverless-image-processing-pipeline) | 🟡 Intermediate | S3, Lambda, DynamoDB, SNS | Free |
| 6 | [Infrastructure as Code with Terraform](#project-6-infrastructure-as-code-with-terraform) | 🟡 Intermediate | Terraform, EC2, VPC, RDS | ~$1 |
| 7 | [CI/CD Pipeline for a Web Application](#project-7-cicd-pipeline-for-a-web-application) | 🟡 Intermediate | GitHub Actions, ECR, ECS Fargate | ~$2 |
| 8 | [Kubernetes Microservices Deployment](#project-8-kubernetes-microservices-deployment) | 🔴 Advanced | EKS, ECR, ALB, IAM | ~$3 |
| 9 | [Real-Time Data Pipeline](#project-9-real-time-data-pipeline) | 🔴 Advanced | Kinesis, Lambda, DynamoDB, S3, Athena | ~$2 |
| 10 | [Multi-Region Disaster Recovery Architecture](#project-10-multi-region-disaster-recovery-architecture) | 🔴 Advanced | Route 53, RDS, S3, Lambda, CloudFormation | ~$3 |

---

## Project 1: Static Website with Custom Domain

**Difficulty:** 🟢 Beginner
**Estimated Time:** 2–3 hours
**AWS Free Tier Cost:** Free (within free tier limits)
**Core Services:** S3, CloudFront, ACM (Certificate Manager), Route 53

---

### What You Are Building

A production-grade static website hosted entirely on AWS — with HTTPS, a custom domain, and global CDN delivery. This is how millions of static websites, documentation sites, and landing pages are deployed professionally.

```
                    ┌─────────────────────────────────────┐
                    │           User's Browser             │
                    └──────────────┬──────────────────────┘
                                   │ HTTPS Request
                                   ▼
                    ┌─────────────────────────────────────┐
                    │         Route 53 (DNS)               │
                    │   yourdomain.com → CloudFront        │
                    └──────────────┬──────────────────────┘
                                   │
                                   ▼
                    ┌─────────────────────────────────────┐
                    │    CloudFront Distribution           │
                    │  (Global CDN — 400+ Edge Locations)  │
                    │  SSL/TLS Certificate (ACM)           │
                    └──────────────┬──────────────────────┘
                                   │ Cache Miss → Origin Request
                                   ▼
                    ┌─────────────────────────────────────┐
                    │         S3 Bucket (Origin)           │
                    │  yourdomain.com.s3.amazonaws.com     │
                    │  index.html, style.css, app.js       │
                    └─────────────────────────────────────┘
```

---

### What You Will Learn

By completing this project you will understand how static site hosting works at a production level, how CDNs cache and serve content globally, how SSL/TLS certificates are provisioned and automatically renewed in AWS, how DNS records connect a domain name to cloud resources, how S3 bucket policies control public access, the difference between S3 website hosting and CloudFront as an origin, and how to invalidate CDN cache when you deploy updates.

---

### Step-by-Step Build

**Step 1: Prepare Your Website Files**

Create a simple but complete website locally. This is your deployable artifact.

```bash
mkdir my-cloud-website && cd my-cloud-website

# Create index.html
cat > index.html << 'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Cloud Website</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>Hello from the Cloud ☁️</h1>
    <p>Hosted on S3 + CloudFront + HTTPS</p>
    <script src="app.js"></script>
</body>
</html>
EOF

cat > style.css << 'EOF'
body { font-family: Arial, sans-serif; text-align: center; padding: 50px; background: #f0f4f8; }
h1 { color: #232f3e; }
EOF

cat > app.js << 'EOF'
console.log('Deployed on AWS CloudFront + S3');
EOF
```

**Step 2: Create and Configure S3 Bucket**

The bucket name must exactly match your domain name for CloudFront to work cleanly.

```bash
# Replace yourdomain.com with your actual domain
BUCKET_NAME="yourdomain.com"
REGION="us-east-1"

# Create the bucket
aws s3 mb s3://$BUCKET_NAME --region $REGION

# Enable static website hosting
aws s3 website s3://$BUCKET_NAME \
  --index-document index.html \
  --error-document error.html

# Upload your website files
aws s3 sync . s3://$BUCKET_NAME --exclude "*.sh"
```

**Step 3: Set S3 Bucket Policy for CloudFront Access**

Rather than making the bucket fully public (a security risk), you will allow access only from your CloudFront distribution using an Origin Access Control (OAC).

Go to the AWS Console → S3 → your bucket → Permissions → Block Public Access → ensure public access is BLOCKED (this is the secure configuration). You will configure CloudFront to access the bucket privately in the next step.

**Step 4: Create CloudFront Distribution**

```bash
# First, request an SSL certificate in ACM (must be in us-east-1 for CloudFront)
aws acm request-certificate \
  --domain-name yourdomain.com \
  --subject-alternative-names "www.yourdomain.com" \
  --validation-method DNS \
  --region us-east-1
```

Note the Certificate ARN from the output. You must validate it by adding the DNS CNAME records that ACM provides to your domain's DNS. This proves you own the domain.

Now create the CloudFront distribution via the console:
- Go to CloudFront → Create Distribution
- Origin Domain: select your S3 bucket
- Origin Access: Origin Access Control Settings (recommended) → Create new OAC
- Viewer Protocol Policy: Redirect HTTP to HTTPS
- Alternate Domain Names (CNAMEs): yourdomain.com, www.yourdomain.com
- SSL Certificate: select the ACM certificate you just created
- Default Root Object: index.html
- Click Create Distribution

Copy the bucket policy that CloudFront generates and apply it to your S3 bucket.

**Step 5: Configure DNS in Route 53**

```bash
# If your domain is registered elsewhere, you can still use Route 53 as the DNS service
# Create a hosted zone
aws route53 create-hosted-zone \
  --name yourdomain.com \
  --caller-reference $(date +%s)
```

In Route 53, create two A records (alias records):
- yourdomain.com → Alias to your CloudFront distribution
- www.yourdomain.com → Alias to your CloudFront distribution

Update your domain registrar's nameservers to the four Route 53 nameservers listed in your hosted zone.

**Step 6: Deploy Updates and Invalidate Cache**

```bash
# After modifying files, sync to S3 and invalidate CloudFront cache
aws s3 sync . s3://$BUCKET_NAME

# Invalidate CloudFront cache (replace DISTRIBUTION_ID with yours)
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/*"
```

---

### What Can Go Wrong and How to Fix It

**Certificate not validating:** You must add the CNAME records ACM provides to your DNS. If using Route 53, ACM can do this automatically. If using another DNS provider, copy the CNAME records manually. Validation can take up to 30 minutes.

**403 Forbidden from CloudFront:** The S3 bucket policy does not allow CloudFront to access it. Double-check that you applied the OAC bucket policy generated by CloudFront to your S3 bucket.

**Changes not reflecting:** CloudFront caches content. Always run a cache invalidation after deploying updates. During development, use a query string parameter to bypass cache.

**www subdomain not resolving:** Ensure you created both the root domain record and the www subdomain record in Route 53.

---

### How to Extend This Project

Add a contact form using Project 2's Lambda backend. Integrate with GitHub Actions to automatically deploy when you push to main branch. Add CloudFront functions to handle URL redirects and rewrites. Enable CloudFront access logging to S3 and analyze traffic patterns. Set up a staging environment pointing to a separate S3 bucket.

---

## Project 2: Serverless Contact Form API

**Difficulty:** 🟢 Beginner
**Estimated Time:** 2–3 hours
**AWS Free Tier Cost:** Free (Lambda free tier: 1M requests/month, SES: 62,000 emails/month from EC2)
**Core Services:** Lambda, API Gateway, SES (Simple Email Service), IAM

---

### What You Are Building

A fully serverless backend API that accepts form submissions from a website and sends email notifications — with zero servers to manage, zero maintenance, and effectively zero cost at small scale.

```
┌──────────────┐     HTTPS POST      ┌─────────────────┐
│   Website    │ ──────────────────► │   API Gateway    │
│ Contact Form │                     │  (REST API)      │
└──────────────┘                     └────────┬────────┘
                                              │ Invoke
                                              ▼
                                     ┌─────────────────┐
                                     │  Lambda Function │
                                     │  (Python 3.12)   │
                                     │                  │
                                     │ 1. Validate input│
                                     │ 2. Send via SES  │
                                     │ 3. Return result │
                                     └────────┬────────┘
                                              │ SendEmail API
                                              ▼
                                     ┌─────────────────┐
                                     │   Amazon SES     │
                                     │ (Email Delivery) │
                                     └─────────────────┘
                                              │
                                              ▼
                                     ┌─────────────────┐
                                     │  Your Inbox 📧   │
                                     └─────────────────┘
```

---

### What You Will Learn

How Lambda functions work — the execution model, cold starts, and the event/context objects. How API Gateway creates HTTP endpoints and passes requests to Lambda. How IAM roles give Lambda permissions to call other AWS services. How SES sends emails programmatically. How to handle CORS so your frontend website can call the API. Input validation in serverless functions. How to test Lambda functions locally and in the console.

---

### Step-by-Step Build

**Step 1: Verify Your Email in SES**

SES starts in sandbox mode — you can only send emails to verified addresses until you request production access.

```bash
# Verify your email address
aws ses verify-email-identity \
  --email-address your-email@gmail.com \
  --region us-east-1

# Check verification status
aws ses get-identity-verification-attributes \
  --identities your-email@gmail.com \
  --region us-east-1
```

Check your inbox and click the verification link.

**Step 2: Create the Lambda Function**

Create a file called `lambda_function.py`:

```python
import json
import boto3
import re
from datetime import datetime

ses_client = boto3.client('ses', region_name='us-east-1')

RECIPIENT_EMAIL = "your-email@gmail.com"
SENDER_EMAIL = "your-email@gmail.com"

def validate_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None

def lambda_handler(event, context):
    # Handle preflight CORS request
    if event.get('httpMethod') == 'OPTIONS':
        return {
            'statusCode': 200,
            'headers': cors_headers(),
            'body': ''
        }
    
    try:
        body = json.loads(event.get('body', '{}'))
        
        name = body.get('name', '').strip()
        email = body.get('email', '').strip()
        message = body.get('message', '').strip()
        
        # Validate inputs
        if not name or len(name) < 2:
            return error_response(400, "Name must be at least 2 characters.")
        
        if not email or not validate_email(email):
            return error_response(400, "Invalid email address.")
        
        if not message or len(message) < 10:
            return error_response(400, "Message must be at least 10 characters.")
        
        if len(message) > 2000:
            return error_response(400, "Message must be under 2000 characters.")
        
        # Send email via SES
        timestamp = datetime.utcnow().strftime('%Y-%m-%d %H:%M:%S UTC')
        
        ses_client.send_email(
            Source=SENDER_EMAIL,
            Destination={'ToAddresses': [RECIPIENT_EMAIL]},
            Message={
                'Subject': {
                    'Data': f'New Contact Form Submission from {name}'
                },
                'Body': {
                    'Html': {
                        'Data': f"""
                        <h2>New Contact Form Submission</h2>
                        <p><strong>Name:</strong> {name}</p>
                        <p><strong>Email:</strong> {email}</p>
                        <p><strong>Time:</strong> {timestamp}</p>
                        <hr>
                        <p><strong>Message:</strong></p>
                        <p>{message.replace(chr(10), '<br>')}</p>
                        """
                    }
                }
            }
        )
        
        return {
            'statusCode': 200,
            'headers': cors_headers(),
            'body': json.dumps({
                'success': True,
                'message': 'Your message has been sent successfully!'
            })
        }
        
    except Exception as e:
        print(f"Error: {str(e)}")
        return error_response(500, "Internal server error. Please try again.")

def cors_headers():
    return {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type',
        'Access-Control-Allow-Methods': 'POST, OPTIONS',
        'Content-Type': 'application/json'
    }

def error_response(status_code, message):
    return {
        'statusCode': status_code,
        'headers': cors_headers(),
        'body': json.dumps({'success': False, 'message': message})
    }
```

**Step 3: Create IAM Role for Lambda**

```bash
# Create trust policy
cat > trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "lambda.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }]
}
EOF

# Create the role
aws iam create-role \
  --role-name ContactFormLambdaRole \
  --assume-role-policy-document file://trust-policy.json

# Attach CloudWatch Logs permission (for Lambda logging)
aws iam attach-role-policy \
  --role-name ContactFormLambdaRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

# Create and attach SES send permission
cat > ses-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["ses:SendEmail", "ses:SendRawEmail"],
    "Resource": "*"
  }]
}
EOF

aws iam put-role-policy \
  --role-name ContactFormLambdaRole \
  --policy-name SESSendEmailPolicy \
  --policy-document file://ses-policy.json
```

**Step 4: Deploy the Lambda Function**

```bash
# Package the function
zip function.zip lambda_function.py

# Get your account ID
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

# Create the Lambda function
aws lambda create-function \
  --function-name ContactFormHandler \
  --runtime python3.12 \
  --handler lambda_function.lambda_handler \
  --role arn:aws:iam::$ACCOUNT_ID:role/ContactFormLambdaRole \
  --zip-file fileb://function.zip \
  --timeout 30 \
  --region us-east-1
```

**Step 5: Create API Gateway**

```bash
# Create REST API
API_ID=$(aws apigateway create-rest-api \
  --name "ContactFormAPI" \
  --region us-east-1 \
  --query 'id' --output text)

echo "API ID: $API_ID"

# Get root resource ID
ROOT_ID=$(aws apigateway get-resources \
  --rest-api-id $API_ID \
  --region us-east-1 \
  --query 'items[0].id' --output text)

# Create /contact resource
RESOURCE_ID=$(aws apigateway create-resource \
  --rest-api-id $API_ID \
  --parent-id $ROOT_ID \
  --path-part contact \
  --region us-east-1 \
  --query 'id' --output text)

# Create POST method
aws apigateway put-method \
  --rest-api-id $API_ID \
  --resource-id $RESOURCE_ID \
  --http-method POST \
  --authorization-type NONE \
  --region us-east-1

# Integrate with Lambda
aws apigateway put-integration \
  --rest-api-id $API_ID \
  --resource-id $RESOURCE_ID \
  --http-method POST \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri arn:aws:apigateway:us-east-1:lambda:path/2015-03-31/functions/arn:aws:lambda:us-east-1:$ACCOUNT_ID:function:ContactFormHandler/invocations \
  --region us-east-1

# Deploy the API
aws apigateway create-deployment \
  --rest-api-id $API_ID \
  --stage-name prod \
  --region us-east-1

echo "Your API endpoint: https://$API_ID.execute-api.us-east-1.amazonaws.com/prod/contact"
```

**Step 6: Grant API Gateway Permission to Invoke Lambda**

```bash
aws lambda add-permission \
  --function-name ContactFormHandler \
  --statement-id apigateway-invoke \
  --action lambda:InvokeFunction \
  --principal apigateway.amazonaws.com \
  --source-arn "arn:aws:execute-api:us-east-1:$ACCOUNT_ID:$API_ID/*/POST/contact" \
  --region us-east-1
```

**Step 7: Test the API**

```bash
# Test with curl
curl -X POST \
  https://$API_ID.execute-api.us-east-1.amazonaws.com/prod/contact \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "email": "test@example.com",
    "message": "This is a test message from the contact form."
  }'
```

---

### What Can Go Wrong and How to Fix It

**CORS errors in the browser:** Ensure the Lambda function returns the correct CORS headers in every response, including error responses. API Gateway also needs an OPTIONS method configured if not using proxy integration.

**Lambda cannot send emails:** Check the IAM role attached to your Lambda has the `ses:SendEmail` permission. Also verify the sender and recipient emails are verified in SES sandbox mode.

**500 errors:** Check CloudWatch Logs for your Lambda function. The logs show the exact exception. Go to CloudWatch → Log Groups → /aws/lambda/ContactFormHandler.

---

### How to Extend This Project

Store form submissions in DynamoDB as a record. Add rate limiting with API Gateway usage plans to prevent abuse. Send an auto-reply to the person who submitted the form. Add Google reCAPTCHA verification in the Lambda to prevent spam. Use SES templates for beautifully formatted emails.

---

## Project 3: Automated EC2 Backup System

**Difficulty:** 🟢 Beginner
**Estimated Time:** 2 hours
**AWS Free Tier Cost:** Free (within free tier)
**Core Services:** EC2, Lambda, CloudWatch Events (EventBridge), SNS, IAM

---

### What You Are Building

An automated system that creates EBS snapshots of your EC2 instances on a schedule, retains only the last N snapshots, and sends email alerts for both successes and failures. This is a real operational task that cloud engineers build and maintain.

```
┌─────────────────────────────────────────────────────────┐
│              EventBridge (CloudWatch Events)             │
│            Cron Schedule: Every day at 2:00 AM          │
└──────────────────────────┬──────────────────────────────┘
                           │ Trigger
                           ▼
              ┌─────────────────────────┐
              │    Lambda Function       │
              │   backup_manager.py      │
              │                          │
              │ 1. Find EC2 instances    │
              │    tagged Backup=true    │
              │ 2. Create EBS snapshots  │
              │ 3. Delete old snapshots  │
              │    (keep last 7)         │
              │ 4. Publish SNS report    │
              └────────────┬────────────┘
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
   ┌─────────────────┐       ┌─────────────────┐
   │   EC2 Instances  │       │   SNS Topic      │
   │   (EBS Volumes)  │       │ backup-alerts    │
   │   SnapShot ✓     │       │                 │
   └─────────────────┘       └────────┬────────┘
                                      │ Email
                                      ▼
                             ┌─────────────────┐
                             │  Your Inbox 📧   │
                             └─────────────────┘
```

---

### What You Will Learn

How EventBridge (formerly CloudWatch Events) schedules automated tasks using cron expressions. How Lambda interacts with EC2 via the boto3 SDK. How to use EC2 tags to identify resources for automation. How EBS snapshots work and their cost implications. How SNS topics and subscriptions send notifications. Building operational automation — a core cloud engineering skill. How to write Lambda functions that handle errors gracefully and report results.

---

### Step-by-Step Build

**Step 1: Launch a Test EC2 Instance**

```bash
# Launch a minimal EC2 instance for testing (t2.micro is free tier)
aws ec2 run-instances \
  --image-id ami-0c02fb55956c7d316 \
  --instance-type t2.micro \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=backup-test},{Key=Backup,Value=true},{Key=RetentionDays,Value=7}]' \
  --region us-east-1 \
  --query 'Instances[0].InstanceId' \
  --output text
```

The tag `Backup=true` is what the Lambda function will search for. This tag-based approach lets you selectively back up only the instances you choose.

**Step 2: Create the SNS Topic**

```bash
# Create SNS topic
TOPIC_ARN=$(aws sns create-topic \
  --name backup-alerts \
  --region us-east-1 \
  --query 'TopicArn' --output text)

# Subscribe your email
aws sns subscribe \
  --topic-arn $TOPIC_ARN \
  --protocol email \
  --notification-endpoint your-email@gmail.com \
  --region us-east-1

echo "Topic ARN: $TOPIC_ARN"
echo "Check your email and confirm the subscription!"
```

**Step 3: Create the Lambda Function**

Create `backup_manager.py`:

```python
import boto3
import json
from datetime import datetime, timezone, timedelta
import os

ec2 = boto3.client('ec2', region_name='us-east-1')
sns = boto3.client('sns', region_name='us-east-1')

SNS_TOPIC_ARN = os.environ.get('SNS_TOPIC_ARN', '')
DEFAULT_RETENTION_DAYS = int(os.environ.get('DEFAULT_RETENTION_DAYS', '7'))

def lambda_handler(event, context):
    print("Starting automated backup process...")
    timestamp = datetime.now(timezone.utc).strftime('%Y-%m-%d %H:%M:%S UTC')
    
    results = {
        'timestamp': timestamp,
        'snapshots_created': [],
        'snapshots_deleted': [],
        'errors': []
    }
    
    # Find all instances tagged with Backup=true
    response = ec2.describe_instances(
        Filters=[{'Name': 'tag:Backup', 'Values': ['true']}]
    )
    
    instances = []
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            if instance['State']['Name'] in ['running', 'stopped']:
                instances.append(instance)
    
    print(f"Found {len(instances)} instances to back up")
    
    for instance in instances:
        instance_id = instance['InstanceId']
        instance_name = next(
            (tag['Value'] for tag in instance.get('Tags', []) if tag['Key'] == 'Name'),
            instance_id
        )
        
        try:
            # Create snapshot for each EBS volume
            for mapping in instance.get('BlockDeviceMappings', []):
                volume_id = mapping['Ebs']['VolumeId']
                device_name = mapping['DeviceName']
                
                snapshot = ec2.create_snapshot(
                    VolumeId=volume_id,
                    Description=f'Automated backup of {instance_name} ({instance_id}) {device_name} - {timestamp}',
                    TagSpecifications=[{
                        'ResourceType': 'snapshot',
                        'Tags': [
                            {'Key': 'Name', 'Value': f'backup-{instance_name}-{device_name}'},
                            {'Key': 'AutoBackup', 'Value': 'true'},
                            {'Key': 'InstanceId', 'Value': instance_id},
                            {'Key': 'CreatedAt', 'Value': timestamp}
                        ]
                    }]
                )
                
                results['snapshots_created'].append({
                    'snapshot_id': snapshot['SnapshotId'],
                    'instance': instance_name,
                    'volume': volume_id
                })
                print(f"Created snapshot {snapshot['SnapshotId']} for volume {volume_id}")
                
                # Delete old snapshots beyond retention period
                deleted = cleanup_old_snapshots(volume_id, instance_id)
                results['snapshots_deleted'].extend(deleted)
                
        except Exception as e:
            error_msg = f"Error backing up {instance_name}: {str(e)}"
            print(error_msg)
            results['errors'].append(error_msg)
    
    # Send notification
    send_notification(results)
    
    return {'statusCode': 200, 'body': json.dumps(results)}

def cleanup_old_snapshots(volume_id, instance_id, max_snapshots=7):
    """Keep only the most recent max_snapshots snapshots for a volume."""
    response = ec2.describe_snapshots(
        Filters=[
            {'Name': 'volume-id', 'Values': [volume_id]},
            {'Name': 'tag:AutoBackup', 'Values': ['true']},
            {'Name': 'tag:InstanceId', 'Values': [instance_id]}
        ],
        OwnerIds=['self']
    )
    
    snapshots = sorted(
        response['Snapshots'],
        key=lambda x: x['StartTime'],
        reverse=True
    )
    
    deleted = []
    snapshots_to_delete = snapshots[max_snapshots:]
    
    for snapshot in snapshots_to_delete:
        ec2.delete_snapshot(SnapshotId=snapshot['SnapshotId'])
        deleted.append(snapshot['SnapshotId'])
        print(f"Deleted old snapshot: {snapshot['SnapshotId']}")
    
    return deleted

def send_notification(results):
    if not SNS_TOPIC_ARN:
        print("No SNS_TOPIC_ARN configured, skipping notification")
        return
    
    created_count = len(results['snapshots_created'])
    deleted_count = len(results['snapshots_deleted'])
    error_count = len(results['errors'])
    
    status = "✅ SUCCESS" if error_count == 0 else "⚠️ COMPLETED WITH ERRORS"
    
    message = f"""
EC2 Backup Report — {results['timestamp']}
Status: {status}

Snapshots Created: {created_count}
Old Snapshots Deleted: {deleted_count}
Errors: {error_count}

CREATED SNAPSHOTS:
{chr(10).join([f"  - {s['snapshot_id']} ({s['instance']} / {s['volume']})" for s in results['snapshots_created']]) or '  None'}

ERRORS:
{chr(10).join([f"  - {e}" for e in results['errors']]) or '  None'}
    """
    
    sns.publish(
        TopicArn=SNS_TOPIC_ARN,
        Subject=f'EC2 Backup Report — {status}',
        Message=message
    )
```

**Step 4: Create IAM Role with Required Permissions**

```bash
cat > backup-trust-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{"Effect": "Allow", "Principal": {"Service": "lambda.amazonaws.com"}, "Action": "sts:AssumeRole"}]
}
EOF

aws iam create-role --role-name BackupLambdaRole --assume-role-policy-document file://backup-trust-policy.json
aws iam attach-role-policy --role-name BackupLambdaRole --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole

cat > backup-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeVolumes",
        "ec2:CreateSnapshot",
        "ec2:DeleteSnapshot",
        "ec2:DescribeSnapshots",
        "ec2:CreateTags"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "*"
    }
  ]
}
EOF

aws iam put-role-policy --role-name BackupLambdaRole --policy-name BackupPolicy --policy-document file://backup-policy.json
```

**Step 5: Deploy Lambda and Set Environment Variables**

```bash
zip backup.zip backup_manager.py

ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

aws lambda create-function \
  --function-name EC2BackupManager \
  --runtime python3.12 \
  --handler backup_manager.lambda_handler \
  --role arn:aws:iam::$ACCOUNT_ID:role/BackupLambdaRole \
  --zip-file fileb://backup.zip \
  --timeout 300 \
  --environment "Variables={SNS_TOPIC_ARN=$TOPIC_ARN,DEFAULT_RETENTION_DAYS=7}" \
  --region us-east-1
```

**Step 6: Create EventBridge Schedule**

```bash
# Create a rule that triggers daily at 2 AM UTC
aws events put-rule \
  --name DailyEC2Backup \
  --schedule-expression "cron(0 2 * * ? *)" \
  --state ENABLED \
  --region us-east-1

# Add Lambda as the target
aws events put-targets \
  --rule DailyEC2Backup \
  --targets "Id=BackupLambda,Arn=arn:aws:lambda:us-east-1:$ACCOUNT_ID:function:EC2BackupManager" \
  --region us-east-1

# Grant EventBridge permission to invoke Lambda
aws lambda add-permission \
  --function-name EC2BackupManager \
  --statement-id eventbridge-invoke \
  --action lambda:InvokeFunction \
  --principal events.amazonaws.com \
  --source-arn arn:aws:events:us-east-1:$ACCOUNT_ID:rule/DailyEC2Backup \
  --region us-east-1
```

**Step 7: Test Immediately**

```bash
# Invoke manually to test
aws lambda invoke \
  --function-name EC2BackupManager \
  --payload '{}' \
  --region us-east-1 \
  output.json

cat output.json
```

Check your email for the backup report and check the EC2 console → Snapshots to see the created snapshot.

---

## Project 4: Three-Tier Web Application

**Difficulty:** 🟡 Intermediate
**Estimated Time:** 4–5 hours
**Estimated Cost:** ~$2 (terminate all resources within 24 hours)
**Core Services:** VPC, EC2, ALB, Auto Scaling, RDS, Security Groups

---

### What You Are Building

The canonical cloud architecture for web applications — a properly secured, highly available three-tier architecture with a web tier, application tier, and database tier, each in their own subnet with appropriate access controls.

```
                        Internet
                           │
                    ┌──────▼──────┐
                    │   Internet   │
                    │   Gateway   │
                    └──────┬──────┘
                           │
              ─────────────┼─────────────
              │      PUBLIC SUBNETS      │
              │  ┌────────────────────┐  │
              │  │  Application Load  │  │
              │  │  Balancer (ALB)    │  │
              │  └────────┬───────────┘  │
              ─────────────┼─────────────
                           │
              ─────────────┼─────────────
              │     PRIVATE SUBNETS      │
              │  (App Tier - AZ-a)  (App Tier - AZ-b) │
              │  ┌──────────────┐  ┌──────────────┐  │
              │  │  EC2 t2.micro│  │  EC2 t2.micro│  │
              │  │  (Web App)   │  │  (Web App)   │  │
              │  └──────┬───────┘  └──────┬───────┘  │
              ─────────────┼───────────────┼──────────
                           │               │
              ─────────────┼───────────────┼──────────
              │    DATA SUBNETS (Multi-AZ)            │
              │  ┌──────────────────────────────────┐ │
              │  │  RDS MySQL (Multi-AZ)             │ │
              │  │  Primary (AZ-a) | Standby (AZ-b) │ │
              │  └──────────────────────────────────┘ │
              ──────────────────────────────────────────
```

---

### What You Will Learn

How to design and build a complete VPC from scratch. The difference between public and private subnets and when to use each. How Application Load Balancers distribute traffic and perform health checks. How Auto Scaling Groups maintain availability and scale capacity. How security groups implement the principle of least privilege at the network level — the web tier can only receive HTTP/HTTPS from the internet, the app tier can only receive traffic from the load balancer, the database tier can only receive connections from the app tier. How RDS Multi-AZ provides high availability. How NAT Gateways allow private instances to download updates without being internet-accessible.

---

### Architecture Design Decisions — Understanding the Why

Before building, understand every design decision.

The load balancer lives in public subnets because it must receive traffic from the internet. The application servers live in private subnets — they are never directly accessible from the internet, only from the load balancer. This means even if your application has a vulnerability, an attacker cannot directly reach the server without compromising the load balancer first.

The database lives in data subnets (also private) and can only accept connections from the application security group. Even if an attacker compromised an app server, they would need separate credentials and network access to reach the database.

The NAT Gateway lives in the public subnet. It allows private subnet instances to initiate outbound internet connections (for package updates, API calls) while remaining unreachable from inbound internet traffic.

---

### Step-by-Step Build

**Step 1: Create the VPC and Subnets**

```bash
# Create VPC
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=three-tier-vpc}]' \
  --query 'Vpc.VpcId' --output text --region us-east-1)

# Enable DNS hostnames (required for RDS)
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-hostnames

echo "VPC: $VPC_ID"

# Create public subnets (for ALB and NAT Gateway)
PUB_SUB_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=public-1a}]' \
  --query 'Subnet.SubnetId' --output text)

PUB_SUB_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=public-1b}]' \
  --query 'Subnet.SubnetId' --output text)

# Create private subnets (for application servers)
PRIV_SUB_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.11.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=private-app-1a}]' \
  --query 'Subnet.SubnetId' --output text)

PRIV_SUB_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.12.0/24 \
  --availability-zone us-east-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=private-app-1b}]' \
  --query 'Subnet.SubnetId' --output text)

# Create data subnets (for RDS)
DATA_SUB_1=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.21.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=data-1a}]' \
  --query 'Subnet.SubnetId' --output text)

DATA_SUB_2=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.22.0/24 \
  --availability-zone us-east-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=data-1b}]' \
  --query 'Subnet.SubnetId' --output text)

echo "Subnets created."
```

**Step 2: Create Internet Gateway and Route Tables**

```bash
# Create and attach Internet Gateway
IGW_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=three-tier-igw}]' \
  --query 'InternetGateway.InternetGatewayId' --output text)

aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $IGW_ID

# Create public route table with internet route
PUB_RT=$(aws ec2 create-route-table --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=public-rt}]' \
  --query 'RouteTable.RouteTableId' --output text)

aws ec2 create-route --route-table-id $PUB_RT \
  --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID

# Associate public subnets with public route table
aws ec2 associate-route-table --subnet-id $PUB_SUB_1 --route-table-id $PUB_RT
aws ec2 associate-route-table --subnet-id $PUB_SUB_2 --route-table-id $PUB_RT

# Create NAT Gateway in public subnet (for private subnet outbound internet)
EIP=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
NAT_GW=$(aws ec2 create-nat-gateway --subnet-id $PUB_SUB_1 \
  --allocation-id $EIP \
  --query 'NatGateway.NatGatewayId' --output text)

echo "Waiting for NAT Gateway to become available..."
aws ec2 wait nat-gateway-available --nat-gateway-ids $NAT_GW

# Create private route table with NAT Gateway route
PRIV_RT=$(aws ec2 create-route-table --vpc-id $VPC_ID \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=private-rt}]' \
  --query 'RouteTable.RouteTableId' --output text)

aws ec2 create-route --route-table-id $PRIV_RT \
  --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_GW

aws ec2 associate-route-table --subnet-id $PRIV_SUB_1 --route-table-id $PRIV_RT
aws ec2 associate-route-table --subnet-id $PRIV_SUB_2 --route-table-id $PRIV_RT
```

**Step 3: Create Security Groups**

```bash
# ALB Security Group — accepts HTTP/HTTPS from anywhere
ALB_SG=$(aws ec2 create-security-group \
  --group-name alb-sg --description "ALB Security Group" \
  --vpc-id $VPC_ID --query 'GroupId' --output text)

aws ec2 authorize-security-group-ingress --group-id $ALB_SG \
  --protocol tcp --port 80 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id $ALB_SG \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

# App Server Security Group — accepts HTTP only from ALB
APP_SG=$(aws ec2 create-security-group \
  --group-name app-sg --description "App Server Security Group" \
  --vpc-id $VPC_ID --query 'GroupId' --output text)

aws ec2 authorize-security-group-ingress --group-id $APP_SG \
  --protocol tcp --port 80 --source-group $ALB_SG

# Database Security Group — accepts MySQL only from App tier
DB_SG=$(aws ec2 create-security-group \
  --group-name db-sg --description "Database Security Group" \
  --vpc-id $VPC_ID --query 'GroupId' --output text)

aws ec2 authorize-security-group-ingress --group-id $DB_SG \
  --protocol tcp --port 3306 --source-group $APP_SG
```

**Step 4: Create Launch Template and Auto Scaling Group**

```bash
# User data script — installs and starts a web server with a simple app
USER_DATA=$(cat << 'USERDATA' | base64 -w 0
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
AZ=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
cat > /var/www/html/index.html << EOF
<html>
<body style="font-family:Arial;text-align:center;padding:50px;background:#f0f4f8">
<h1>Three-Tier Architecture ☁️</h1>
<p>Instance ID: $INSTANCE_ID</p>
<p>Availability Zone: $AZ</p>
<p>Served by EC2 via ALB in a private subnet</p>
</body>
</html>
EOF
USERDATA
)

# Create launch template
LT_ID=$(aws ec2 create-launch-template \
  --launch-template-name three-tier-lt \
  --launch-template-data "{
    \"ImageId\": \"ami-0c02fb55956c7d316\",
    \"InstanceType\": \"t2.micro\",
    \"SecurityGroupIds\": [\"$APP_SG\"],
    \"UserData\": \"$USER_DATA\",
    \"TagSpecifications\": [{
      \"ResourceType\": \"instance\",
      \"Tags\": [{\"Key\": \"Name\", \"Value\": \"three-tier-app\"}]
    }]
  }" \
  --query 'LaunchTemplate.LaunchTemplateId' --output text)

# Create ALB target group
TG_ARN=$(aws elbv2 create-target-group \
  --name three-tier-tg \
  --protocol HTTP --port 80 \
  --vpc-id $VPC_ID \
  --health-check-path "/" \
  --health-check-interval-seconds 30 \
  --query 'TargetGroups[0].TargetGroupArn' --output text)

# Create Application Load Balancer
ALB_ARN=$(aws elbv2 create-load-balancer \
  --name three-tier-alb \
  --subnets $PUB_SUB_1 $PUB_SUB_2 \
  --security-groups $ALB_SG \
  --query 'LoadBalancers[0].LoadBalancerArn' --output text)

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn $ALB_ARN \
  --protocol HTTP --port 80 \
  --default-actions Type=forward,TargetGroupArn=$TG_ARN

# Create Auto Scaling Group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name three-tier-asg \
  --launch-template LaunchTemplateId=$LT_ID,Version='$Latest' \
  --min-size 2 --max-size 4 --desired-capacity 2 \
  --vpc-zone-identifier "$PRIV_SUB_1,$PRIV_SUB_2" \
  --target-group-arns $TG_ARN \
  --health-check-type ELB \
  --health-check-grace-period 300

# Get ALB DNS name
ALB_DNS=$(aws elbv2 describe-load-balancers \
  --load-balancer-arns $ALB_ARN \
  --query 'LoadBalancers[0].DNSName' --output text)

echo "✅ Three-tier architecture deployed!"
echo "Access your application at: http://$ALB_DNS"
```

Refresh the URL multiple times — you will see the instance ID change, confirming the load balancer is distributing requests across both instances in different availability zones.

---

## Project 5: Serverless Image Processing Pipeline

**Difficulty:** 🟡 Intermediate
**Estimated Time:** 3 hours
**AWS Free Tier Cost:** Free
**Core Services:** S3, Lambda, DynamoDB, SNS, IAM

---

### What You Are Building

An event-driven pipeline that automatically processes images the moment they are uploaded to S3 — extracting metadata, generating thumbnails, storing records in DynamoDB, and sending notifications.

```
  User Uploads Image
         │
         ▼
┌─────────────────────┐
│  S3 Bucket          │
│  (uploads/)         │ ──── S3 Event Notification ────►
└─────────────────────┘                                  │
                                                         ▼
                                              ┌────────────────────┐
                                              │  Lambda Function    │
                                              │  image_processor    │
                                              │                     │
                                              │ 1. Extract metadata │
                                              │ 2. Resize → thumb   │
                                              │ 3. Save to DynamoDB │
                                              │ 4. Notify via SNS   │
                                              └──────────┬─────────┘
                                                         │
                              ┌──────────────────────────┼──────────────────────┐
                              │                          │                      │
                              ▼                          ▼                      ▼
                   ┌──────────────────┐      ┌──────────────────┐    ┌──────────────────┐
                   │  S3 Bucket       │      │  DynamoDB Table  │    │  SNS Topic       │
                   │  (thumbnails/)   │      │  (image-records) │    │  (notifications) │
                   └──────────────────┘      └──────────────────┘    └──────────────────┘
```

---

### What You Will Learn

S3 event notifications and how they trigger Lambda. How to use Lambda Layers for third-party dependencies (Pillow for image processing). DynamoDB data modeling — designing a table for image metadata. Event-driven architecture patterns. How to write a Lambda function that coordinates multiple AWS services. How to debug event-driven systems using CloudWatch Logs.

---

### Step-by-Step Build

**Step 1: Create S3 Buckets and DynamoDB Table**

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
BUCKET_NAME="image-pipeline-$ACCOUNT_ID"
THUMB_BUCKET="image-thumbnails-$ACCOUNT_ID"

aws s3 mb s3://$BUCKET_NAME --region us-east-1
aws s3 mb s3://$THUMB_BUCKET --region us-east-1

# Create DynamoDB table for image records
aws dynamodb create-table \
  --table-name image-records \
  --attribute-definitions \
    AttributeName=imageId,AttributeType=S \
  --key-schema \
    AttributeName=imageId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region us-east-1
```

**Step 2: Create Lambda with Pillow Layer**

AWS provides a managed Lambda layer with Pillow (Python image processing library). Use it rather than packaging Pillow yourself.

Create `image_processor.py`:

```python
import boto3
import json
import os
import uuid
from datetime import datetime, timezone
from PIL import Image
import io

s3 = boto3.client('s3')
dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')

TABLE_NAME = os.environ.get('TABLE_NAME', 'image-records')
THUMB_BUCKET = os.environ.get('THUMB_BUCKET', '')
SNS_TOPIC_ARN = os.environ.get('SNS_TOPIC_ARN', '')
THUMB_MAX_SIZE = (300, 300)

def lambda_handler(event, context):
    table = dynamodb.Table(TABLE_NAME)
    
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        size_bytes = record['s3']['object']['size']
        
        # Only process images
        if not key.lower().endswith(('.jpg', '.jpeg', '.png', '.gif', '.webp')):
            print(f"Skipping non-image file: {key}")
            continue
        
        print(f"Processing: s3://{bucket}/{key}")
        
        # Download image
        response = s3.get_object(Bucket=bucket, Key=key)
        image_data = response['Body'].read()
        
        # Open with Pillow
        image = Image.open(io.BytesIO(image_data))
        original_width, original_height = image.size
        image_format = image.format
        color_mode = image.mode
        
        # Generate thumbnail
        thumb = image.copy()
        thumb.thumbnail(THUMB_MAX_SIZE, Image.Resampling.LANCZOS)
        thumb_buffer = io.BytesIO()
        thumb.save(thumb_buffer, format=image_format or 'JPEG', quality=85)
        thumb_buffer.seek(0)
        
        # Upload thumbnail
        thumb_key = f"thumbnails/{key}"
        s3.put_object(
            Bucket=THUMB_BUCKET,
            Key=thumb_key,
            Body=thumb_buffer,
            ContentType=response['ContentType']
        )
        
        thumb_width, thumb_height = thumb.size
        
        # Save record to DynamoDB
        image_id = str(uuid.uuid4())
        timestamp = datetime.now(timezone.utc).isoformat()
        
        table.put_item(Item={
            'imageId': image_id,
            'originalKey': key,
            'sourceBucket': bucket,
            'thumbnailKey': thumb_key,
            'thumbnailBucket': THUMB_BUCKET,
            'originalWidth': original_width,
            'originalHeight': original_height,
            'thumbnailWidth': thumb_width,
            'thumbnailHeight': thumb_height,
            'fileSizeBytes': size_bytes,
            'format': image_format,
            'colorMode': color_mode,
            'processedAt': timestamp,
            'status': 'processed'
        })
        
        print(f"Saved record {image_id} to DynamoDB")
        
        # Send notification
        if SNS_TOPIC_ARN:
            sns.publish(
                TopicArn=SNS_TOPIC_ARN,
                Subject=f'Image Processed: {key}',
                Message=json.dumps({
                    'imageId': image_id,
                    'file': key,
                    'dimensions': f'{original_width}x{original_height}',
                    'thumbnail': f'{thumb_width}x{thumb_height}',
                    'processedAt': timestamp
                }, indent=2)
            )
    
    return {'statusCode': 200, 'body': 'Processing complete'}
```

**Step 3: Deploy and Configure S3 Event Trigger**

After creating the Lambda function (with the Pillow AMS-managed layer ARN for your region), configure S3 to trigger it:

```bash
# Add permission for S3 to invoke Lambda
aws lambda add-permission \
  --function-name ImageProcessor \
  --statement-id s3-invoke \
  --action lambda:InvokeFunction \
  --principal s3.amazonaws.com \
  --source-arn arn:aws:s3:::$BUCKET_NAME \
  --region us-east-1

# Configure S3 event notification
aws s3api put-bucket-notification-configuration \
  --bucket $BUCKET_NAME \
  --notification-configuration "{
    \"LambdaFunctionConfigurations\": [{
      \"LambdaFunctionArn\": \"arn:aws:lambda:us-east-1:$ACCOUNT_ID:function:ImageProcessor\",
      \"Events\": [\"s3:ObjectCreated:*\"],
      \"Filter\": {
        \"Key\": {
          \"FilterRules\": [{
            \"Name\": \"suffix\",
            \"Value\": \".jpg\"
          }]
        }
      }
    }]
  }"
```

**Step 4: Test the Pipeline**

```bash
# Upload a test image
aws s3 cp ~/any-image.jpg s3://$BUCKET_NAME/test-image.jpg

# Wait a few seconds, then check DynamoDB
aws dynamodb scan --table-name image-records --region us-east-1

# Check thumbnail was created
aws s3 ls s3://$THUMB_BUCKET/thumbnails/
```

---

## Project 6: Infrastructure as Code with Terraform

**Difficulty:** 🟡 Intermediate
**Estimated Time:** 3–4 hours
**Estimated Cost:** ~$1 (terminate after testing)
**Core Services:** Terraform, EC2, VPC, RDS, Security Groups

---

### What You Are Building

You will recreate the three-tier architecture from Project 4 — but entirely as Terraform code. The infrastructure becomes version-controlled, repeatable, and destroyable with a single command. This is how every professional cloud team manages infrastructure.

---

### What You Will Learn

Terraform's core concepts: providers, resources, variables, outputs, state, and modules. How IaC enables repeatable, version-controlled infrastructure. The Terraform workflow: `init → plan → apply → destroy`. How to manage Terraform state. How to write reusable modules. The massive difference between manual click-ops and code-defined infrastructure.

---

### Project File Structure

```
terraform-three-tier/
├── main.tf           # Root module — orchestrates everything
├── variables.tf      # Input variable declarations
├── outputs.tf        # Output values
├── terraform.tfvars  # Variable values (do not commit secrets)
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── compute/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   └── database/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
└── README.md
```

**main.tf:**

```hcl
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

module "vpc" {
  source = "./modules/vpc"
  
  vpc_cidr        = var.vpc_cidr
  project_name    = var.project_name
  environment     = var.environment
  azs             = var.availability_zones
}

module "compute" {
  source = "./modules/compute"
  
  project_name      = var.project_name
  environment       = var.environment
  vpc_id            = module.vpc.vpc_id
  public_subnet_ids = module.vpc.public_subnet_ids
  private_subnet_ids = module.vpc.private_subnet_ids
  instance_type     = var.instance_type
  min_size          = var.asg_min_size
  max_size          = var.asg_max_size
  desired_capacity  = var.asg_desired_capacity
}

module "database" {
  source = "./modules/database"
  
  project_name      = var.project_name
  environment       = var.environment
  vpc_id            = module.vpc.vpc_id
  data_subnet_ids   = module.vpc.data_subnet_ids
  app_security_group_id = module.compute.app_security_group_id
  db_username       = var.db_username
  db_password       = var.db_password
  db_name           = var.db_name
}
```

**variables.tf:**

```hcl
variable "aws_region" {
  description = "AWS region to deploy resources"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Name prefix for all resources"
  type        = string
  default     = "three-tier"
}

variable "environment" {
  description = "Environment: dev, staging, prod"
  type        = string
  default     = "dev"
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "List of availability zones to use"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "asg_min_size"         { type = number; default = 2 }
variable "asg_max_size"         { type = number; default = 4 }
variable "asg_desired_capacity" { type = number; default = 2 }

variable "db_username" {
  description = "RDS master username"
  type        = string
  sensitive   = true
}

variable "db_password" {
  description = "RDS master password"
  type        = string
  sensitive   = true
}

variable "db_name" {
  description = "Initial database name"
  type        = string
  default     = "appdb"
}
```

**modules/vpc/main.tf:**

```hcl
locals {
  public_subnet_cidrs = [for i, az in var.azs : cidrsubnet(var.vpc_cidr, 8, i + 1)]
  private_subnet_cidrs = [for i, az in var.azs : cidrsubnet(var.vpc_cidr, 8, i + 11)]
  data_subnet_cidrs   = [for i, az in var.azs : cidrsubnet(var.vpc_cidr, 8, i + 21)]
}

resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name        = "${var.project_name}-${var.environment}-vpc"
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.project_name}-${var.environment}-igw"
  }
}

resource "aws_subnet" "public" {
  count             = length(var.azs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = local.public_subnet_cidrs[count.index]
  availability_zone = var.azs[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.project_name}-public-${var.azs[count.index]}"
    Type = "public"
  }
}

resource "aws_subnet" "private" {
  count             = length(var.azs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = local.private_subnet_cidrs[count.index]
  availability_zone = var.azs[count.index]
  
  tags = {
    Name = "${var.project_name}-private-${var.azs[count.index]}"
    Type = "private"
  }
}

resource "aws_subnet" "data" {
  count             = length(var.azs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = local.data_subnet_cidrs[count.index]
  availability_zone = var.azs[count.index]
  
  tags = {
    Name = "${var.project_name}-data-${var.azs[count.index]}"
    Type = "data"
  }
}

resource "aws_eip" "nat" {
  domain = "vpc"
  depends_on = [aws_internet_gateway.main]
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id
  
  tags = { Name = "${var.project_name}-nat-gw" }
  
  depends_on = [aws_internet_gateway.main]
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  
  tags = { Name = "${var.project_name}-public-rt" }
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id
  
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }
  
  tags = { Name = "${var.project_name}-private-rt" }
}

resource "aws_route_table_association" "public" {
  count          = length(aws_subnet.public)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count          = length(aws_subnet.private)
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}
```

**Terraform Workflow:**

```bash
# Install Terraform (macOS example)
brew tap hashicorp/tap && brew install hashicorp/tap/terraform

cd terraform-three-tier

# Create terraform.tfvars (never commit this file)
cat > terraform.tfvars << 'EOF'
db_username = "admin"
db_password = "YourSecurePassword123!"
environment = "dev"
EOF

# Initialize (downloads providers)
terraform init

# Preview changes — always read this carefully
terraform plan

# Apply changes
terraform apply

# When done, destroy everything to avoid charges
terraform destroy
```

The `terraform plan` output shows you exactly what will be created, modified, or destroyed before anything happens. This is one of the most valuable features of IaC — full visibility into changes before they occur.

---

## Project 7: CI/CD Pipeline for a Web Application

**Difficulty:** 🟡 Intermediate
**Estimated Time:** 4 hours
**Estimated Cost:** ~$2
**Core Services:** GitHub Actions, ECR, ECS Fargate, ALB

---

### What You Are Building

A complete CI/CD pipeline where every `git push` to the main branch automatically: runs tests, builds a Docker container, pushes it to ECR, and deploys it to ECS Fargate — all without manually touching a server.

```
Developer Pushes Code
        │
        ▼
┌───────────────────┐
│  GitHub Repository│
│  (main branch)    │
└────────┬──────────┘
         │ Triggers
         ▼
┌───────────────────────────────────────────────────┐
│              GitHub Actions Pipeline               │
│                                                   │
│  Job 1: Test                                      │
│    ├─ Checkout code                               │
│    ├─ Install dependencies                        │
│    └─ Run unit tests                              │
│                                                   │
│  Job 2: Build & Push (on main only)               │
│    ├─ Build Docker image                          │
│    ├─ Tag with git SHA                            │
│    └─ Push to Amazon ECR                          │
│                                                   │
│  Job 3: Deploy                                    │
│    ├─ Update ECS task definition                  │
│    └─ Deploy new task definition to ECS           │
└───────────────────────────────────────────────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │   Amazon ECR           │
              │   (Container Registry) │
              │   myapp:abc1234        │
              └───────────┬────────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │   ECS Fargate Service  │
              │   (Serverless Containers)│
              │   2 running tasks       │
              └────────────────────────┘
```

---

### What You Will Learn

How Docker containers work in a real deployment pipeline. Amazon ECR as a private Docker registry. ECS Fargate — running containers without managing EC2 instances. Task definitions — how ECS knows what container to run. GitHub Actions — the most important CI/CD platform to know. Using OIDC federation to allow GitHub Actions to authenticate to AWS without long-lived access keys. Blue/green deployments — deploying without downtime.

---

### Step-by-Step Build

**Step 1: Create the Application**

```bash
mkdir cicd-project && cd cicd-project

# Simple Python Flask app
cat > app.py << 'EOF'
from flask import Flask, jsonify
import os

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify({
        'message': 'Hello from ECS Fargate!',
        'version': os.environ.get('APP_VERSION', 'unknown'),
        'environment': os.environ.get('ENVIRONMENT', 'unknown')
    })

@app.route('/health')
def health():
    return jsonify({'status': 'healthy'}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
EOF

cat > requirements.txt << 'EOF'
flask==3.0.0
gunicorn==21.2.0
pytest==7.4.0
EOF

cat > test_app.py << 'EOF'
import pytest
from app import app

@pytest.fixture
def client():
    app.config['TESTING'] = True
    with app.test_client() as client:
        yield client

def test_home(client):
    response = client.get('/')
    assert response.status_code == 200
    data = response.get_json()
    assert 'message' in data

def test_health(client):
    response = client.get('/health')
    assert response.status_code == 200
    assert response.get_json()['status'] == 'healthy'
EOF

cat > Dockerfile << 'EOF'
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8080
CMD ["gunicorn", "--bind", "0.0.0.0:8080", "--workers", "2", "app:app"]
EOF
```

**Step 2: Create ECR Repository and ECS Infrastructure**

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REGION="us-east-1"

# Create ECR repository
aws ecr create-repository \
  --repository-name my-webapp \
  --region $REGION

# Create ECS cluster
aws ecs create-cluster \
  --cluster-name production \
  --capacity-providers FARGATE \
  --region $REGION
```

Create the ECS task definition (`task-definition.json`):

```json
{
  "family": "my-webapp",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::ACCOUNT_ID:role/ecsTaskExecutionRole",
  "containerDefinitions": [{
    "name": "my-webapp",
    "image": "ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/my-webapp:latest",
    "portMappings": [{"containerPort": 8080, "protocol": "tcp"}],
    "environment": [
      {"name": "ENVIRONMENT", "value": "production"},
      {"name": "APP_VERSION", "value": "1.0.0"}
    ],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/my-webapp",
        "awslogs-region": "us-east-1",
        "awslogs-stream-prefix": "ecs"
      }
    },
    "healthCheck": {
      "command": ["CMD-SHELL", "curl -f http://localhost:8080/health || exit 1"],
      "interval": 30,
      "timeout": 5,
      "retries": 3
    }
  }]
}
```

**Step 3: GitHub Actions Workflow**

Create `.github/workflows/deploy.yml`:

```yaml
name: Build and Deploy to ECS

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: my-webapp
  ECS_SERVICE: my-webapp-service
  ECS_CLUSTER: production
  CONTAINER_NAME: my-webapp

permissions:
  id-token: write   # Required for OIDC authentication
  contents: read

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.12'
      
      - name: Install dependencies
        run: pip install -r requirements.txt
      
      - name: Run tests
        run: pytest test_app.py -v

  deploy:
    name: Build, Push, and Deploy
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials (OIDC — no long-lived keys)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::${{ secrets.AWS_ACCOUNT_ID }}:role/GitHubActionsRole
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Build, tag, and push image to ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT
      
      - name: Download current task definition
        run: |
          aws ecs describe-task-definition \
            --task-definition my-webapp \
            --query taskDefinition > task-definition.json
      
      - name: Update task definition with new image
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.build-image.outputs.image }}
      
      - name: Deploy to ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
```

Add `AWS_ACCOUNT_ID` to your GitHub repository secrets. Create the OIDC identity provider and IAM role in AWS to allow GitHub Actions to authenticate without access keys — this is the secure, modern approach.

---

## Project 8: Kubernetes Microservices Deployment

**Difficulty:** 🔴 Advanced
**Estimated Time:** 5–6 hours
**Estimated Cost:** ~$3
**Core Services:** EKS, ECR, ALB, IAM, kubectl

---

### What You Are Building

A two-service microservices application deployed on Amazon EKS — an API service and a frontend service — with an ingress controller routing traffic, Kubernetes-managed secrets, ConfigMaps, and horizontal pod autoscaling.

```
          Internet
             │
             ▼
  ┌─────────────────────┐
  │  AWS ALB Ingress     │
  │  (Ingress Controller)│
  │  myapp.example.com   │
  └──────────┬──────────┘
             │
    ┌────────┴────────┐
    │                 │
    ▼                 ▼
/api/*           /frontend/*
    │                 │
    ▼                 ▼
┌──────────┐    ┌──────────────┐
│ api-svc   │    │ frontend-svc │
│ (Service) │    │  (Service)   │
└─────┬─────┘    └──────┬───────┘
      │                 │
      ▼                 ▼
┌──────────┐    ┌──────────────┐
│ API Pods  │    │Frontend Pods │
│(3 replicas)│   │(2 replicas)  │
│ Port 8080 │    │  Port 3000   │
└──────────┘    └──────────────┘
      │
      ▼
┌──────────────┐
│  DynamoDB    │
│  (via IAM    │
│   Role for   │
│  ServiceAcct)│
└──────────────┘
```

---

### What You Will Learn

Creating and connecting to an EKS cluster. Writing Kubernetes manifests: Deployments, Services, Ingress, ConfigMaps, Secrets, HorizontalPodAutoscaler. How to use IAM Roles for Service Accounts (IRSA) to give pods AWS permissions without credentials. The AWS Load Balancer Controller for provisioning ALBs from Kubernetes. Kubernetes namespaces for environment isolation. Rolling deployments and rollback. Kubernetes resource requests and limits.

---

### Step-by-Step Build

**Step 1: Create EKS Cluster with eksctl**

```bash
# Install eksctl
brew tap weaveworks/tap && brew install weaveworks/tap/eksctl

# Create cluster (this takes ~15 minutes)
eksctl create cluster \
  --name production \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 2 \
  --nodes-max 4 \
  --with-oidc \
  --ssh-access \
  --managed

# Verify connection
kubectl get nodes
```

**Step 2: Kubernetes Manifests**

Create `k8s/api-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
  namespace: production
  labels:
    app: api-service
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-service
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: api-service
        version: v1
    spec:
      serviceAccountName: api-service-sa
      containers:
        - name: api
          image: ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/api-service:latest
          ports:
            - containerPort: 8080
          env:
            - name: ENVIRONMENT
              valueFrom:
                configMapKeyRef:
                  name: app-config
                  key: environment
            - name: DB_SECRET_ARN
              valueFrom:
                secretKeyRef:
                  name: api-secrets
                  key: db_secret_arn
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "200m"
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: api-service
  namespace: production
spec:
  selector:
    app: api-service
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-service-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

Create `k8s/ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
spec:
  rules:
    - http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

**Step 3: Deploy and Manage**

```bash
# Create namespace
kubectl create namespace production

# Apply all manifests
kubectl apply -f k8s/ -n production

# Watch rollout
kubectl rollout status deployment/api-service -n production

# Scale manually
kubectl scale deployment api-service --replicas=5 -n production

# Rolling update (change image tag)
kubectl set image deployment/api-service api=ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/api-service:v2

# Watch rollout progress
kubectl rollout status deployment/api-service -n production

# Rollback if something goes wrong
kubectl rollout undo deployment/api-service -n production

# Get all resources
kubectl get all -n production

# Check pod logs
kubectl logs -f deployment/api-service -n production

# Get ingress address (the ALB DNS name)
kubectl get ingress app-ingress -n production
```

---

## Project 9: Real-Time Data Pipeline

**Difficulty:** 🔴 Advanced
**Estimated Time:** 5 hours
**Estimated Cost:** ~$2
**Core Services:** Kinesis Data Streams, Lambda, DynamoDB, S3, Athena, Glue

---

### What You Are Building

A real-time data pipeline that ingests streaming events (simulating e-commerce clickstream data), processes them in real-time, stores them in both a real-time database and a data lake, and enables SQL analytics over the entire dataset.

```
Event Producer
(Script simulating
 clickstream events)
        │
        ▼
┌──────────────────┐
│ Kinesis Data     │
│ Streams          │
│ (2 shards)       │
└──────┬───────────┘
       │
  ┌────┴──────┐
  │           │
  ▼           ▼
┌──────┐  ┌─────────────────────┐
│Lambda│  │Kinesis Data Firehose│
│(Real │  │(Batch to S3 every   │
│Time) │  │ 60 seconds)         │
└──┬───┘  └──────────┬──────────┘
   │                 │
   ▼                 ▼
┌──────────┐   ┌──────────────┐
│DynamoDB  │   │  S3 Data Lake │
│(Hot store│   │  (Parquet)    │
│real-time │   │               │
│aggregates│   └──────┬────────┘
└──────────┘          │
                       ▼
               ┌──────────────┐
               │  AWS Glue    │
               │  (Catalog)   │
               └──────┬───────┘
                       │
                       ▼
               ┌──────────────┐
               │   Athena     │
               │ (SQL Queries)│
               └──────────────┘
```

---

### What You Will Learn

Kinesis Data Streams for real-time event ingestion. How stream processing differs from batch processing. Lambda as a stream processor — reading from Kinesis in real time. DynamoDB for high-speed real-time aggregations. Kinesis Firehose for reliable S3 delivery with automatic batching. AWS Glue Data Catalog for schema management. Athena for serverless SQL queries over S3. The Lambda architecture pattern — combining real-time and batch processing layers.

---

### Step-by-Step Build

**Step 1: Create the Kinesis Stream**

```bash
aws kinesis create-stream \
  --stream-name clickstream-events \
  --shard-count 2 \
  --region us-east-1

# Wait for it to become active
aws kinesis wait stream-exists \
  --stream-name clickstream-events \
  --region us-east-1
```

**Step 2: Create DynamoDB Table for Real-Time Aggregations**

```bash
aws dynamodb create-table \
  --table-name real-time-aggregates \
  --attribute-definitions \
    AttributeName=pk,AttributeType=S \
    AttributeName=sk,AttributeType=S \
  --key-schema \
    AttributeName=pk,KeyType=HASH \
    AttributeName=sk,KeyType=RANGE \
  --billing-mode PAY_PER_REQUEST \
  --region us-east-1
```

**Step 3: Create the Stream Processing Lambda**

Create `stream_processor.py`:

```python
import json
import base64
import boto3
from datetime import datetime, timezone
from collections import defaultdict
from decimal import Decimal

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('real-time-aggregates')

def lambda_handler(event, context):
    """Process Kinesis records in real-time."""
    
    # Aggregate events by page and event type for this batch
    page_views = defaultdict(int)
    event_counts = defaultdict(int)
    
    processed = 0
    errors = 0
    
    for record in event['Records']:
        try:
            # Decode and parse the event
            payload = base64.b64decode(record['kinesis']['data']).decode('utf-8')
            click_event = json.loads(payload)
            
            event_type = click_event.get('event_type', 'unknown')
            page = click_event.get('page', 'unknown')
            user_id = click_event.get('user_id', 'anonymous')
            
            # Aggregate
            page_views[page] += 1
            event_counts[event_type] += 1
            
            processed += 1
            
        except Exception as e:
            print(f"Error processing record: {e}")
            errors += 1
    
    # Write aggregated counts to DynamoDB
    timestamp_minute = datetime.now(timezone.utc).strftime('%Y-%m-%dT%H:%M')
    
    # Update page view counts (atomic increment)
    for page, count in page_views.items():
        table.update_item(
            Key={
                'pk': f'PAGE#{page}',
                'sk': f'MINUTE#{timestamp_minute}'
            },
            UpdateExpression='ADD #count :val SET #ttl = :ttl',
            ExpressionAttributeNames={
                '#count': 'view_count',
                '#ttl': 'expires_at'
            },
            ExpressionAttributeValues={
                ':val': Decimal(str(count)),
                # TTL: keep real-time data for 24 hours
                ':ttl': Decimal(str(int(datetime.now(timezone.utc).timestamp()) + 86400))
            }
        )
    
    print(f"Processed {processed} records, {errors} errors. Pages: {dict(page_views)}")
    
    return {'statusCode': 200, 'processed': processed, 'errors': errors}
```

**Step 4: Create Event Source Mapping**

```bash
# Get stream ARN
STREAM_ARN=$(aws kinesis describe-stream \
  --stream-name clickstream-events \
  --query 'StreamDescription.StreamARN' \
  --output text --region us-east-1)

# Connect Lambda to Kinesis stream
aws lambda create-event-source-mapping \
  --function-name StreamProcessor \
  --event-source-arn $STREAM_ARN \
  --starting-position TRIM_HORIZON \
  --batch-size 100 \
  --bisect-batch-on-function-error \
  --region us-east-1
```

**Step 5: Create an Event Producer to Test**

Create `produce_events.py`:

```python
import boto3
import json
import random
import time
import uuid
from datetime import datetime, timezone

kinesis = boto3.client('kinesis', region_name='us-east-1')

PAGES = ['home', 'products', 'cart', 'checkout', 'about', 'search']
EVENT_TYPES = ['page_view', 'click', 'add_to_cart', 'purchase', 'search']

def generate_event():
    return {
        'event_id': str(uuid.uuid4()),
        'user_id': f'user_{random.randint(1000, 9999)}',
        'session_id': f'session_{random.randint(10000, 99999)}',
        'event_type': random.choice(EVENT_TYPES),
        'page': random.choice(PAGES),
        'timestamp': datetime.now(timezone.utc).isoformat(),
        'properties': {
            'user_agent': 'Mozilla/5.0',
            'referrer': random.choice(['google.com', 'direct', 'facebook.com']),
            'duration_ms': random.randint(100, 5000)
        }
    }

print("Producing events to Kinesis... Press Ctrl+C to stop")
count = 0

while True:
    records = [
        {
            'Data': json.dumps(generate_event()),
            'PartitionKey': f'user-{random.randint(1,10)}'
        }
        for _ in range(10)  # Send 10 events at a time
    ]
    
    kinesis.put_records(StreamName='clickstream-events', Records=records)
    count += 10
    print(f"Sent {count} events...")
    time.sleep(1)
```

**Step 6: Set Up Athena for Historical Analysis**

```bash
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
LAKE_BUCKET="data-lake-$ACCOUNT_ID"

aws s3 mb s3://$LAKE_BUCKET --region us-east-1

# After Firehose delivers data, create a Glue database and table
aws glue create-database --database-input '{"Name": "clickstream_db"}'

# Use Athena to query the data
aws athena start-query-execution \
  --query-string "
    SELECT 
      page,
      event_type,
      COUNT(*) as event_count,
      COUNT(DISTINCT user_id) as unique_users,
      date_trunc('hour', from_iso8601_timestamp(timestamp)) as hour
    FROM clickstream_db.events
    WHERE timestamp >= current_timestamp - interval '24' hour
    GROUP BY 1, 2, 5
    ORDER BY event_count DESC
  " \
  --result-configuration OutputLocation=s3://$LAKE_BUCKET/athena-results/ \
  --region us-east-1
```

---

## Project 10: Multi-Region Disaster Recovery Architecture

**Difficulty:** 🔴 Advanced
**Estimated Time:** 6 hours
**Estimated Cost:** ~$3 (clean up promptly)
**Core Services:** Route 53, RDS Cross-Region Replica, S3 Cross-Region Replication, Lambda, CloudFormation, SNS

---

### What You Are Building

A disaster recovery architecture where your application can automatically failover from a primary region (us-east-1) to a secondary region (us-west-2) if the primary region experiences an outage. This is real-world enterprise resilience engineering.

```
                        ┌──────────────────────────┐
                        │    Route 53 Health Check  │
                        │    + Failover Routing     │
                        └─────────────┬─────────────┘
                                      │
               ┌──────────────────────┴──────────────────────┐
               │ PRIMARY (Active)                              │ SECONDARY (Standby)
               │ us-east-1                                     │ us-west-2
               │                                               │
               │ ┌─────────────┐         ┌─────────────────┐  │
               │ │  ALB        │         │  ALB (standby)  │  │
               │ └──────┬──────┘         └────────┬────────┘  │
               │        │                          │            │
               │ ┌──────▼──────┐         ┌────────▼────────┐  │
               │ │ App Servers  │         │  App Servers    │  │
               │ │ (Active)     │         │  (Warm Standby) │  │
               │ └──────┬──────┘         └────────┬────────┘  │
               │        │                          │            │
               │ ┌──────▼──────┐  Async   ┌────────▼────────┐ │
               │ │ RDS Primary  │ Repl.    │  RDS Read       │ │
               │ │ (Read/Write) │ ──────►  │  Replica        │ │
               │ └─────────────┘          └─────────────────┘ │
               │                                               │
               │ ┌─────────────┐  CRR     ┌─────────────────┐ │
               │ │ S3 Primary   │ ──────►  │ S3 Replica      │ │
               │ └─────────────┘          └─────────────────┘ │
               └───────────────────────────────────────────────┘
                        ▲
          Lambda health monitor checks primary every 60s.
          If primary fails health checks, promotes RDS replica
          to primary and Route 53 shifts traffic to secondary.
```

---

### What You Will Learn

Route 53 health checks and failover routing policies. RDS cross-region read replicas and promotion to primary. S3 cross-region replication. Building automated failover with Lambda. Recovery Time Objective (RTO) and Recovery Point Objective (RPO) — the two key DR metrics. The difference between Active-Active, Active-Passive (warm standby), and Cold Standby DR strategies. CloudFormation StackSets for deploying identical infrastructure across multiple regions.

---

### Understanding DR Strategies Before Building

**Cold Standby (Backup & Restore):** RTO: hours. RPO: hours to days. Just backups stored in another region. Cheapest but slowest recovery.

**Warm Standby:** RTO: minutes. RPO: seconds to minutes. Secondary infrastructure exists but at reduced capacity. Data is replicated continuously. What we are building.

**Active-Active (Multi-Region):** RTO: seconds. RPO: near-zero. Both regions actively serve traffic. Most expensive but highest resilience. Used by Netflix, Amazon, etc.

---

### Step-by-Step Build

**Step 1: Deploy Infrastructure in Both Regions Using CloudFormation**

Create `infrastructure.yaml`:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Multi-region DR infrastructure'

Parameters:
  Environment:
    Type: String
    AllowedValues: [primary, secondary]
  VpcCidr:
    Type: String
    Default: '10.0.0.0/16'
  DBUsername:
    Type: String
    NoEcho: true
  DBPassword:
    Type: String
    NoEcho: true

Conditions:
  IsPrimary: !Equals [!Ref Environment, primary]

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCidr
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Sub '${Environment}-vpc'

  RDSSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: 'RDS subnet group'
      SubnetIds:
        - !Ref DataSubnet1
        - !Ref DataSubnet2

  PrimaryRDS:
    Type: AWS::RDS::DBInstance
    Condition: IsPrimary
    Properties:
      DBInstanceClass: db.t3.micro
      Engine: mysql
      EngineVersion: '8.0'
      MasterUsername: !Ref DBUsername
      MasterUserPassword: !Ref DBPassword
      DBSubnetGroupName: !Ref RDSSubnetGroup
      MultiAZ: true
      BackupRetentionPeriod: 7
      StorageEncrypted: true
      Tags:
        - Key: Environment
          Value: !Ref Environment

Outputs:
  RDSEndpoint:
    Condition: IsPrimary
    Value: !GetAtt PrimaryRDS.Endpoint.Address
    Export:
      Name: !Sub '${Environment}-rds-endpoint'
```

```bash
# Deploy primary region
aws cloudformation deploy \
  --template-file infrastructure.yaml \
  --stack-name dr-primary \
  --parameter-overrides Environment=primary DBUsername=admin DBPassword=SecurePass123 \
  --region us-east-1

# Deploy secondary region
aws cloudformation deploy \
  --template-file infrastructure.yaml \
  --stack-name dr-secondary \
  --parameter-overrides Environment=secondary DBUsername=admin DBPassword=SecurePass123 \
  --region us-west-2
```

**Step 2: Set Up S3 Cross-Region Replication**

```bash
# Enable versioning on both buckets (required for CRR)
aws s3api put-bucket-versioning \
  --bucket primary-app-bucket \
  --versioning-configuration Status=Enabled \
  --region us-east-1

aws s3api put-bucket-versioning \
  --bucket secondary-app-bucket \
  --versioning-configuration Status=Enabled \
  --region us-west-2

# Configure replication
aws s3api put-bucket-replication \
  --bucket primary-app-bucket \
  --replication-configuration '{
    "Role": "arn:aws:iam::ACCOUNT_ID:role/S3ReplicationRole",
    "Rules": [{
      "Status": "Enabled",
      "Filter": {"Prefix": ""},
      "Destination": {
        "Bucket": "arn:aws:s3:::secondary-app-bucket",
        "StorageClass": "STANDARD"
      },
      "DeleteMarkerReplication": {"Status": "Enabled"}
    }]
  }'
```

**Step 3: Create RDS Cross-Region Read Replica**

```bash
PRIMARY_RDS_ARN=$(aws rds describe-db-instances \
  --query 'DBInstances[?DBInstanceIdentifier==`primary-db`].DBInstanceArn' \
  --output text --region us-east-1)

# Create cross-region read replica
aws rds create-db-instance-read-replica \
  --db-instance-identifier secondary-db \
  --source-db-instance-identifier $PRIMARY_RDS_ARN \
  --db-instance-class db.t3.micro \
  --region us-west-2
```

**Step 4: Route 53 Health Checks and Failover**

```bash
# Create health check for primary endpoint
HEALTH_CHECK_ID=$(aws route53 create-health-check \
  --caller-reference $(date +%s) \
  --health-check-config '{
    "Type": "HTTPS",
    "FullyQualifiedDomainName": "primary-alb.us-east-1.elb.amazonaws.com",
    "Port": 443,
    "ResourcePath": "/health",
    "RequestInterval": 30,
    "FailureThreshold": 3
  }' \
  --query 'HealthCheck.Id' --output text)

# Create primary (FAILOVER PRIMARY) DNS record
aws route53 change-resource-record-sets \
  --hosted-zone-id YOUR_ZONE_ID \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "app.yourdomain.com",
        "Type": "A",
        "SetIdentifier": "Primary",
        "Failover": "PRIMARY",
        "HealthCheckId": "'$HEALTH_CHECK_ID'",
        "AliasTarget": {
          "HostedZoneId": "Z35SXDOTRQ7X7K",
          "DNSName": "primary-alb.us-east-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }]
  }'

# Create secondary (FAILOVER SECONDARY) DNS record
aws route53 change-resource-record-sets \
  --hosted-zone-id YOUR_ZONE_ID \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "app.yourdomain.com",
        "Type": "A",
        "SetIdentifier": "Secondary",
        "Failover": "SECONDARY",
        "AliasTarget": {
          "HostedZoneId": "Z1H1FL5HABSF5",
          "DNSName": "secondary-alb.us-west-2.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }]
  }'
```

**Step 5: Automated Failover Lambda**

Create `failover_handler.py`:

```python
import boto3
import json
import os

rds_us_west = boto3.client('rds', region_name='us-west-2')
sns_us_east = boto3.client('sns', region_name='us-east-1')

SNS_TOPIC_ARN = os.environ.get('ALERT_TOPIC_ARN')
REPLICA_IDENTIFIER = os.environ.get('REPLICA_IDENTIFIER', 'secondary-db')

def lambda_handler(event, context):
    """
    Triggered by Route 53 health check alarm when primary fails.
    Promotes RDS read replica to standalone primary.
    """
    
    print(f"Failover triggered. Event: {json.dumps(event)}")
    
    # Promote read replica to primary
    try:
        response = rds_us_west.promote_read_replica(
            DBInstanceIdentifier=REPLICA_IDENTIFIER,
            BackupRetentionPeriod=7
        )
        
        new_endpoint = response['DBInstance']['Endpoint']['Address']
        
        # Notify team
        sns_us_east.publish(
            TopicArn=SNS_TOPIC_ARN,
            Subject='🚨 DR FAILOVER INITIATED',
            Message=f"""
DISASTER RECOVERY FAILOVER HAS BEEN INITIATED

Time: {context.aws_request_id}
Action: RDS read replica promoted to primary
New Primary Endpoint: {new_endpoint}
Region: us-west-2

Route 53 has already shifted traffic to the secondary region.
Please investigate the primary region and plan for failback.

This is an automated message from the DR system.
            """
        )
        
        return {
            'status': 'failover_complete',
            'new_primary': new_endpoint
        }
        
    except Exception as e:
        print(f"Failover error: {e}")
        # Still notify even on error — operators must intervene
        sns_us_east.publish(
            TopicArn=SNS_TOPIC_ARN,
            Subject='🚨 DR FAILOVER FAILED — IMMEDIATE ACTION REQUIRED',
            Message=f"Automated failover failed: {str(e)}. Manual intervention required."
        )
        raise
```

**Step 6: Test the Failover**

```bash
# Simulate primary failure by temporarily blocking traffic
# In a real test, you would modify the ALB security group to block external access

# Monitor Route 53 health check status
aws route53 get-health-check-status \
  --health-check-id $HEALTH_CHECK_ID

# After test, verify traffic shifted to secondary
dig app.yourdomain.com
```

---

## 🧹 Cleanup Checklist

**Always run this after completing any project to avoid unexpected charges.**

```bash
# Terminate EC2 instances
aws ec2 describe-instances --query 'Reservations[].Instances[].InstanceId' --output text | \
  xargs aws ec2 terminate-instances --instance-ids

# Delete RDS instances (snapshot first if needed)
aws rds delete-db-instance --db-instance-identifier primary-db --skip-final-snapshot

# Delete EKS cluster
eksctl delete cluster --name production --region us-east-1

# Delete Kinesis stream
aws kinesis delete-stream --stream-name clickstream-events

# Delete Lambda functions
aws lambda list-functions --query 'Functions[].FunctionName' --output text | \
  tr '\t' '\n' | xargs -I {} aws lambda delete-function --function-name {}

# Empty and delete S3 buckets
aws s3 rb s3://your-bucket-name --force

# Delete CloudFormation stacks
aws cloudformation delete-stack --stack-name dr-primary --region us-east-1
aws cloudformation delete-stack --stack-name dr-secondary --region us-west-2

# Release Elastic IPs (these cost money if unattached)
aws ec2 describe-addresses --query 'Addresses[].AllocationId' --output text | \
  tr '\t' '\n' | xargs -I {} aws ec2 release-address --allocation-id {}

# Delete NAT Gateways (check billing dashboard for any others)
aws ec2 describe-nat-gateways --query 'NatGateways[].NatGatewayId' --output text | \
  tr '\t' '\n' | xargs -I {} aws ec2 delete-nat-gateway --nat-gateway-id {}
```

---

## 📊 Skills Gained by Completing All 10 Projects

| Skill | Projects |
|-------|---------|
| S3 and static hosting | 1, 5, 9, 10 |
| Lambda and serverless | 2, 3, 5, 9, 10 |
| VPC and networking | 4, 6, 8 |
| RDS and databases | 4, 6, 10 |
| DynamoDB | 5, 9 |
| Load balancing and auto-scaling | 4, 6, 7, 8 |
| IAM and security | All projects |
| Terraform / IaC | 6 |
| Docker and containers | 7, 8 |
| Kubernetes | 8 |
| CI/CD pipelines | 7 |
| Event-driven architecture | 3, 5, 9 |
| Real-time data processing | 9 |
| Disaster recovery | 10 |
| Multi-region architecture | 10 |
| Route 53 and DNS | 1, 10 |
| CloudFormation | 10 |
| Cost optimization | Every project (free tier) |

---

## 🎯 After Completing All Projects

You have built a portfolio that demonstrates real, practical cloud engineering ability. Your next steps:

Push every project to individual GitHub repositories with detailed READMEs explaining the architecture, design decisions, and what you learned. Record a 2–3 minute screen recording walking through each deployed project and explaining the architecture. Add the architecture diagrams to your LinkedIn profile. Reference specific projects in job applications and interviews — "I built a multi-region DR system with automated failover..." is far more compelling than "I have cloud experience."

You are now ready to pursue AWS Solutions Architect Associate certification and cloud engineering roles.

---

*Document Version 1.0 | Created for GitHub Repository | February 2026*
*This is a living document — add your own notes, architecture variations, and lessons learned as you build.*
