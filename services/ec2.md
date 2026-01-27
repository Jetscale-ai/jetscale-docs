# EC2 Optimization

JetScale provides AI-powered cost optimization for Amazon EC2 (Elastic Compute Cloud) standalone instances. Our specialized agents analyze your compute workloads to identify right-sizing opportunities, instance family migrations, and cost-effective alternatives.

## Overview

JetScale optimizes EC2 resources by analyzing:
- **Instance utilization**: CPU and network usage patterns
- **Memory metrics**: When CloudWatch agent is installed
- **Cost analysis**: Current spending vs. optimal configuration
- **Workload characteristics**: CPU patterns, resource balance needs

## Supported EC2 Types

### Standalone EC2 Instances

JetScale optimizes standalone EC2 instances that are not part of Auto Scaling Groups.

**Instance Architecture:**
```
Ec2Instance (Optimization Target, Billable)
├── EbsVolume - Billable (optimized separately)
└── EbsVolume - Billable
```

**What We Optimize:**
- **Instance type right-sizing**: Change to smaller/larger instances based on actual usage
- **Instance family migration**: Switch between families (T, M, C, R, X, etc.) based on workload characteristics
- **Generation upgrades**: Migrate to latest generation (M8, C8, R8) for better price/performance
- **Graviton migration**: Switch to ARM-based instances (g suffix) for 10-40% cost savings
- **Delete unused**: Remove instances with zero or near-zero activity (100% savings)

**Important:** JetScale currently optimizes standalone EC2 instances only. Instances managed by Auto Scaling Groups, EKS, ECS, or other orchestration systems are not yet supported.

## Instance Families

JetScale considers all AWS EC2 instance families when optimizing:

### General Purpose (M)
**M8i/M8g/M8a** - Balanced compute/memory/network
- Use for: Web servers, application servers, small databases, mixed workloads
- Latest generations: M8 > M7 > M6 > M5

### Compute Optimized (C)
**C8i/C8g, C7a** - High CPU ratio
- Use for: Batch processing, HPC, media transcoding, gaming, high-traffic web servers
- Best when: CPU utilization is consistently high, memory needs are moderate

### Memory Optimized (R)
**R8i/R8g, R7a** - High memory ratio
- Use for: Databases, in-memory caches, big data analytics
- Best when: Memory utilization is high, CPU is moderate

### Extreme Memory (X)
**X8g, X2idn/X2iedn** - Very high memory ratios
- Use for: Large databases, in-memory analytics, SAP HANA
- Cost premium: Significantly higher than R-family

### Burstable (T)
**T4g, T3, T3a** - Baseline + burst CPU credits
- Use for: Dev/test, low-traffic web servers, microservices, variable workloads
- Best when: CPU usage is low most of the time with occasional spikes
- Caution: Monitor CPU credit balance to avoid throttling

### Storage Optimized

**I-series (NVMe SSD)**: I8g/I8ge, I7i/I7ie - Low-latency local storage
- Use for: Transactional databases (MySQL, MongoDB, Cassandra)

**D-series (HDD)**: D3/D3en - High-density HDD storage
- Use for: Distributed file systems (HDFS), data warehousing

**H-series (HDD)**: H1 - Sequential HDD throughput
- Use for: Big data, MapReduce, log processing

### Accelerated Computing

**GPU and ML accelerators** - P6, G6, Inf2, Trn2
- Use for: Machine learning training/inference, graphics rendering
- Not typically recommended for cost optimization (specialized workloads)

### Processor Types

- **Intel** (default): Most common, broad compatibility
- **AMD** (a suffix): Cost-effective alternative with good performance
- **Graviton** (g suffix): ARM64 architecture, best price/performance ratio (10-40% savings)

### Instance Sizing

Sizes follow pattern: `.large < .xlarge < .2xlarge < .4xlarge < .8xlarge < .12xlarge < .16xlarge < .24xlarge`

Each step typically doubles vCPU and memory.

## How JetScale Optimizes EC2

### 1. Data Collection

JetScale analyzes multiple data sources:

**CloudWatch Metrics** (14-day rolling window):
- `CPUUtilization` - Instance CPU usage (always available)
- `NetworkIn` / `NetworkOut` - Network throughput (always available)
- `mem_used_percent` - Memory usage (requires CloudWatch agent)
- `disk_used_percent` - Disk usage (requires CloudWatch agent)

**EC2 API Data**:
- Instance type and state
- Launch time
- Availability zone
- Tags

**Cost Explorer Data**:
- Current monthly spend per instance
- Historical cost trends

**AWS Compute Optimizer** (if enabled):
- AWS-generated right-sizing recommendations
- Performance risk assessments

