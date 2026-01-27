# MCP Integration

JetScale integrates with the Model Context Protocol (MCP) to enhance Terraform code generation with real-time access to provider documentation, community modules, and security scanning tools.

## Overview

The MCP (Model Context Protocol) integration allows JetScale's AI agents to:
- Access up-to-date AWS Terraform provider documentation
- Search AWS-IA (AWS Infrastructure as Code) modules
- Query community Terraform modules
- Execute Checkov security scans on generated code
- Run Terraform validation commands
- Leverage AWS knowledge base for best practices

This integration ensures generated Terraform code follows current best practices, uses recommended modules, and passes security compliance checks.

## What is MCP?

Model Context Protocol (MCP) is an open protocol developed by Anthropic that enables AI models to securely access external tools and data sources. Think of it as a standardized way for AI agents to interact with specialized services.

### Why MCP for Terraform?

Terraform documentation and modules evolve constantly:
- Provider versions update regularly
- New resource types are added
- Best practices change
- Security compliance rules evolve

Without MCP, AI agents would be limited to their training data cutoff. With MCP, JetScale's agents can:
- Access the latest Terraform provider documentation in real-time
- Find current module versions from the Terraform Registry
- Get up-to-date security scan results

## MCP Server

JetScale uses the **AWS Labs Terraform MCP Server** (`awslabs.terraform-mcp-server`) which provides:

### Available Tools

**1. SearchAwsProviderDocs**
- Searches AWS Terraform provider documentation
- Returns resource schemas, attributes, and examples
- Covers `aws` provider (AWS native Terraform provider)

**2. SearchAwsccProviderDocs**
- Searches AWSCC (AWS Cloud Control) provider documentation
- Alternative provider for CloudFormation-style resources
- Useful for resources not yet in AWS provider

**3. SearchSpecificAwsIaModules**
- Searches AWS-IA (AWS Infrastructure as Code) official modules
- Curated, tested modules from AWS
- Includes security best practices

**4. SearchUserProvidedModule**
- Searches any Terraform Registry module by URL
- Supports version constraints
- Retrieves module variables and usage examples

**5. RunCheckovScan**
- Executes Checkov security scanning
- Identifies compliance violations
- Provides remediation suggestions

**6. ExecuteTerraformCommand**
- Runs Terraform CLI commands
- Supports: init, validate, plan, fmt
- Returns formatted output

## How JetScale Uses MCP

### 1. Code Generation Workflow

When generating Terraform code for a recommendation:

```
Recommendation → Adapter → MCP Strategy → MCP Tools → Terraform Code
```

**Step 1: Adapter Analysis**
- JetScale analyzes the cost optimization recommendation
- Identifies AWS resource type (EC2, RDS, EBS, etc.)
- Determines optimal Terraform resource(s) needed

**Step 2: MCP Strategy Selection**
- AI agent selects which MCP tools to call
- Prioritizes based on recommendation complexity
- Optimizes for speed (only calls necessary tools)

**Step 3: MCP Tool Execution**
- Calls selected MCP tools in parallel when possible
- Retrieves latest provider documentation
- Searches for relevant modules
- Gathers security best practices

**Step 4: Code Generation**
- Uses MCP results to generate accurate Terraform
- Applies security best practices from scans
- Includes module recommendations when beneficial
- Validates syntax and structure

### 2. Example MCP Strategy

For an EC2 instance right-sizing recommendation:

```json
{
  "mcp_tools_to_call": [
    "SearchAwsProviderDocs",
    "SearchSpecificAwsIaModules"
  ],
  "reasoning": "Need current aws_instance schema and check for official AWS modules",
  "aws_resource": "aws_instance"
}
```

**MCP Calls:**
1. `SearchAwsProviderDocs(asset_name="aws_instance")`
   - Returns: Complete `aws_instance` resource schema
   - Latest attributes and configuration options

2. `SearchSpecificAwsIaModules(query="ec2")`
   - Returns: AWS-IA EC2 modules
   - Recommendation: `terraform-aws-modules/ec2-instance/aws`

**Generated Code:**
```hcl
# JetScale can choose between raw resource or module based on complexity

# Option 1: Raw resource (simpler cases)
resource "aws_instance" "web_server" {
  instance_type = "t3.medium"
  ami           = var.ami_id
  # ... attributes from MCP provider docs
}

# Option 2: AWS-IA module (complex cases)
module "ec2_instance" {
  source  = "terraform-aws-modules/ec2-instance/aws"
  version = "~> 5.0"

  instance_type = "t3.medium"
  # ... using module variables from MCP
}
```

