# RDS Optimization

JetScale provides AI-powered cost optimization for Amazon RDS (Relational Database Service) including both standalone instances and Aurora clusters. Our specialized agents analyze your database workloads to identify right-sizing opportunities and configuration improvements.

## Overview

JetScale optimizes RDS resources by analyzing:
- **Instance utilization**: CPU, memory, network, and connection patterns
- **Cost analysis**: Current spending vs. optimal configuration
- **Performance metrics**: Query performance, replication lag, buffer cache hit ratios
- **High availability**: Multi-AZ configurations for standalone instances

## Supported RDS Types

### Aurora Clusters

Aurora clusters are our **primary optimization target**. We analyze the cluster as a whole rather than individual instances.

**Cluster Architecture:**
```
RdsDbCluster (Optimization Target)
├── RdsDbInstance (Writer) - Billable
├── RdsDbInstance (Reader) - Billable
├── RdsDbInstance (Reader) - Billable
└── AuroraDbClusterStorage - Billable (I/O + Storage)
```

**What We Optimize:**
- **Instance class**: Right-size all cluster nodes to match actual workload (all nodes use same instance class)
- **Graviton migration**: Switch to ARM-based instances (r6g, r7g, m6g, m7g) for 10-40% savings
- **Storage configuration**: Aurora Standard vs. I/O-Optimized based on I/O patterns

**Storage Configurations:**
- **Aurora Standard** (`aurora`): Lower compute cost, $0.20/million I/O charges
- **Aurora I/O-Optimized** (`aurora-iopt1`): Higher compute cost (+20%), zero I/O charges

**Important:** All instances in an Aurora cluster must use the same instance class. JetScale sizes for the node with the highest resource usage to ensure adequate performance across all cluster members.

### Standalone RDS Instances

Traditional RDS instances (MySQL, PostgreSQL, MariaDB, SQL Server, Oracle) are optimized individually.

**Instance Architecture:**
```
RdsDbInstance (Optimization Target, Billable)
└── RdsDbInstanceStorage - Billable
```

**What We Optimize:**
- **Instance class**: Right-size to appropriate family (t3, m5, r5, r6g, etc.)
- **Graviton migration**: Switch to ARM-based instances (r6g, r7g, m6g, m7g) for 10-40% savings
- **Multi-AZ configuration**: Disable Multi-AZ for ~50% savings when high availability isn't critical (reduces availability)

**Billing Model:**
- Per-second billing with 10-minute minimum
- Reserved Instances: Up to 66% savings (1-year or 3-year commitment)
- Database Savings Plans: Additional flexibility across instance families

## How JetScale Optimizes RDS

### 1. Data Collection

JetScale analyzes multiple data sources:

**CloudWatch Metrics** (14-day rolling window):
- `CPUUtilization` - Instance CPU usage
- `FreeableMemory` - Available memory
- `DatabaseConnections` - Active connections
- `ReadLatency` / `WriteLatency` - Storage performance
- `ReadThroughput` / `WriteThroughput` - Disk I/O
- `NetworkReceiveThroughput` / `NetworkTransmitThroughput`
- `AuroraBinlogReplicaLag` (Aurora only)

**RDS API Data**:
- Instance class and engine version
- Storage type (Aurora Standard vs I/O-Optimized)
- Multi-AZ status (standalone instances only)
- Cluster members and roles (Aurora only)

**Cost Explorer Data**:
- Current monthly spend per instance/cluster
- Historical cost trends
- Reserved Instance utilization

**AWS Compute Optimizer** (if enabled):
- AWS-generated right-sizing recommendations
- Performance risk assessments

### 2. Analysis

Our AI agents perform deep analysis:

**Utilization Patterns:**
- Peak vs. average utilization across all metrics
- Time-of-day patterns (identify idle periods)
- Day-of-week patterns (weekend vs. weekday load)
- Growth trends over time

**Performance Assessment:**
- Buffer cache hit ratio (indicates memory adequacy)
- Storage throughput vs. provisioned IOPS
- Network throughput vs. instance limits
- Replication lag patterns

**Cost Modeling:**
- Current cost breakdown (compute, storage, I/O, backup)
- Projected cost for alternative configurations
- Reserved Instance / Savings Plan opportunities
- Multi-AZ premium analysis