### 2. Analysis

Our AI agents perform deep analysis:

**Utilization Patterns:**
- Peak vs. average CPU utilization
- Time-of-day patterns (identify idle periods)
- Day-of-week patterns (weekend vs. weekday load)
- CPU credit balance (for T-series instances)

**Memory Analysis** (if metrics available):
- Memory usage percentage
- Headroom above peak usage
- Minimum required memory with buffer

**Workload Classification:**
- **Variable CPU**: Consider T-series (burstable)
- **Steady high CPU**: Consider C-series (compute optimized)
- **Balanced**: Consider M-series (general purpose)
- **Memory-intensive**: Consider R-series (memory optimized)

**Cost Modeling:**
- Current monthly cost
- Projected cost for alternative instance types
- Savings percentage and annual impact

**Risk Evaluation:**
- Performance degradation probability
- Headroom calculation (buffer above peak usage)
- Memory safety checks

### 3. Recommendations

JetScale generates specific, actionable recommendations:

#### Instance Right-Sizing

**Example Recommendation:**
```
Resource: web-server-prod-1
Current: m5.2xlarge (8 vCPUs, 32 GB RAM)
Recommended: m5.xlarge (4 vCPUs, 16 GB RAM)

Cost Impact:
- Current: $280/month
- Projected: $140/month
- Savings: $140/month (50%), $1,680/year

Performance Analysis:
- Average CPU: 18%
- Peak CPU: 32%
- Average Memory: 28% (requires CloudWatch agent for memory metrics)
- Recommended provides 2x headroom above peak

Risk: Low - Ample headroom maintained
```

#### Family Migration

**Example Recommendation:**
```
Resource: batch-processor-2
Current: m5.xlarge (4 vCPUs, 16 GB RAM)
Recommended: c6i.xlarge (4 vCPUs, 8 GB RAM)

Cost Impact:
- Current: $140/month
- Projected: $124/month
- Savings: $16/month (11%), $192/year

Workload Analysis:
- Average CPU: 75%
- Peak CPU: 92%
- Memory usage: Low (25%)
- Workload type: Compute-intensive, memory underutilized

Recommendation: Migrate to C-family (compute optimized)
Risk: Very Low - CPU-bound workload, memory headroom adequate
```

#### Graviton Migration

**Example Recommendation:**
```
Resource: api-server-1
Current: m5.large (2 vCPUs, 8 GB RAM, Intel-based)
Recommended: m6g.large (2 vCPUs, 8 GB RAM, Graviton2-based)

Cost Impact:
- Current: $70/month
- Projected: $56/month
- Savings: $14/month (20%), $168/year

Performance Analysis:
- Graviton2 offers equivalent or better performance
- 20% lower cost per instance
- Same vCPU and memory configuration
- Requires ARM64 compatible AMI

Risk: Very Low - Proven Graviton performance
Note: Verify application compatibility with ARM64 architecture
```

#### Generation Upgrade

**Example Recommendation:**
```
Resource: database-app-server
Current: m5.xlarge (4 vCPUs, 16 GB RAM)
Recommended: m7i.xlarge (4 vCPUs, 16 GB RAM)

Cost Impact:
- Current: $140/month
- Projected: $133/month
- Savings: $7/month (5%), $84/year

Performance Analysis:
- M7i offers better CPU performance (4th gen Intel Xeon)
- Same specifications, lower cost
- Better network performance (up to 12.5 Gbps)
- Newer generation efficiency

Risk: Very Low - Same family, newer generation
```

#### Delete Unused Instance

**Example Recommendation:**
```
Resource: legacy-test-server
Current: t3.medium (2 vCPUs, 4 GB RAM)
Recommended: Delete/Terminate

Cost Impact:
- Current: $30/month
- Projected: $0/month
- Savings: $30/month (100%), $360/year

Usage Analysis:
- Average CPU: 0.2%
- Network activity: Near zero
- Launched: 18 months ago
- Tags indicate: test environment

Risk: Very Low - Appears unused
Recommendation: Verify with application team before deletion
```

### 4. Terraform Generation

For each recommendation, JetScale generates production-ready Terraform code:

**Example: Instance Right-Sizing**

