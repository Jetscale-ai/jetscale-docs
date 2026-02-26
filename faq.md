# FAQ

## General

<details>
<summary><strong>What is JetScale?</strong></summary>

JetScale is an AI-powered cloud cost optimization platform that automatically analyzes your infrastructure and generates actionable recommendations with production-ready infrastructure-as-code. Our specialized AI agents analyze your AWS and Azure resources to identify cost optimization opportunities while maintaining performance and SLAs.

</details>

<details>
<summary><strong>How does JetScale work?</strong></summary>

JetScale uses specialized AI agents trained on specific cloud services (RDS, EC2, EBS, ElastiCache, S3, EKS, Azure VMs). These agents analyze real-time data from CloudWatch, Cost Explorer, and Azure Monitor to identify optimization opportunities. When opportunities are found, JetScale generates production-ready Terraform code and integrates directly into your existing GitOps workflows through GitHub, Jira, or Bitbucket.

</details>

<details>
<summary><strong>Which cloud providers does JetScale support?</strong></summary>

JetScale currently supports AWS and Azure. We analyze services including:
- **AWS**: RDS, EC2, EBS, ElastiCache, S3, EKS
- **Azure**: Azure SQL, Virtual Machines, Managed Disks, Azure Cache for Redis

Additional services (Lambda, DynamoDB, ECS, Azure Cosmos DB, Azure Functions) are coming soon.

</details>

<details>
<summary><strong>How much can I save with JetScale?</strong></summary>

Savings vary based on your infrastructure, but our customers typically see 20-40% reductions in cloud costs. The actual savings depend on factors like current resource utilization, instance types, and infrastructure architecture. JetScale provides detailed cost impact analysis for each recommendation before you implement it.

</details>

## Security & Access

<details>
<summary><strong>What permissions does JetScale need?</strong></summary>

JetScale requires **read-only access** to your cloud accounts:
- **AWS**: Cross-account IAM role with read permissions for Cost Explorer, Compute Optimizer, CloudWatch, and service metadata
- **Azure**: Service principal with Reader, Cost Management Reader, and Monitoring Reader roles

We never require write, modify, or delete permissions. We cannot access actual data stored in your databases, storage accounts, or S3 buckets. We only access metadata and configuration information.

</details>

<details>
<summary><strong>How does JetScale keep my data secure?</strong></summary>

Security is our top priority:
- All access is read-only with no write permissions
- We use AWS STS with External ID for secure cross-account access
- Azure authentication uses certificate-based or secret-based service principals
- All data transmission is encrypted in transit and at rest
- We follow SOC 2 Type II compliance standards
- Access can be revoked instantly by deleting the IAM role or service principal

</details>

<details>
<summary><strong>Can JetScale access my application data?</strong></summary>

No. JetScale only has access to infrastructure metadata, configuration, metrics, and cost data. We cannot access:
- Data stored in databases (RDS, Azure SQL, etc.)
- Files in S3 buckets or Azure Storage
- Application code or secrets
- User credentials or sensitive information

We only analyze resource configurations, utilization metrics, and cost information.

</details>

<details>
<summary><strong>How do I revoke JetScale's access?</strong></summary>

Access can be revoked immediately:
- **AWS**: Delete the IAM role created for JetScale
- **Azure**: Remove role assignments from the service principal or delete the App Registration

Once revoked, JetScale immediately loses all access to your cloud account.

</details>

## Recommendations

<details>
<summary><strong>How are recommendations generated?</strong></summary>

Our AI agents analyze multiple data sources:
1. **Performance metrics**: CPU, memory, IOPS, network utilization from CloudWatch/Azure Monitor
2. **Cost data**: Current spending from Cost Explorer/Cost Management
3. **Configuration**: Instance types, storage classes, cluster topology
4. **AWS/Azure best practices**: Compute Optimizer and Azure Advisor recommendations

Recommendations are validated to ensure they maintain your application's performance requirements and SLAs before being presented.

</details>