**Risk Evaluation:**
- Headroom calculation (buffer above peak usage)
- Performance degradation probability
- Availability impact assessment

### 3. Recommendations

JetScale generates specific, actionable recommendations:

#### Instance Right-Sizing

**Example Recommendation:**
```
Resource: production-postgres-db
Current: db.r5.2xlarge (8 vCPUs, 64 GB RAM)
Recommended: db.r5.xlarge (4 vCPUs, 32 GB RAM)

Cost Impact:
- Current: $730/month
- Projected: $365/month
- Savings: $365/month (50%), $4,380/year

Performance Analysis:
- Average CPU: 15-25%
- Peak CPU: 35%
- Average Memory: 30-40%
- Recommended provides 2x headroom above peak

Risk: Low - Ample headroom maintained
```

#### Multi-AZ Optimization

**Example Recommendation:**
```
Resource: staging-postgres-db
Current: db.m5.large with Multi-AZ enabled
Recommended: db.m5.large with Multi-AZ disabled

Cost Impact:
- Current: $280/month (Multi-AZ premium included)
- Projected: $140/month (Single-AZ)
- Savings: $140/month (50%), $1,680/year

Availability Analysis:
- Environment: Staging/Non-Production
- Current SLA: 99.95% (Multi-AZ)
- Projected SLA: 99.9% (Single-AZ)
- RPO/RTO: Acceptable for staging workload

Risk: Low - Non-production environment
Note: Multi-AZ provides automatic failover in ~2 minutes
```

#### Aurora I/O-Optimized

**Example Recommendation:**
```
Resource: customer-aurora-cluster
Current: Standard configuration
Recommended: I/O-Optimized configuration

Cost Impact:
- Current: $584/month (instances) + $375/month (I/O) = $959/month
- Projected: $701/month (instances) + $0 (I/O) = $701/month
- Savings: $258/month (27%), $3,096/year

I/O Analysis:
- Monthly I/O requests: 1.9 billion
- I/O cost at $0.20/million: $375/month
- I/O-Optimized premium: 20% instance cost increase ($117/month)
- Net savings: $258/month

Recommendation: Switch to I/O-Optimized
Threshold: Beneficial when I/O costs > 15% of instance costs
```

#### Graviton Migration

**Example Recommendation:**
```
Resource: api-database-cluster
Current: 3x db.r5.xlarge (Intel-based)
Recommended: 3x db.r6g.xlarge (Graviton2-based)

Cost Impact:
- Current: $1,095/month (3 instances @ $365/month each)
- Projected: $876/month (3 instances @ $292/month each)
- Savings: $219/month (20%), $2,628/year

Performance Analysis:
- Graviton2 offers equivalent or better performance
- 20% lower cost per instance
- Engine: aurora-postgresql 14.7 (Graviton compatible)
- Same vCPU and memory configuration

Risk: Very Low - Proven Graviton performance
Note: Requires engine version compatibility check
```

### 4. Terraform Generation

For each recommendation, JetScale generates production-ready Terraform code:

**Example: Instance Right-Sizing**

```hcl
# RDS Instance Optimization
# Generated by JetScale on 2024-01-15
# Recommendation ID: rec_rds_001

resource "aws_db_instance" "production_postgres" {
  identifier = "production-postgres-db"

  # Previous: db.r5.2xlarge ($730/month)
  # Optimized: db.r5.xlarge ($365/month)
  # Cost Reduction: 50% ($365/month, $4,380/year)
  #
  # Justification:
  # - Average CPU utilization: 15-25%
  # - Peak CPU utilization: 35%
  # - Average memory utilization: 30-40%
  # - New instance provides 2x headroom above peak usage
  # - Performance monitoring shows no memory pressure
  instance_class = "db.r5.xlarge"

  engine         = "postgres"
  engine_version = "14.7"

  allocated_storage     = 500
  storage_type          = "gp3"
  iops                  = 3000
  storage_encrypted     = true

  multi_az = true

  # Existing configuration preserved
  db_name  = var.db_name
  username = var.db_username
  password = var.db_password

  vpc_security_group_ids = var.security_group_ids
  db_subnet_group_name   = var.db_subnet_group_name

  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"

  skip_final_snapshot = false
  final_snapshot_identifier = "${var.identifier}-final-snapshot"

  tags = merge(
    var.tags,
    {
      "jetscale:optimized" = "true"
      "jetscale:recommendation" = "rec_rds_001"
    }
  )
}
```

