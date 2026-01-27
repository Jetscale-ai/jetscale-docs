# Azure Setup Guide

This guide will walk you through connecting your Azure subscription to JetScale securely using Azure App Registration with service principal authentication.

## Prerequisites

Before starting, ensure you have:

1. **Azure Subscription Access**
   You'll need Owner or User Access Administrator role on the subscription

2. **Azure Services Enabled**
   - Azure Cost Management
   - Azure Advisor
   - Azure Monitor

3. **Azure CLI or Portal Access**
   Either Azure Portal or Azure CLI installed for configuration

## Overview

JetScale connects to your Azure subscription using an **App Registration (Service Principal)** with read-only RBAC permissions. This approach:

- Does not require user credentials
- Provides secure, auditable access via Azure Active Directory
- Supports certificate-based authentication (most secure) or client secrets
- Can be revoked instantly through Azure IAM
- Follows Microsoft security best practices

## Setup Process

### Step 1: Create an App Registration

1. **Navigate to Microsoft Entra ID** (formerly Azure Active Directory)
   Azure Portal → Microsoft Entra ID → App registrations

2. **Create New Registration**
   - Click "New registration"
   - **Name**: `jetscale-readonly-access`
   - **Supported account types**: Single tenant
   - **Redirect URI**: Leave blank
   - Click "Register"

