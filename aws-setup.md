# AWS Setup Guide

This guide will walk you through connecting your AWS account to JetScale securely using IAM role-based access.

## Prerequisites

Before starting, ensure you have:

1. **Enabled AWS Cost Optimization Hub**
   Follow the [AWS Cost Optimization Hub Getting Started Guide](https://aws.amazon.com/aws-cost-management/cost-optimization-hub/getting-started/)

2. **Enabled AWS Compute Optimizer**
   Follow the [AWS Compute Optimizer Getting Started Guide](https://aws.amazon.com/compute-optimizer/getting-started/#topic-0)

3. **AWS Account Administrator Access**
   You'll need permissions to create IAM roles and policies

## Overview

JetScale connects to your AWS account using a **cross-account IAM role** with read-only permissions. This approach:

- Does not require AWS access keys or passwords
- Provides secure, auditable access via AWS STS (Security Token Service)
- Uses External ID for additional security against confused deputy attacks
- Can be revoked instantly by deleting the IAM role
- Follows AWS security best practices

## Setup Process

### Step 1: Generate an External ID

The External ID is a unique identifier that adds an extra layer of security to the cross-account role assumption.

**You have two options:**

#### Option A: We Generate It For You (Recommended)
JetScale will generate a secure External ID and share it with you during onboarding.

#### Option B: You Generate It
If you prefer to generate your own External ID, use one of these methods:

```bash
# Generate a UUID (most common)
uuidgen

# Generate random hex string
openssl rand -hex 32

# Generate base64 encoded random string
openssl rand -base64 32
```

**Security Note:** If you generate your own External ID, please share it with us via encrypted email. Share the decryption password through a separate communication channel.

### Step 2: Create the IAM Role

1. **Navigate to IAM Console**
   Go to AWS Console → IAM → Roles → Create Role

2. **Select Trusted Entity**
   - Choose "AWS account"
   - Select "Another AWS account"
   - Enter the **JetScale Account ID** (provided by JetScale team)

3. **Configure Trust Policy**

   Add the External ID to your trust relationship:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "AWS": "arn:aws:iam::[JETSCALE_ACCOUNT_ID]:root"
         },
         "Action": "sts:AssumeRole",
         "Condition": {
           "StringEquals": {
             "sts:ExternalId": "[YOUR_UNIQUE_EXTERNAL_ID]"
           }
         }
       }
     ]
   }
   ```

4. **Name Your Role**
   - Role name: `jetscale-readonly-access` (or your preferred name)
   - Description: "JetScale read-only access for cost optimization"

### Step 3: Attach Permissions

JetScale requires read-only access to analyze your infrastructure and costs. The permissions policy includes:

**Core Services:**
- **Cost Management**: Cost Explorer, Cost Optimization Hub, Compute Optimizer
- **EC2**: Instance metadata, volumes, pricing information
- **RDS**: Database instances, clusters, and configuration
- **S3**: Bucket information and analytics
- **ElastiCache**: Cache cluster information
- **CloudWatch**: Metrics and monitoring data

**Security Considerations:**
- All permissions are read-only (Describe, Get, List actions only)
- No write, modify, or delete permissions
- No access to actual data stored in databases or S3 buckets
- No access to secrets or credentials
- Limited to metadata and configuration information

During onboarding, the JetScale team will provide the complete IAM policy document and help validate permissions.

### Step 4: Share Role ARN

Once the role is created:

1. Copy the **Role ARN** (format: `arn:aws:iam::123456789012:role/jetscale-readonly-access`)
2. Share it with your JetScale contact along with:
   - AWS Account ID
   - External ID (if you generated it yourself)
   - AWS Region (if you want to limit scope)

## Validation

The JetScale team will validate the connection during onboarding by:

1. Attempting to assume the role
2. Running read-only queries against key services
3. Confirming data access for cost analysis

If any permissions are missing, we'll work with you to resolve them before proceeding.

## Security Best Practices

### Regular Reviews
- Review CloudTrail logs periodically to audit JetScale's API calls
- Set up CloudWatch alarms for unusual activity

### Least Privilege
- The provided policy follows AWS least-privilege principles
- Request permission adjustments if your security policy requires it

### Role Expiration (Optional)
Consider adding a maximum session duration to the role:
- Recommended: 12 hours
- This limits the validity of assumed credentials

## Troubleshooting

### Common Issues

**"Access Denied" errors during validation:**
- Verify the External ID matches exactly
- Confirm the JetScale Account ID is correct
- Check that the role's trust policy allows `sts:AssumeRole`

**Missing cost data:**
- Ensure Cost Explorer is enabled (24-hour activation period)
- Confirm Cost Optimization Hub is enabled
- Verify Compute Optimizer has opt-in status

**Regional access issues:**
- JetScale analyzes resources across all enabled regions
- Ensure the role is global (not region-specific)

## Support

Need help with AWS setup?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Setup Calls**: Available for hands-on guidance
- **Documentation**: Detailed permission requirements provided during onboarding

All permission issues will be resolved during the validation process—we're here to help!
