# Bitbucket Integration

JetScale integrates with Bitbucket to automatically create pull requests containing production-ready Terraform code for your cost optimization recommendations.

## Overview

The Bitbucket integration allows JetScale to:
- Push generated Terraform code to your Bitbucket repositories
- Create pull requests with detailed optimization recommendations
- Integrate seamlessly with your existing code review and CI/CD workflows
- Track recommendation implementation status through Bitbucket pipelines

## Prerequisites

Before connecting Bitbucket, you'll need:

1. **Bitbucket Account** (Cloud or Server)
2. **Repository Access** with write permissions
3. **App Password** or **Personal Access Token** with appropriate scopes

### Supported Bitbucket Versions

- **Bitbucket Cloud** (bitbucket.org) - Recommended
- **Bitbucket Server** / **Data Center** (self-hosted)

## Setup Guide

### Step 1: Create a Bitbucket App Password

![Bitbucket App Password Creation](#)
*Screenshot placeholder: Bitbucket app password settings*

#### For Bitbucket Cloud:

1. Navigate to [Bitbucket Settings → App passwords](https://bitbucket.org/account/settings/app-passwords/)
2. Click **Create app password**
3. Give it a descriptive label (e.g., "JetScale Integration")
4. Select the required permissions:
   - ✓ **Account: Read**
   - ✓ **Workspace membership: Read**
   - ✓ **Projects: Read**
   - ✓ **Repositories: Write**
   - ✓ **Pull requests: Write**
5. Click **Create**
6. **Important**: Copy your app password immediately - you won't see it again

#### For Bitbucket Server:

1. Navigate to **Profile Settings → Personal access tokens**
2. Click **Create a token**
3. Give it a name (e.g., "JetScale")
4. Select permissions:
   - ✓ **Repository read**
   - ✓ **Repository write**
5. Set expiration date
6. Click **Create**
7. Copy your token

> **Security Note**: Store your app password or token securely. JetScale encrypts credentials at rest.

### Step 2: Connect Bitbucket in JetScale

![Bitbucket Connection Screen](#)
*Screenshot placeholder: JetScale Bitbucket integration page*

1. Navigate to **Settings** → **Integrations** in JetScale
2. Click **Connect Bitbucket**
3. Enter your configuration:
   - **Bitbucket Type**: Cloud or Server
   - **Server URL**: (For Bitbucket Server only) e.g., `https://bitbucket.yourcompany.com`
   - **Username**: Your Bitbucket username
   - **App Password/Token**: The credential created in Step 1
4. Click **Verify and Connect**

JetScale will verify your credentials and display your accessible repositories.

### Step 3: Select Your Repository

![Repository Selection](#)
*Screenshot placeholder: Bitbucket repository selection dropdown*

1. From the **Repository** dropdown, select your Terraform infrastructure repository
2. Format: `workspace/repository-slug` (Cloud) or `project/repository` (Server)
3. JetScale will verify you have write access
4. Click **Save**

The selected repository will be used for all future pull requests.

## How It Works

### Pull Request Creation

When you approve a cost optimization recommendation in JetScale:

1. **Branch Creation**: JetScale creates a new feature branch from your default branch (usually `main` or `master`)
   - Branch naming: `jetscale/optimization-{resource-type}-{timestamp}`
   - Example: `jetscale/optimization-rds-20240115-143022`

2. **File Push**: Generated Terraform files are committed to the branch
   - Files placed in `terraform/` directory
   - Commit message includes optimization summary and cost impact

3. **Pull Request**: A PR is automatically created with:
   - **Title**: Concise optimization description
   - **Description**: Detailed cost analysis, performance considerations, implementation steps
   - **Reviewers**: Automatically added based on repository settings (optional)

### Pull Request Structure

```markdown
# Cost Optimization: RDS Instance Right-Sizing

## Summary
Downgrade RDS instance `production-db` from **db.r5.2xlarge** to **db.r5.xlarge**

## Cost Impact

| Metric | Current | Projected | Savings |
|--------|---------|-----------|---------|
| Monthly Cost | $730.00 | $365.00 | **$365.00 (50%)** |
| Annual Cost | $8,760.00 | $4,380.00 | **$4,380.00** |

## Performance Analysis

✓ Current CPU utilization: 15-25% average
✓ Current memory utilization: 30-40% average
✓ Recommended instance provides 2x current peak usage
✓ **Risk Assessment**: Low - No performance degradation expected

## Files Changed

- `terraform/rds_production_db.tf` - Updated instance class from db.r5.2xlarge to db.r5.xlarge
- `terraform/variables.tf` - Added configurable instance_class variable

## Implementation Checklist

- [ ] Review changes in Terraform plan output
- [ ] Apply to staging environment first
- [ ] Monitor performance for 24-48 hours
- [ ] Verify application health checks pass
- [ ] Apply to production with change management approval

## Testing Recommendations

1. Run `terraform plan` to preview changes
2. Check for any dependent resources
3. Verify backup/snapshot policies
4. Confirm monitoring dashboards are updated

---
🤖 Generated by [JetScale](https://jetscale.ai) | [View in JetScale Dashboard](#)
```

### Terraform Code Example

```hcl
# Optimized RDS instance configuration
# Generated by JetScale on 2024-01-15

resource "aws_db_instance" "production_db" {
  identifier     = "production-db"
  engine         = "postgres"
  engine_version = "14.7"

  # Previous: db.r5.2xlarge ($730/month)
  # New: db.r5.xlarge ($365/month)
  # Cost Reduction: 50% ($365/month, $4,380/year)
  #
  # Justification:
  # - Average CPU utilization: 15-25%
  # - Average memory utilization: 30-40%
  # - New instance provides 2x headroom above peak usage
  instance_class = var.rds_production_instance_class

  allocated_storage     = 100
  storage_type          = "gp3"
  storage_encrypted     = true

  # ... other configuration unchanged
}
```

## Repository Permissions

JetScale requires **write** access to your repository to create branches and pull requests. The integration will:

✓ Create feature branches
✓ Commit Terraform files
✓ Create pull requests
✓ Read repository metadata and branches

✗ Cannot merge pull requests
✗ Cannot delete branches
✗ Cannot modify settings or webhooks
✗ Cannot access repository secrets or pipeline variables

## Bitbucket Pipeline Integration

### Automatic Validation

JetScale pull requests trigger your existing Bitbucket Pipelines:

```yaml
# bitbucket-pipelines.yml example
pipelines:
  pull-requests:
    '**':
      - step:
          name: Validate Terraform
          script:
            - terraform init
            - terraform fmt -check
            - terraform validate
            - terraform plan
```

### Status Checks

JetScale monitors pipeline status:
- ✅ All checks passed: Ready to merge
- ⚠️ Checks pending: Waiting for pipeline
- ❌ Checks failed: Review pipeline logs

## Managing the Integration

### View Integration Status

From **Settings** → **Integrations** → **Bitbucket**:
- Connected Bitbucket account
- Server URL (if Bitbucket Server)
- Selected workspace/project and repository
- Last verification time
- Recent pull requests created

### Change Repository

1. Navigate to **Settings** → **Integrations** → **Bitbucket**
2. Click **Change Repository**
3. Select a different repository
4. Click **Update**

### Update Credentials

If your app password or token expires:

1. Generate a new app password/token in Bitbucket
2. Navigate to **Settings** → **Integrations** → **Bitbucket**
3. Click **Update Credentials**
4. Enter new credentials
5. Click **Save and Verify**

### Disconnect Bitbucket

To remove the integration:

1. Navigate to **Settings** → **Integrations** → **Bitbucket**
2. Click **Disconnect**
3. Confirm disconnection

**Important**: Disconnecting will:
- Remove stored credentials from JetScale
- Stop automatic PR creation
- Preserve existing pull requests in Bitbucket

## Troubleshooting

### Authentication Failed

**Problem**: "Invalid credentials" error when connecting

**Solutions**:
- Verify username is correct (case-sensitive)
- Check app password/token hasn't expired
- Ensure all required scopes are selected
- For Bitbucket Server, verify server URL is correct and accessible
- Try creating a new app password

### Repository Not Found

**Problem**: Can't find repository in selection dropdown

**Solutions**:
- Verify you have write access to the repository
- Check repository isn't archived
- Ensure workspace/project membership is active
- For Bitbucket Server, confirm project permissions
- Try refreshing the repository list

### Pull Request Creation Failed

**Problem**: Recommendation approved but PR wasn't created

**Solutions**:
- Verify branch doesn't already exist with same name
- Check you still have write access
- Ensure default branch exists and isn't locked
- Confirm repository isn't in read-only mode
- Review Bitbucket API rate limits

### Pipeline Not Triggering

**Problem**: Bitbucket Pipelines doesn't run on JetScale PRs

**Solutions**:
- Verify Pipelines are enabled for the repository
- Check `bitbucket-pipelines.yml` includes pull-request triggers
- Ensure pipeline quota isn't exhausted
- Review branch pattern matching in pipeline configuration

## Best Practices

### Credential Management

- **Dedicated app password**: Create separate credentials for JetScale
- **Set expiration**: Use reasonable expiration dates (90-180 days)
- **Rotate regularly**: Update credentials before expiry
- **Revoke unused**: Delete old app passwords immediately

### Repository Configuration

- **Branch protection**: Require reviews on default branch
- **Required reviewers**: Configure CODEOWNERS for automatic review assignment
- **Pipeline validation**: Enable required pipeline checks before merge
- **Merge strategies**: Use squash or no-fast-forward for clean history

### Pull Request Workflow

1. **Automated validation**: Let pipelines validate Terraform changes
2. **Peer review**: Have infrastructure team review cost optimizations
3. **Staging deployment**: Test changes in non-production first
4. **Monitoring**: Watch metrics after applying changes
5. **Documentation**: Update runbooks if infrastructure changes

## Bitbucket Cloud vs Server

### Feature Comparison

| Feature | Bitbucket Cloud | Bitbucket Server |
|---------|-----------------|------------------|
| Authentication | App Password | Personal Access Token |
| Repository Format | workspace/repo-slug | project/repo |
| API Version | REST 2.0 | REST 1.0 |
| Webhooks | Native support | Native support |
| Pipelines | Built-in | Plugin required |

### Configuration Differences

**Bitbucket Cloud**:
- Uses workspace-based permissions
- App passwords have granular scopes
- Automatic HTTPS endpoints

**Bitbucket Server**:
- Uses project-based permissions
- Requires full server URL
- May need firewall/VPN configuration

## Security Considerations

### Credential Security

- JetScale encrypts app passwords and tokens at rest (AES-256)
- Credentials never appear in logs or API responses
- All Bitbucket API calls use HTTPS with TLS 1.2+
- Credentials stored separately with restricted access

### Audit Trail

All JetScale actions are logged:
- Bitbucket API calls
- Pull request creation per recommendation
- Failed authentication attempts
- Repository access events

Audit logs available in JetScale dashboard under **Settings** → **Audit Log**.

### Revoking Access

To immediately revoke access:

**Bitbucket Cloud**:
1. Go to [App passwords](https://bitbucket.org/account/settings/app-passwords/)
2. Click **Revoke** next to JetScale password

**Bitbucket Server**:
1. Navigate to **Personal access tokens**
2. Click **Revoke** next to JetScale token

**JetScale**:
1. Go to **Settings** → **Integrations** → **Bitbucket**
2. Click **Disconnect**

## API Reference

For programmatic access, see our [API Documentation](../api-reference.md#bitbucket-integration).

Key endpoints:
- `POST /api/v1/bitbucket/action/connect` - Connect Bitbucket account
- `POST /api/v1/bitbucket/action/select-repo` - Select repository
- `GET /api/v1/bitbucket/repos` - List accessible repositories
- `POST /api/v1/bitbucket/action/pr/create` - Create pull request
- `POST /api/v1/bitbucket/action/disconnect` - Disconnect Bitbucket

## Support

Need help with Bitbucket integration?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: [FAQ](../faq.md)
- **GitHub Issues**: [Report a problem](https://github.com/Jetscale-ai/jetscale-docs/issues)
