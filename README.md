# Overview

JetScale is an AI-powered cloud cost optimization platform that automatically analyzes your infrastructure and generates actionable recommendations with production-ready infrastructure-as-code.

---

## Getting Started

<div class="card-grid">

<div class="doc-card">

**[AWS Setup](aws-setup.md)**

Connect your AWS account securely using IAM roles with read-only permissions

</div>

<div class="doc-card">

**[Azure Setup](azure-setup.md)**

Connect Azure using service principal authentication with certificate or secret-based auth

</div>

<div class="doc-card">

**[Quick Start Guide](getting-started.md)**

Get up and running in minutes with our step-by-step onboarding guide

</div>

</div>

---

## What is JetScale?

JetScale uses specialized AI agents to analyze your cloud infrastructure, identify cost optimization opportunities, and generate production-ready infrastructure-as-code changes integrated into your existing workflows.

**Key Capabilities:**
- AI agents trained on specific cloud services (RDS, EC2, EBS, ElastiCache, Azure VMs)
- Real-time data analysis from CloudWatch, Cost Explorer, and Azure Monitor
- Automated validation ensuring recommendations maintain performance and SLAs
- GitOps-native integration with GitHub, Jira, and Bitbucket
- Production-ready Terraform code generation

---

## Supported Services

<div class="card-grid">

<div class="doc-card">

**Databases**

AWS RDS - MySQL, PostgreSQL, Aurora clusters
Azure SQL - Azure SQL, MySQL, PostgreSQL
Right-sizing, instance class selection, topology optimization

</div>

<div class="doc-card">

**Compute**

AWS EC2 - All instance types and families
Azure VMs - All VM series
Instance type recommendations, reservation opportunities

</div>

<div class="doc-card">

**Storage**

AWS EBS - gp2, gp3, io1, io2 volumes
Azure Storage - Blob, Files, Managed Disks
Volume type optimization, tier optimization, IOPS provisioning

</div>

<div class="doc-card">

**Caching**

AWS ElastiCache - Redis, Memcached
Azure Cache - Redis, Redis Enterprise
Node type optimization, cluster configuration

</div>

</div>

> **Coming Soon:** S3, Lambda, DynamoDB, Azure Cosmos DB, Azure Functions

---

## Documentation

<div class="card-grid">

<div class="doc-card">

**Core Concepts**

[Architecture Overview](architecture.md) - System design and workflows
[AI Agents](ai-agents.md) - How specialized agents work
[Recommendation Workflow](recommendation-workflow.md) - End-to-end process

</div>

<div class="doc-card">

**Integrations**

[GitHub Integration](integrations/github.md) - PR creation and tracking
[Jira Integration](integrations/jira.md) - Issue management
[Bitbucket Integration](integrations/bitbucket.md) - Repository integration

</div>

<div class="doc-card">

**Service Guides**

[RDS Optimization](services/rds.md) - Database optimization
[EC2 Optimization](services/ec2.md) - Compute optimization
[EBS Optimization](services/ebs.md) - Storage optimization
[ElastiCache Optimization](services/elasticache.md) - Cache optimization

</div>

<div class="doc-card">

**Reference**

[API Reference](api-reference.md) - REST API documentation
[Configuration](configuration.md) - Platform configuration
[Deployment Guide](deployment.md) - Self-hosted deployment

</div>

</div>

---

## Support

**Need help?**
[support@jetscale.ai](mailto:support@jetscale.ai)
[GitHub Discussions](https://github.com/Jetscale-ai/jetscale-docs/discussions)
[Report an Issue](https://github.com/Jetscale-ai/jetscale-docs/issues)

**Resources:**
[GitHub Organization](https://github.com/Jetscale-ai) · [Change Log](https://github.com/Jetscale-ai/jetscale-docs/releases)
