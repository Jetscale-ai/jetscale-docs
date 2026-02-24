# JetScale Documentation

Transform cloud costs into competitive advantage with AI-powered optimization.

---

## What is JetScale?

JetScale automatically discovers cost-saving opportunities in your cloud infrastructure and delivers them as ready-to-deploy changes. No more manual analysis, no guesswork—just proven savings you can implement immediately.

**Your cloud bill doesn't have to be a mystery.** JetScale gives you:
- **Instant visibility** into where money is being wasted
- **AI-validated recommendations** that maintain performance
- **Production-ready changes** integrated into your workflow
- **Measurable ROI** from day one

---

## Quick Start (3 Simple Steps)

### 1. Connect Your Cloud Account
Grant JetScale read-only access to your AWS or Azure environment. We'll automatically discover all your resources—EC2 instances, RDS databases, storage volumes, and more.

### 2. Review Recommendations
Our AI analyzes historical usage patterns and identifies optimization opportunities. Each recommendation includes:
- Estimated monthly savings
- Performance impact assessment
- Before/after configuration comparison
- Implementation risk level

### 3. Deploy Changes
Select recommendations to implement. JetScale generates Terraform code and creates a pull request in your repository. Review, approve, and deploy with confidence.

**Average customers save 30-40% on cloud spend in the first 90 days.**

---

## Why JetScale?

### AI That Understands Your Infrastructure
Unlike generic cost tools, JetScale uses specialized AI agents trained on specific cloud services. Our agents understand the nuances of RDS multi-AZ deployments, EC2 burstable instances, and EBS IOPS requirements.

### Safety-First Approach
Every recommendation is validated to ensure:
- Performance SLAs are maintained
- High-availability configurations are preserved
- Sufficient headroom for traffic spikes
- Easy rollback if needed

### Integrated Into Your Workflow
JetScale works with the tools you already use:
- **GitHub/Bitbucket**: Automated pull requests with detailed documentation
- **Jira**: Automatic ticket creation with savings tracking
- **Terraform**: Production-ready infrastructure code
- **Slack**: Real-time notifications for new opportunities

---

## What JetScale Optimizes

### AWS Services
- **EC2**: Right-size instances, identify idle resources, Graviton migration
- **RDS**: Instance sizing, storage optimization, Reserved Instance recommendations
- **EBS**: Volume type optimization (gp2→gp3), snapshot cleanup
- **ElastiCache**: Node type optimization, cluster configuration
- **Cost Explorer**: Reserved Instances, Savings Plans analysis

### Azure Services
- **Virtual Machines**: Right-sizing, B-series optimization, reserved capacity
- **Azure SQL**: Tier recommendations, elastic pool optimization
- **Managed Disks**: Premium vs Standard optimization
- **Azure Cache for Redis**: Tier and capacity recommendations

### Coming Soon
S3/Blob Storage, Lambda/Functions, DynamoDB/Cosmos DB, ECS/AKS, Load Balancers

---

## Security & Trust

**Your data, your control:**
- Read-only access to cloud accounts (no write permissions)
- Cross-account roles with external ID verification
- SOC 2 Type II compliant infrastructure
- No credential storage—everything uses temporary tokens
- Your code stays in your repositories

**Enterprise-grade security:**
- Data encrypted at rest and in transit (TLS 1.3)
- Regular third-party security audits
- GDPR and CCPA compliant
- Configurable data retention policies

---

## Documentation

### Getting Started
- [Getting Started Guide](getting-started.md) - Complete setup in 5 minutes
- [AWS Account Setup](aws-setup.md) - Connect AWS cloud account
- [Azure Account Setup](azure-setup.md) - Connect Azure cloud account

### How It Works
- [How JetScale Works](how-it-works.md) - Platform overview and workflow
- [AI Analysis](ai-analysis.md) - How our AI validates recommendations
- [Integration Guide](integrations/README.md) - GitHub, Jira, Slack setup

### Reference
- [Supported Services](services/README.md) - Complete list of optimized resources
- [API Documentation](api-reference.md) - Public API for integrations (Beta)
- [FAQ](faq.md) - Frequently asked questions

---

## Support

**Get Help:**
- Email: [support@jetscale.ai](mailto:support@jetscale.ai)
- Community: [GitHub Discussions](https://github.com/jetscale-ai/community/discussions)
- Documentation Issues: [Report Here](https://github.com/jetscale-ai/docs/issues)

**Resources:**
- [Product Updates](https://jetscale.ai/changelog)
- [Case Studies](https://jetscale.ai/customers)
- [Engineering Blog](https://jetscale.ai/blog)

---

## About JetScale

JetScale was founded by cloud infrastructure veterans frustrated by the complexity of cost optimization. We believe every engineering team should have access to enterprise-grade cost intelligence without needing dedicated FinOps specialists.

Our mission: Make cloud cost optimization automatic, accurate, and actionable.

[Learn More About Us](https://jetscale.ai/about) · [We're Hiring](https://jetscale.ai/careers)

---

*© 2025 JetScale, Inc. All rights reserved.*