```hcl
# EC2 Instance Optimization
# Generated by JetScale on 2024-01-15
# Recommendation ID: rec_ec2_001

resource "aws_instance" "web_server_prod_1" {
  # Previous: m5.2xlarge ($280/month)
  # Optimized: m5.xlarge ($140/month)
  # Cost Reduction: 50% ($140/month, $1,680/year)
  #
  # Justification:
  # - Average CPU utilization: 18%
  # - Peak CPU utilization: 32%
  # - Average memory utilization: 28%
  # - New instance provides 2x headroom above peak usage
  instance_type = "m5.xlarge"

  ami           = var.ami_id
  key_name      = var.key_name
  subnet_id     = var.subnet_id

  vpc_security_group_ids = var.security_group_ids

  # Existing configuration preserved
  monitoring    = true
  ebs_optimized = true

  root_block_device {
    volume_type           = "gp3"
    volume_size           = 100
    encrypted             = true
    delete_on_termination = true
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"       = "true"
      "jetscale:recommendation"  = "rec_ec2_001"
      "jetscale:previous_type"   = "m5.2xlarge"
    }
  )

  lifecycle {
    create_before_destroy = true
  }
}
```

**Example: Graviton Migration**

```hcl
# EC2 Graviton Migration
# Generated by JetScale on 2024-01-15

resource "aws_instance" "api_server_1" {
  # Previous: m5.large (Intel, $70/month)
  # Optimized: m6g.large (Graviton2, $56/month)
  # Cost Reduction: 20% ($14/month, $168/year)
  #
  # Requirements:
  # - ARM64-compatible AMI required
  # - Verify application compatibility
  # - Test in staging before production
  instance_type = "m6g.large"

  # IMPORTANT: Use ARM64 AMI
  # Replace with your ARM64-compatible AMI ID
  ami = var.ami_id_arm64

  key_name  = var.key_name
  subnet_id = var.subnet_id

  vpc_security_group_ids = var.security_group_ids

  monitoring    = true
  ebs_optimized = true

  root_block_device {
    volume_type           = "gp3"
    volume_size           = 50
    encrypted             = true
    delete_on_termination = true
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"    = "true"
      "jetscale:architecture" = "arm64"
      "jetscale:processor"    = "graviton2"
    }
  )

  lifecycle {
    create_before_destroy = true
  }
}
```

## Memory Metrics Handling

### With CloudWatch Agent

When CloudWatch agent is installed and reporting memory metrics:
- JetScale can safely recommend instances with less memory
- Memory sizing based on peak usage + 20% headroom (AWS Compute Optimizer standard)
- More aggressive optimization possible

**Example:**
```
Current: 32 GB RAM, Peak usage: 20 GB (62.5%)
Minimum safe RAM: 20 GB × 1.20 = 24 GB
Recommendation: Instance with 32 GB RAM or less (if available)
```

### Without CloudWatch Agent

When memory metrics are unavailable:
- **Safety constraint**: All alternatives must have EXACTLY the same amount of memory
- Cannot recommend instances with less memory (risk of OOM crashes)
- Optimization limited to CPU, network, and cost improvements

**Example:**
```
Current: 32 GB RAM, no memory metrics
Constraint: All alternatives MUST have exactly 32 GB RAM
Optimization: Instance family, generation, processor type only
```

**To enable memory metrics:**
1. Install CloudWatch agent on EC2 instances
2. Configure agent to collect `mem_used_percent` metric
3. Wait 14 days for sufficient historical data
4. JetScale will automatically use memory metrics in recommendations

## Best Practices

### Testing Strategy

**Pre-Production Testing:**
1. Apply changes to dev/staging environment first
2. Run load tests simulating peak traffic
3. Monitor for 48-72 hours under realistic load
4. Validate application performance

**Production Rollout:**
1. Use `create_before_destroy` lifecycle in Terraform
2. Schedule changes during maintenance windows
3. Have rollback plan ready (previous Terraform state)
4. Monitor actively during and after change

### Graviton Migration

**Before migrating to Graviton:**
1. Verify application is ARM64 compatible (most modern apps are)
2. Check dependencies and libraries for ARM support
3. Test with ARM64 AMI in staging
4. Review AWS Graviton compatibility documentation
5. Common compatible: Node.js, Python, Java, Go, .NET, Ruby, PHP

**Incompatible workloads:**
- x86-only binaries or libraries
- Legacy applications without ARM builds
- Specific software requiring Intel/AMD instructions

### Reserved Instances & Savings Plans

While JetScale doesn't currently generate RI/Savings Plan recommendations, consider them after right-sizing:

**Best practice:**
1. Right-size instances with JetScale first
2. Run optimized configuration for 30-60 days
3. Purchase RIs or Savings Plans for stable workloads
4. Never reserve over-sized instances

## Common Optimization Patterns

### Pattern 1: Over-Provisioned Instances

**Symptoms:**
- CPU utilization < 20% average
- Memory utilization < 30% (if metrics available)
- Instance running 24/7 with consistent low usage

**JetScale Recommendation:**
- Downsize by 1-2 instance sizes (e.g., 2xlarge → xlarge)
- Typical savings: 40-50%
- Risk: Low (2-3x headroom maintained)