<details>
<summary><strong>What format are recommendations provided in?</strong></summary>

JetScale generates production-ready **Terraform code** for each recommendation. This code:
- Follows your existing Terraform module structure and naming conventions
- Includes detailed comments explaining the changes
- Can be reviewed and tested before applying
- Integrates into your GitOps workflow via pull requests

</details>

<details>
<summary><strong>Can I customize recommendation criteria?</strong></summary>

Yes. You can configure:
- **Cost thresholds**: Minimum savings amount to generate recommendations
- **Risk tolerance**: Conservative, balanced, or aggressive optimization strategies
- **Service scope**: Enable/disable specific services or resource types
- **Performance requirements**: Set custom CPU, memory, or IOPS thresholds
- **Business hours**: Define peak/off-peak periods for right-sizing analysis

</details>

<details>
<summary><strong>How often are recommendations updated?</strong></summary>

Recommendations are continuously updated based on your infrastructure changes and usage patterns. New recommendations appear as:
- Infrastructure changes are detected
- Usage patterns evolve over time
- New cost optimization opportunities emerge
- AWS/Azure introduce new instance types or pricing models

</details>

<details>
<summary><strong>What happens if I ignore a recommendation?</strong></summary>

You're always in control. If you ignore a recommendation:
- It will be marked as dismissed in the dashboard
- You can provide feedback on why it wasn't suitable
- Similar recommendations can be suppressed based on your feedback
- You can revisit dismissed recommendations at any time

</details>

## Integration

<details>
<summary><strong>How does JetScale integrate with my workflow?</strong></summary>

JetScale integrates natively with your existing tools:
- **GitHub**: Creates pull requests with Terraform code changes
- **Jira**: Creates tickets for review and approval workflows
- **Bitbucket**: Integrates with Bitbucket pipelines and pull requests
- **Terraform**: Generates code compatible with your existing modules

Recommendations flow through your standard code review and deployment processes.

</details>

<details>
<summary><strong>Does JetScale automatically apply recommendations?</strong></summary>

No. JetScale **never** automatically applies changes to your infrastructure. Every recommendation:
1. Is presented for human review
2. Requires explicit approval
3. Goes through your standard PR/CI process
4. Can be tested in non-production environments first

You maintain complete control over what changes are implemented and when.

</details>

<details>
<summary><strong>Can JetScale work with my existing Terraform code?</strong></summary>

Yes. JetScale analyzes your existing Terraform state and module structure to:
- Generate code that matches your style and conventions
- Respect your naming patterns and resource organization
- Integrate with your existing modules and variables
- Maintain compatibility with your Terraform version

</details>

## Pricing & Support

<details>
<summary><strong>How is JetScale priced?</strong></summary>

JetScale pricing is based on your monthly cloud spend under management. Contact [support@jetscale.ai](mailto:support@jetscale.ai) for detailed pricing information tailored to your infrastructure size and requirements.

</details>

<details>
<summary><strong>Is there a free trial?</strong></summary>

Yes. We offer a 14-day free trial with full platform access. During the trial, you can:
- Connect your AWS and/or Azure accounts
- Receive cost optimization recommendations
- Review generated Terraform code
- Test the GitHub/Jira/Bitbucket integrations

No credit card required to start your trial.

</details>

<details>
<summary><strong>What support options are available?</strong></summary>

We provide:
- **Email support**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: Comprehensive guides and API reference
- **Setup assistance**: Hands-on help with initial configuration
- **Dedicated support**: Available for enterprise customers
- **Community**: GitHub Discussions for questions and feedback

Enterprise customers receive priority support with dedicated Slack channels and SLA guarantees.

</details>

<details>
<summary><strong>Can JetScale help with multi-cloud environments?</strong></summary>

Yes. JetScale supports both AWS and Azure in a single platform. You can:
- Manage multiple AWS accounts and Azure subscriptions
- Get unified cost optimization recommendations across clouds
- Compare costs between AWS and Azure for similar services
- Optimize resources across both providers from one dashboard

</details>