3. **Save Important Values**

   After creation, note these IDs (you'll share them with JetScale):
   - **Application (client) ID**
   - **Directory (tenant) ID**
   - **Subscription ID** (from Subscriptions page)

### Step 2: Choose Authentication Method

JetScale supports two authentication methods. Choose the one that fits your security policy:

#### Option A: Certificate-Based Authentication (Recommended)

Certificate-based authentication is the most secure option and recommended for production environments.

**Generate a Certificate:**

```bash
# Generate a self-signed certificate (valid for 2 years)
openssl req -x509 -newkey rsa:4096 -keyout jetscale-cert-key.pem \
  -out jetscale-cert-public.pem -days 730 -nodes \
  -subj "/CN=jetscale-read-access/O=YourOrganization/C=US"

# Extract public certificate for Azure Portal upload
openssl x509 -in jetscale-cert-public.pem -out jetscale-cert.cer -outform DER
```

**Upload to App Registration:**

1. Navigate to your App Registration → Certificates & secrets → Certificates
2. Click "Upload certificate"
3. Select `jetscale-cert.cer`
4. Add description: "JetScale Access Certificate"
5. Click "Add"
6. **Save the Thumbprint** displayed

#### Option B: Client Secret Authentication

If your organization prefers secrets over certificates:

1. Navigate to App Registration → Certificates & secrets
2. Click "New client secret"
3. **Description**: `JetScale Access`
4. **Expires**: Choose duration (recommend 24 months or less per your security policy)
5. Click "Add"
6. **Important**: Copy the secret **Value** immediately—it's only shown once

### Step 3: Assign RBAC Permissions

JetScale requires read-only access at the subscription level. You can choose between built-in roles or a custom role:

#### Option A: Use Built-in Roles (Simplest)

Assign these three built-in roles to your App Registration:

1. Navigate to **Subscriptions** → Select your subscription → **Access control (IAM)**
2. Click **Add** → **Add role assignment**
3. Assign each of the following roles:

| Role | Scope | Purpose |
|------|-------|---------|
| **Reader** | Subscription | Read-only access to all resources |
| **Cost Management Reader** | Subscription | Access to cost and usage data |
| **Monitoring Reader** | Subscription | Access to Azure Monitor metrics |

For each role:
- Search for your App Registration by name (`jetscale-readonly-access`)
- Select it and click "Save"

#### Option B: Create Custom Role (Advanced)

For granular control, create a custom role with specific permissions.

**What the Custom Role Includes:**
- **Compute**: VM metadata, sizing information, performance metrics
- **Cost Management**: Cost analysis, usage data, billing information
- **Storage**: Storage account metadata and analytics
- **Databases**: Azure SQL, MySQL, PostgreSQL, MariaDB, CosmosDB metadata
- **Caching**: Redis and Redis Enterprise configuration
- **Networking**: Virtual networks, load balancers, application gateways
- **Monitoring**: Azure Monitor metrics and insights
- **Advisor**: Azure Advisor recommendations

**Security Safeguards:**
- All actions are read-only
- Explicitly excludes storage account key access (`listkeys`, `regeneratekey`)
- No write, modify, or delete permissions
- No access to actual data stored in databases or storage accounts

The JetScale team will provide the complete custom role definition during onboarding.

### Step 4: Securely Share Credentials

Share the following information with JetScale:

**Required Information:**
- **Tenant ID** (Directory ID)
- **Subscription ID**
- **Application (client) ID**
- **Authentication method chosen** (certificate or secret)
- **Certificate Thumbprint** (if using certificates) OR **Client Secret** (if using secrets)

**Security Best Practices for Sharing:**

#### For Certificate-Based Authentication:

1. Create a credentials metadata file (doesn't contain the private key):
   ```json
   {
     "tenant_id": "[YOUR_TENANT_ID]",
     "subscription_id": "[YOUR_SUBSCRIPTION_ID]",
     "client_id": "[YOUR_CLIENT_ID]",
     "auth_method": "certificate",
     "certificate_thumbprint": "[CERTIFICATE_THUMBPRINT]"
   }
   ```

2. Encrypt both the certificate and metadata file:
   ```bash
   gpg --symmetric --cipher-algo AES256 jetscale-cert-key.pem
   gpg --symmetric --cipher-algo AES256 jetscale-azure-creds.json
   ```

3. Email the encrypted files to your JetScale contact
4. Share the decryption password via a separate channel (e.g., Slack, phone)

#### For Secret-Based Authentication:

1. Create an encrypted credentials file:
   ```bash
   # Create credentials file
   cat > jetscale-azure-creds.json <<EOF
   {
     "tenant_id": "[YOUR_TENANT_ID]",
     "subscription_id": "[YOUR_SUBSCRIPTION_ID]",
     "client_id": "[YOUR_CLIENT_ID]",
     "auth_method": "secret",
     "client_secret": "[YOUR_CLIENT_SECRET]"
   }
   EOF

   # Encrypt with GPG
   gpg --symmetric --cipher-algo AES256 jetscale-azure-creds.json
   ```

2. Email the encrypted file to your JetScale contact
3. Share the decryption password via a separate channel

## Validation

You can test the credentials yourself before sharing:

**For Certificate Authentication:**
```bash
az login --service-principal \
  -u [CLIENT_ID] \
  --certificate jetscale-cert-key.pem \
  --tenant [TENANT_ID]

az account show
```

**For Secret Authentication:**
```bash
az login --service-principal \
  -u [CLIENT_ID] \
  -p [CLIENT_SECRET] \
  --tenant [TENANT_ID]

az account show
```

The JetScale team will also validate access during onboarding by:
1. Authenticating with provided credentials
2. Running read-only queries against key Azure services
3. Confirming data access for cost analysis

## Security Best Practices

### Regular Credential Rotation
- **Certificates**: Rotate before 2-year expiration
- **Secrets**: Rotate every 12-24 months per your security policy

### Activity Monitoring
- Review Azure Activity Logs for service principal activity
- Set up alerts for unusual API patterns

### Least Privilege
- Start with built-in roles; move to custom roles if needed
- Regularly review assigned permissions

### Conditional Access (Optional)
Consider adding conditional access policies:
- IP restrictions for service principal
- Time-based access policies

## Troubleshooting

### Common Issues

**"Authentication failed" errors:**
- Verify Tenant ID and Client ID are correct
- Confirm certificate thumbprint matches (if using certificates)
- Check that secret hasn't expired (if using secrets)

**"Insufficient permissions" errors:**
- Ensure all three built-in roles are assigned (Reader, Cost Management Reader, Monitoring Reader)
- Verify role assignments are at the subscription level, not resource group level

**Missing cost data:**
- Cost data can take 24-48 hours to appear in Cost Management
- Ensure Cost Management is enabled for your subscription

**Multi-subscription access:**
- JetScale can analyze multiple subscriptions
- Assign the same App Registration to additional subscriptions with the same RBAC roles

## Support

Need help with Azure setup?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Setup Calls**: Available for hands-on guidance
- **Documentation**: Detailed permission requirements provided during onboarding

All permission issues will be resolved during the validation process—we're here to help!