### Pattern 2: Wrong Instance Family

**Symptoms:**
- Using M-family with consistently high CPU (>70%)
- Using R-family with low memory usage (<40%)
- Using C-family with high memory needs

**JetScale Recommendation:**
- Migrate to appropriate family (M → C for CPU, R → M for balanced)
- Typical savings: 10-30%
- Risk: Low to Medium (verify workload characteristics)

### Pattern 3: Legacy Generations

**Symptoms:**
- Using M4, M5, C4, C5 instances
- Newer generations available (M6, M7, M8)
- Same cost or lower for better performance

**JetScale Recommendation:**
- Upgrade to latest generation (M5 → M7, C5 → C7)
- Typical savings: 5-15% with better performance
- Risk: Very Low (same family, newer hardware)

### Pattern 4: Missed Graviton Opportunities

**Symptoms:**
- Using Intel-based instances (m5, c5, r5)
- ARM64-compatible applications
- No specialized x86 requirements

**JetScale Recommendation:**
- Migrate to Graviton (m5 → m6g, c5 → c6g, r5 → r6g)
- Typical savings: 10-40% with same or better performance
- Risk: Low (verify ARM64 compatibility)

## Troubleshooting

### Recommendation Concerns

**Q: Will downsizing my instance impact performance?**

A: JetScale maintains 2-3x headroom above peak usage. We analyze:
- 99th percentile (P99) CPU utilization over 14 days
- Peak memory usage (if available)
- Network saturation points
- Historical spike patterns

Recommendations only proceed if performance risk is Low or Very Low.

**Q: What if I don't have memory metrics?**

A: JetScale applies a safety constraint: all alternatives must have EXACTLY the same amount of memory as the current instance. This prevents under-provisioning risks. To unlock more aggressive memory optimization, install CloudWatch agent.

**Q: Can I test Graviton without downtime?**

A: Yes! Recommended approach:
1. Launch new Graviton instance with `create_before_destroy = true`
2. Health check passes → traffic shifts to new instance
3. Old instance automatically terminated
4. Total downtime: typically <30 seconds during cutover

### Performance Issues After Optimization

**Symptom: Increased latency or response times**

Possible causes:
- Insufficient CPU for peak loads
- Memory pressure (if recommended size too small)
- Network throughput bottleneck

**Resolution:**
1. Check CloudWatch CPU, memory, network metrics
2. Compare current vs. previous instance performance
3. If needed, increase instance size by one step
4. Simple rollback: revert Terraform to previous state

**Symptom: T-series CPU credit exhaustion**

Possible causes:
- Migrated from fixed-performance instance to burstable
- CPU usage higher than T-series baseline
- Insufficient CPU credits for workload pattern

**Resolution:**
1. Check `CPUCreditBalance` metric
2. If consistently depleted, migrate to M/C-series
3. Consider T3 Unlimited mode (additional cost for burst)

## Security Considerations

### Instance Configuration

JetScale recommendations preserve:
- VPC and security group associations
- IAM instance profiles
- Key pairs
- EBS encryption settings
- Monitoring configuration

### AMI Compatibility

When migrating to Graviton (ARM64):
- Use ARM64-compatible AMI
- Verify all software supports ARM architecture
- Test thoroughly in non-production first

### Lifecycle Management

Use Terraform lifecycle rules:
```hcl
lifecycle {
  create_before_destroy = true
  prevent_destroy = var.is_production
}
```

## Limitations

**Not Currently Supported:**
- Auto Scaling Group optimization
- Spot instance recommendations
- Reserved Instance purchase recommendations
- Savings Plan recommendations
- Scheduled start/stop automation
- Multi-instance orchestration (EKS, ECS-managed instances)

**Roadmap:**
These features are planned for future releases. Currently, JetScale focuses on standalone EC2 instance optimization where immediate cost savings can be achieved through right-sizing and migration.

## API Integration

JetScale provides API access for programmatic optimization:

```bash
# List EC2 recommendations
GET /api/v1/recommendations?resource_type=ec2

# Get specific recommendation details
GET /api/v1/recommendations/{recommendation_id}

# Approve recommendation (generates Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Retrieve generated Terraform
GET /api/v1/recommendations/{recommendation_id}/terraform
```

See our [API Documentation](../api-reference.md) for complete reference.

## Support

Need help with EC2 optimization?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: [FAQ](../faq.md)
- **GitHub Issues**: [Report a problem](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

**Related Documentation:**
- [EBS Optimization](ebs.md)
- [RDS Optimization](rds.md)
- [ElastiCache Optimization](elasticache.md)