**Example: Aurora I/O-Optimized**

```hcl
# Aurora Cluster I/O Optimization
# Generated by JetScale on 2024-01-15

resource "aws_rds_cluster" "customer_aurora" {
  cluster_identifier = "customer-aurora-cluster"

  engine         = "aurora-postgresql"
  engine_version = "15.4"
  engine_mode    = "provisioned"

  # Switch to I/O-Optimized configuration
  # Previous: Standard (High I/O costs: $375/month)
  # Optimized: I/O-Optimized (20% instance premium, $0 I/O costs)
  # Net savings: $258/month (27%), $3,096/year
  #
  # Analysis:
  # - Monthly I/O: 1.9 billion requests
  # - I/O cost (Standard): $375/month
  # - Instance premium (I/O-Optimized): $117/month
  # - Net benefit: $258/month
  storage_type = "aurora-iopt1"

  master_username = var.master_username
  master_password = var.master_password

  database_name = var.database_name

  vpc_security_group_ids = var.vpc_security_group_ids
  db_subnet_group_name   = var.db_subnet_group_name

  backup_retention_period = 7
  preferred_backup_window = "03:00-04:00"
  preferred_maintenance_window = "mon:04:00-mon:05:00"

  enabled_cloudwatch_logs_exports = ["postgresql"]

  tags = merge(
    var.tags,
    {
      "jetscale:optimized" = "true"
      "jetscale:storage-mode" = "io-optimized"
    }
  )
}

resource "aws_rds_cluster_instance" "customer_aurora_instances" {
  count = 3

  identifier         = "customer-aurora-${count.index + 1}"
  cluster_identifier = aws_rds_cluster.customer_aurora.id

  instance_class = "db.r6g.xlarge"
  engine         = aws_rds_cluster.customer_aurora.engine
  engine_version = aws_rds_cluster.customer_aurora.engine_version

  publicly_accessible = false

  tags = var.tags
}
```

## Best Practices

### Monitoring After Changes

After applying JetScale recommendations:

1. **First 24 Hours**: Monitor key metrics closely
   - CPU and memory utilization
   - Database connections
   - Query latency (p50, p95, p99)
   - Storage IOPS and throughput

2. **Week 1**: Validate performance
   - Compare query execution times
   - Check for any memory pressure indicators
   - Monitor replication lag (if applicable)
   - Review application error rates

3. **Week 2-4**: Confirm cost savings
   - Verify AWS billing reflects expected savings
   - Ensure no unexpected charges
   - Check Reserved Instance recommendations

### Testing Strategy

**Pre-Production Testing:**
1. Apply changes to dev/staging environment first
2. Run load tests simulating peak traffic
3. Monitor for 48-72 hours under realistic load
4. Validate backup/restore procedures

**Production Rollout:**
1. Schedule changes during maintenance windows
2. Use blue/green deployments for Aurora (zero-downtime)
3. Have rollback plan ready
4. Monitor actively during and after change

### Reserved Instances & Savings Plans

