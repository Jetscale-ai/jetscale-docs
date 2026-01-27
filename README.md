# JetScale Documentation

Welcome to JetScale - AI-powered cloud cost optimization platform that automatically analyzes your AWS infrastructure and generates actionable recommendations.

## What is JetScale?

JetScale uses specialized AI agents to analyze your cloud resources, identify optimization opportunities, and generate infrastructure-as-code changes that you can review and deploy through your existing GitOps workflows.

## Supported AWS Services

### Database Services (RDS)
- **RDS Standalone Instances**: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server
- **Aurora Clusters**: Multi-instance cluster optimization including reader/writer topology analysis
- **Optimization Focus**: Right-sizing based on CPU, memory, IOPS metrics; instance class recommendations

### Compute Services (EC2)
- **EC2 Instances**: All instance types and families
- **Optimization Focus**: Instance type recommendations based on utilization patterns, cost-performance analysis

### Storage Services
- **EBS Volumes**: gp2, gp3, io1, io2, magnetic volumes
- **Optimization Focus**: Volume type optimization, IOPS provisioning, size right-sizing
- **ElastiCache**: Redis and Memcached optimization

## How It Works

1. **Connect Your AWS Account**: Securely connect via IAM role with read-only permissions
2. **AI Analysis**: Multi-agent system analyzes resource utilization, pricing, and usage patterns
3. **Generate Recommendations**: AI agents create optimized configurations with:
   - Cost impact analysis
   - Performance validation
   - Terraform/IaC code generation
4. **Review & Deploy**: Integrate with GitHub, Jira, or Bitbucket for approval workflows
5. **Track Savings**: Monitor implemented optimizations and realized cost savings

## Key Features

- **AI-Powered Recommendations**: LangGraph-based multi-agent system with specialized expertise per service type
- **Data Enrichment**: Real-time CloudWatch metrics, Cost Explorer data, and AWS Pricing API integration
- **Validation Layer**: Automated checks ensure recommendations maintain performance and availability
- **GitOps Integration**: Native GitHub, Jira, and Bitbucket integration for change management
- **Real-Time Updates**: WebSocket-based live progress tracking during analysis
- **Terraform Generation**: Production-ready infrastructure-as-code for approved changes

## Quick Links

- [Getting Started](getting-started.md)
- [Architecture Overview](architecture.md)
- [API Reference](api-reference.md)
- [Deployment Guide](deployment.md)

## Support

For questions or issues, visit [github.com/Jetscale-ai](https://github.com/Jetscale-ai)