### 3. Security Scanning Integration

After generating Terraform code, JetScale can optionally run Checkov:

```python
# MCP Checkov Scan
result = await mcp.run_checkov_scan(
    working_directory="/tmp/terraform",
    framework="terraform",
    output_format="json"
)
```

**Scan Results:**
- Critical/High/Medium/Low severity issues
- CIS AWS Foundations Benchmark compliance
- Terraform best practices violations
- Remediation suggestions

If security issues are found, JetScale automatically:
1. Logs the violations
2. Optionally regenerates code with fixes
3. Includes security notes in pull request description

### 4. Knowledge Base Search

For complex optimizations, JetScale searches AWS knowledge base:

```python
# Search for specific best practices
kb_result = await mcp.search_aws_knowledge_base(
    query="rds instance class right-sizing",
    limit=3
)
```

Returns:
- AWS best practice articles
- Well-Architected Framework guidance
- Performance optimization patterns
- Cost optimization strategies

## Resource Mapping

JetScale maintains mappings from AWS service types to Terraform resources:

| JetScale Resource | AWS Provider | AWSCC Provider |
|-------------------|--------------|----------------|
| Ec2Instance | aws_instance | awscc_ec2_instance |
| RdsInstance | aws_db_instance | awscc_rds_db_instance |
| ElastiCacheCluster | aws_elasticache_cluster | awscc_elasticache_cache_cluster |
| EbsVolume | aws_ebs_volume | awscc_ec2_volume |
| S3Bucket | aws_s3_bucket | awscc_s3_bucket |
| LambdaFunction | aws_lambda_function | awscc_lambda_function |
| AutoScalingGroup | aws_autoscaling_group | awscc_autoscaling_auto_scaling_group |

### Module Suggestions

For common resources, JetScale suggests well-maintained community modules:

| Resource Type | Suggested Module |
|---------------|------------------|
| Ec2Instance | terraform-aws-modules/ec2-instance/aws |
| RdsInstance | terraform-aws-modules/rds/aws |
| LambdaFunction | terraform-aws-modules/lambda/aws |
| AutoScalingGroup | terraform-aws-modules/autoscaling/aws |
| S3Bucket | terraform-aws-modules/s3-bucket/aws |

## Benefits of MCP Integration

### 1. Always Up-to-Date

**Without MCP:**
- AI model trained on Terraform docs from training cutoff
- Misses new resource attributes
- May use deprecated syntax
- Outdated module versions

**With MCP:**
- Real-time access to current provider documentation
- Latest resource schemas and attributes
- Current module versions from registry
- Up-to-date best practices

### 2. Better Code Quality

**MCP-Enhanced Code Includes:**
- All required attributes (no missing fields)
- Optional but recommended attributes
- Proper syntax for current provider version
- Security best practices from Checkov
- Well-Architected Framework patterns

### 3. Module Recommendations

**Intelligent Module Selection:**
- Simple resources: Raw Terraform resources
- Complex resources: AWS-IA or community modules
- Trade-off: Simplicity vs. best practices

**Example Decision:**
```
Simple EC2 instance → aws_instance resource
Complex VPC setup → terraform-aws-modules/vpc/aws
```

### 4. Security Compliance

**Automated Security Checks:**
- CIS AWS Foundations Benchmark
- AWS Security Best Practices
- Terraform-specific security patterns
- Encryption requirements
- IAM permission boundaries

**Violations Detected:**
- Unencrypted storage
- Overly permissive security groups
- Missing logging/monitoring
- Non-compliant tagging

## Configuration

### MCP Server Installation

JetScale requires the AWS Labs Terraform MCP Server to be installed:

```bash
# Install via npm (requires Node.js)
npm install -g @awslabs/terraform-mcp-server

# Or via pip (Python package)
pip install awslabs-terraform-mcp-server

# Verify installation
awslabs.terraform-mcp-server --version
```

### Environment Variables

The MCP client uses these environment variables:

```bash
# Set log level (ERROR, WARN, INFO, DEBUG)
export FASTMCP_LOG_LEVEL=ERROR

# Optional: AWS credentials for Terraform commands
export AWS_REGION=us-east-1
export AWS_ACCESS_KEY_ID=your_key
export AWS_SECRET_ACCESS_KEY=your_secret
```