**When to Purchase:**
- Stable, long-running databases (> 1 year)
- After right-sizing (don't reserve oversized instances)
- When commitment savings > opportunity cost

**JetScale Recommendations:**
- We analyze RI utilization and suggest optimal purchases
- Consider 1-year over 3-year for flexibility
- Size-flexible RIs allow instance family changes
- Combine with Compute Savings Plans for maximum flexibility

## Common Optimization Patterns

### Pattern 1: Over-Provisioned OLTP Database

**Symptoms:**
- CPU utilization < 20% average
- Memory utilization < 40%
- Infrequent connection spikes

**JetScale Recommendation:**
- Downsize by 1-2 instance sizes
- Typical savings: 50-66%
- Risk: Low (2-3x headroom maintained)

### Pattern 2: High I/O Costs on Aurora

**Symptoms:**
- I/O costs > 15% of total RDS bill
- Frequent read-heavy queries
- Large table scans

**JetScale Recommendation:**
- Switch to I/O-Optimized configuration
- Typical savings: 20-40% on total cluster cost
- Additional benefit: Predictable costs

### Pattern 3: Non-Production Multi-AZ

**Symptoms:**
- Dev/staging/test environments with Multi-AZ enabled
- High availability not required for non-production
- Acceptable downtime tolerance

**JetScale Recommendation:**
- Disable Multi-AZ for non-production workloads
- Typical savings: 50% on instance costs
- Trade-off: Manual failover required vs automatic
- Best for: Development, staging, testing, analytics

### Pattern 4: Missed Graviton Opportunities

**Symptoms:**
- Using Intel-based instances (r5, m5, r6i)
- Compatible engine versions available
- No ARM-specific dependencies

**JetScale Recommendation:**
- Migrate to Graviton instances (r6g, r7g, m6g, m7g)
- Typical savings: 10-40% with same or better performance
- Requirements: Verify engine version compatibility
- Best for: Most modern RDS engines (PostgreSQL 12+, MySQL 8+, MariaDB 10.4+)

## Troubleshooting

### Recommendation Concerns

**Q: Will downgrading my instance impact performance?**

A: JetScale maintains 2-3x headroom above peak usage. We analyze:
- P99 CPU/memory utilization over 14 days
- Buffer cache hit ratios
- Network and storage saturation points
- Growth trends

Recommendations only proceed if performance risk is Low or Very Low.

**Q: What about sudden traffic spikes?**

A: Our analysis includes:
- P99 (99th percentile) metrics, not just averages
- Buffer for unexpected spikes (2-3x above peak)
- Historical spike patterns
- Application-level health check validation

**Q: Can I test before committing?**

A: Yes! Apply changes to dev/staging first:
1. JetScale generates Terraform for all environments
2. Test in non-production for 48-72 hours
3. Monitor performance metrics
4. Rollback if needed (simple Terraform revert)
5. Apply to production once validated

### Performance Issues After Optimization

**Symptom: Increased query latency**

Possible causes:
- Insufficient connection pool size for smaller instance
- Memory pressure causing increased disk I/O
- Parameter group settings need tuning

**Resolution:**
1. Check `FreeableMemory` metric
2. Review slow query logs
3. Increase instance size by one step if needed
4. Adjust `shared_buffers` or equivalent parameter

**Symptom: High I/O costs on Aurora**

Possible causes:
- Using Aurora Standard with high I/O workload
- Frequent read-heavy queries
- Large table scans

**Resolution:**
1. Check I/O cost percentage of total RDS bill
2. If I/O costs > 15% of instance costs, consider I/O-Optimized
3. Monitor `VolumeReadIOPs` and `VolumeWriteIOPs` metrics
4. I/O-Optimized adds 20% to instance cost but eliminates all I/O charges

## Security Considerations

### Encryption

JetScale recommendations preserve existing encryption settings:
- Storage encryption state maintained
- KMS keys unchanged
- TLS/SSL connection requirements preserved

### Compliance

Instance changes maintain compliance:
- Multi-AZ configuration only changed when explicitly recommended
- Engine and engine version never modified
- VPC and security group associations preserved
- Existing parameter groups maintained

### Credentials

- Terraform uses `var.db_password` (externalized)
- Master passwords not exposed in generated code
- Secrets should use AWS Secrets Manager
- Rotation policies unchanged

## API Integration

JetScale provides API access for programmatic optimization:

```bash
# List RDS recommendations
GET /api/v1/recommendations?resource_type=rds

# Get specific recommendation details
GET /api/v1/recommendations/{recommendation_id}

# Approve recommendation (generates Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Retrieve generated Terraform
GET /api/v1/recommendations/{recommendation_id}/terraform
```

See our [API Documentation](../api-reference.md) for complete reference.

## Support

Need help with RDS optimization?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: [FAQ](../faq.md)
- **GitHub Issues**: [Report a problem](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

**Related Documentation:**
- [ElastiCache Optimization](elasticache.md)
- [EC2 Optimization](ec2.md)
- [EBS Optimization](ebs.md)