### JetScale Configuration

MCP integration is enabled by default for Terraform code generation. No additional configuration needed.

## Limitations

### Current Limitations

1. **Provider Support**
   - Currently supports AWS provider only
   - AWSCC provider support (CloudFormation-based)
   - Other providers (Azure, GCP) not yet supported

2. **Module Scope**
   - Searches public Terraform Registry modules
   - Private registry modules require additional configuration
   - Custom internal modules not automatically discovered

3. **Security Scanning**
   - Checkov scans Terraform code only
   - Does not scan deployed infrastructure
   - Requires Checkov installation on server

4. **Performance**
   - MCP calls add latency to code generation
   - Typical overhead: 2-5 seconds per recommendation
   - Parallel calls optimize when possible

### Future Enhancements

**Planned Features:**
- Multi-cloud provider support (Azure, GCP)
- Private module registry integration
- Real-time infrastructure scanning
- Custom policy enforcement
- Terraform Cloud integration

## Troubleshooting

### MCP Server Connection Issues

**Problem:** "Failed to connect to MCP server"

**Solutions:**
1. Verify MCP server is installed:
   ```bash
   which awslabs.terraform-mcp-server
   ```

2. Check server can start:
   ```bash
   awslabs.terraform-mcp-server --help
   ```

3. Review environment variables:
   ```bash
   echo $FASTMCP_LOG_LEVEL
   ```

4. Check logs for connection errors:
   ```bash
   tail -f /var/log/jetscale/mcp.log
   ```

### Tool Execution Failures

**Problem:** "MCP tool returned error"

**Solutions:**
- Check internet connectivity (MCP queries Terraform Registry)
- Verify AWS credentials for Terraform commands
- Ensure working directory has write permissions
- Review specific tool error messages

### Checkov Scan Failures

**Problem:** "Checkov scan failed"

**Solutions:**
1. Verify Checkov is installed:
   ```bash
   checkov --version
   ```

2. Check working directory permissions:
   ```bash
   ls -la /tmp/terraform
   ```

3. Verify Terraform files are valid:
   ```bash
   terraform validate
   ```

### Module Search Returns No Results

**Problem:** "No modules found for query"

**Possible Causes:**
- Terraform Registry is down (check status.hashicorp.com)
- Module name misspelled
- Module is private (not in public registry)
- Network connectivity issues

**Solutions:**
- Verify query string
- Try broader search terms
- Check Terraform Registry directly
- Fall back to raw resources if modules unavailable

## Best Practices

### 1. Trust MCP Results

MCP provides authoritative, up-to-date information:
- Always use latest provider docs from MCP
- Prefer MCP results over cached/static documentation
- Trust module version recommendations

### 2. Balance Modules vs. Resources

**Use Modules When:**
- Complex resource configurations
- Multiple related resources needed
- AWS best practices are critical
- Team wants standardized patterns

**Use Raw Resources When:**
- Simple, single resource changes
- Minimal configuration needed
- Team prefers explicit code
- Module adds unnecessary complexity

### 3. Security First

Always enable security scanning:
- Run Checkov on generated code
- Review violations before creating PR
- Fix critical/high severity issues
- Document accepted risks for medium/low

### 4. Monitor MCP Performance

Track MCP call performance:
- Log MCP response times
- Identify slow tools
- Optimize tool selection strategy
- Cache results when appropriate

## API Integration

MCP integration is transparent to the JetScale API. When you request Terraform code generation:

```bash
# Generate Terraform for recommendation
POST /api/v1/recommendations/{recommendation_id}/terraform

# Response includes MCP-enhanced code
{
  "terraform_code": "...",
  "mcp_tools_used": ["SearchAwsProviderDocs", "SearchSpecificAwsIaModules"],
  "security_scan_results": {...},
  "module_recommendations": [...]
}
```

See our [API Documentation](../api-reference.md) for complete reference.

## Support

Need help with MCP integration?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: [FAQ](../faq.md)
- **MCP Protocol**: [Anthropic MCP Docs](https://github.com/anthropics/mcp)
- **AWS Labs MCP Server**: [GitHub](https://github.com/awslabs/terraform-mcp-server)

---

**Related Documentation:**
- [GitHub Integration](github.md)
- [Jira Integration](jira.md)
- [Bitbucket Integration](bitbucket.md)
