# Azure SQL Optimization

JetScale provides AI-powered cost optimization for Azure SQL Database, including single databases, elastic pools, and Azure SQL Managed Instance. Our specialized agents analyze your database workloads to identify right-sizing opportunities and configuration improvements.

## Overview

JetScale optimizes Azure SQL resources by analyzing:
- **Database utilization**: DTU/vCore usage, CPU, memory, and I/O patterns
- **Cost analysis**: Current spending vs. optimal configuration
- **Performance metrics**: Query performance, connection patterns, storage usage
- **Service tier alignment**: Basic, Standard, Premium, Business Critical, Hyperscale

## Supported Azure SQL Types

### Azure SQL Database (Single Database)

Single databases are our primary optimization target.

**Database Architecture:**
```
SqlDatabase (Optimization Target, Billable)
├── Compute (DTU or vCore)
├── Storage (Data + Log)
└── Backup Storage
```

**What We Optimize:**
- **Service tier**: Right-size between Basic, Standard, Premium, Business Critical, Hyperscale
- **Compute tier**: Provisioned vs. Serverless
- **DTU/vCore sizing**: Match actual workload requirements
- **Storage configuration**: Data max size and performance tier

### Azure SQL Elastic Pools

Elastic pools share resources across multiple databases.

**Pool Architecture:**
```
SqlElasticPool (Optimization Target, Billable)
├── Database 1 - Shares pool resources
├── Database 2 - Shares pool resources
└── Database N - Shares pool resources
```

**What We Optimize:**
- **Pool sizing**: Total DTUs/vCores for all databases
- **Service tier**: Standard, Premium, Business Critical
- **Database density**: Optimal number of databases per pool

### Azure SQL Managed Instance

Fully managed SQL Server instance.

**Instance Architecture:**
```
SqlManagedInstance (Optimization Target, Billable)
├── Compute (vCores)
├── Storage
└── Multiple Databases
```

**What We Optimize:**
- **Instance sizing**: vCore count (4, 8, 16, 24, 32, 40, 64, 80)
- **Service tier**: General Purpose vs. Business Critical
- **Storage configuration**: Data and log storage sizing

## Service Tiers

### DTU-Based Model (Single Database & Elastic Pools)

**Basic Tier**
- **Use for**: Small databases, development, testing, light workloads
- **DTU range**: 5 DTUs
- **Max database size**: 2 GB
- **Typical cost**: ~$5/month
- **Best when**: Learning, prototyping, minimal production use

**Standard Tier**
- **Use for**: Most production workloads, web applications, line-of-business apps
- **DTU range**: 10, 20, 50, 100, 200, 400, 800, 1600, 3000 DTUs
- **Max database size**: Up to 1 TB
- **Typical cost**: $15-$1,500/month
- **Best when**: Predictable performance requirements, balanced workloads

**Premium Tier**
- **Use for**: Mission-critical applications, high transaction rate, low latency
- **DTU range**: 125, 250, 500, 1000, 1750, 4000 DTUs
- **Max database size**: Up to 4 TB
- **Typical cost**: $465-$14,500/month
- **Best when**: Highest performance, availability, and recovery requirements

### vCore-Based Model (All Types)

**General Purpose**
- **Use for**: Most production workloads, balanced compute and I/O
- **vCores**: 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 24, 32, 40, 80, 128
- **Max memory**: 625 GB (128 vCore)
- **Storage**: Remote storage (SSD), up to 4 TB (single) or 16 TB (managed instance)
- **Typical cost**: $300-$8,000/month
- **Best when**: Standard business workloads

**Business Critical**
- **Use for**: Mission-critical applications, low latency, high availability
- **vCores**: Same range as General Purpose
- **Max memory**: 625 GB (128 vCore)
- **Storage**: Local SSD, up to 4 TB (single) or 16 TB (managed instance)
- **Built-in HA**: Always-on availability group with readable secondary replicas
- **Typical cost**: 2.5x General Purpose pricing
- **Best when**: Highest performance and availability required

**Hyperscale** (Single Database only)
- **Use for**: Very large databases (100 TB+), rapid scale-up/down
- **vCores**: 2-80 vCores
- **Storage**: Highly scalable, up to 100 TB
- **Architecture**: Multi-tier with page servers and compute nodes
- **Typical cost**: Similar to General Purpose, storage charged separately
- **Best when**: Database > 4 TB, need rapid scaling

### Compute Tiers

**Provisioned**
- **Billing**: Per second for compute resources
- **Best for**: Predictable workloads, continuous usage
- **Minimum**: No minimum compute

**Serverless** (General Purpose only)
- **Billing**: Per second when active, auto-pause when idle
- **Best for**: Intermittent, unpredictable usage patterns
- **Auto-pause**: Configurable delay (1 hour - 7 days)
- **Savings**: Up to 90% during idle periods
- **Note**: Cold start delay when resuming from paused state

## How JetScale Optimizes Azure SQL

### 1. Data Collection

JetScale analyzes multiple data sources:

**Azure Monitor Metrics** (14-day rolling window):
- `cpu_percent` - CPU utilization
- `dtu_consumption_percent` - DTU usage (DTU model)
- `storage_percent` - Storage utilization
- `connection_successful` / `connection_failed` - Connection patterns
- `sessions_percent` - Active sessions
- `workers_percent` - Worker thread usage
- `deadlock` - Deadlock occurrences
- `log_write_percent` - Transaction log write usage

**Azure Resource Manager Data**:
- Service tier and compute tier
- DTU/vCore configuration
- Current storage size and max size
- Geo-replication status
- Backup retention settings

**Cost Management Data**:
- Current monthly spend per database
- Compute, storage, and backup costs breakdown
- Historical cost trends

**Azure Advisor** (if available):
- Azure-generated right-sizing recommendations
- Performance optimization suggestions

### 2. Analysis

Our AI agents perform deep analysis:

**Utilization Patterns:**
- Peak vs. average DTU/vCore usage
- Time-of-day patterns (identify idle periods for serverless)
- Day-of-week patterns (weekend vs. weekday load)
- Growth trends over time

**Performance Assessment:**
- Query performance metrics
- Connection patterns and failures
- Deadlock frequency
- Transaction log pressure

**Cost Modeling:**
- Current cost breakdown (compute, storage, backup)
- Projected cost for alternative configurations
- Provisioned vs. Serverless cost comparison
- Reserved capacity opportunities

**Tier Selection:**
- Basic vs. Standard vs. Premium alignment
- General Purpose vs. Business Critical requirements
- Hyperscale suitability for large databases
- DTU vs. vCore model efficiency

### 3. Recommendations

JetScale generates specific, actionable recommendations:

#### Service Tier Downgrade

**Example Recommendation:**
```
Resource: app-database-prod
Current: Premium P2 (250 DTUs, $930/month)
Recommended: Standard S3 (100 DTUs, $300/month)

Cost Impact:
- Current: $930/month
- Projected: $300/month
- Savings: $630/month (68%), $7,560/year

Performance Analysis:
- Average DTU usage: 35%
- Peak DTU usage: 58%
- Connection patterns: Normal
- Query performance: Well within Standard tier capabilities

Risk: Low - Peak usage well below new tier limits
Note: Premium tier features (lower latency, more IOPS) not fully utilized
```

#### DTU Right-Sizing

**Example Recommendation:**
```
Resource: customer-db-01
Current: Standard S7 (800 DTUs, $600/month)
Recommended: Standard S3 (100 DTUs, $300/month)

Cost Impact:
- Current: $600/month
- Projected: $300/month
- Savings: $300/month (50%), $3,600/year

Performance Analysis:
- Average DTU usage: 18%
- Peak DTU usage: 42%
- 95th percentile: 35%
- Recommended tier provides 2.4x headroom above peak

Risk: Very Low - Substantial headroom maintained
```

#### Serverless Migration

**Example Recommendation:**
```
Resource: reporting-database
Current: General Purpose (4 vCores, Provisioned, $730/month)
Recommended: General Purpose (4 vCores, Serverless, ~$290/month)

Cost Impact:
- Current: $730/month (24/7 billing)
- Projected: $290/month (avg 40% active time)
- Savings: $440/month (60%), $5,280/year

Usage Analysis:
- Active hours: ~10 hours/day (business hours only)
- Idle time: Nights and weekends (60% of time)
- Query patterns: Batch processing and reporting
- Cold start tolerance: Acceptable (reporting use case)

Configuration:
- Auto-pause delay: 1 hour
- Min vCores: 1 (when active)
- Max vCores: 4 (during peak)

Risk: Very Low - Intermittent usage pattern ideal for serverless
Note: ~2-minute resume time from paused state
```

#### vCore Optimization

**Example Recommendation:**
```
Resource: api-database-prod
Current: General Purpose (16 vCores, $1,460/month)
Recommended: General Purpose (8 vCores, $730/month)

Cost Impact:
- Current: $1,460/month
- Projected: $730/month
- Savings: $730/month (50%), $8,760/year

Performance Analysis:
- Average CPU: 22%
- Peak CPU: 38%
- Memory usage: 45%
- I/O patterns: Well within General Purpose limits

Risk: Low - 2.6x headroom above peak CPU usage
```

#### Business Critical → General Purpose

**Example Recommendation:**
```
Resource: legacy-app-database
Current: Business Critical (8 vCores, $3,650/month)
Recommended: General Purpose (8 vCores, $1,460/month)

Cost Impact:
- Current: $3,650/month
- Projected: $1,460/month
- Savings: $2,190/month (60%), $26,280/year

Workload Analysis:
- Business Critical features not utilized:
  × No readable secondary replicas in use
  × Low latency requirements not critical
  × Local SSD performance not required
- Performance requirements met by General Purpose

Risk: Medium - Verify application tolerates remote storage latency
Recommendation: Test in staging environment first
```

#### Elastic Pool Consolidation

**Example Recommendation:**
```
Resource: 12 Standard S3 databases
Current: 12 × S3 (100 DTUs each, $3,600/month total)
Recommended: 1 × Elastic Pool (600 eDTUs, $900/month)

Cost Impact:
- Current: $3,600/month (12 individual databases)
- Projected: $900/month (elastic pool with 12 databases)
- Savings: $2,700/month (75%), $32,400/year

Usage Analysis:
- Individual database peak usage: 40-60 DTUs
- Simultaneous peak usage: Rare (time-shifted workloads)
- Total concurrent DTU usage: ~300 DTUs peak
- Pool provides 600 eDTUs with resource sharing

Risk: Low - Sufficient pool capacity for peak concurrent usage
Note: Databases share pool resources, reducing over-provisioning
```

### 4. Terraform Generation

For each recommendation, JetScale generates production-ready Terraform code:

**Example: Service Tier Optimization**

```hcl
# Azure SQL Database Optimization
# Generated by JetScale on 2024-01-15
# Recommendation ID: rec_azuresql_001

resource "azurerm_mssql_database" "app_database_prod" {
  name      = "app-database-prod"
  server_id = azurerm_mssql_server.example.id

  # Previous: Premium P2 (250 DTUs, $930/month)
  # Optimized: Standard S3 (100 DTUs, $300/month)
  # Cost Reduction: 68% ($630/month, $7,560/year)
  #
  # Justification:
  # - Average DTU usage: 35%
  # - Peak DTU usage: 58%
  # - Premium tier features (lower latency, higher IOPS) not fully utilized
  # - Standard S3 provides adequate performance for workload
  sku_name = "S3"

  max_size_gb = 250

  # Existing configuration preserved
  collation               = "SQL_Latin1_General_CP1_CI_AS"
  zone_redundant          = false
  read_scale              = false
  storage_account_type    = "Geo"

  threat_detection_policy {
    state                      = "Enabled"
    email_account_admins       = "Enabled"
    retention_days             = 30
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"      = "true"
      "jetscale:recommendation" = "rec_azuresql_001"
      "jetscale:previous_sku"   = "P2"
    }
  )
}
```

**Example: Serverless Migration**

```hcl
# Azure SQL Serverless Migration
# Generated by JetScale on 2024-01-15

resource "azurerm_mssql_database" "reporting_database" {
  name      = "reporting-database"
  server_id = azurerm_mssql_server.example.id

  # Previous: Provisioned (4 vCores, 24/7, $730/month)
  # Optimized: Serverless (1-4 vCores, ~40% active, $290/month)
  # Cost Reduction: 60% ($440/month, $5,280/year)
  #
  # Usage Pattern Analysis:
  # - Active: Business hours only (~10 hrs/day)
  # - Idle: Nights and weekends (60% of time)
  # - Use case: Batch reporting (cold start acceptable)
  #
  # Serverless Configuration:
  # - Auto-pause after 1 hour of inactivity
  # - Min 1 vCore when active, max 4 vCores
  # - ~2-minute resume time from paused state
  sku_name = "GP_S_Gen5_4"

  min_capacity = 1.0
  max_capacity = 4.0

  # Auto-pause after 1 hour (60 minutes) of inactivity
  auto_pause_delay_in_minutes = 60

  max_size_gb = 100

  collation               = "SQL_Latin1_General_CP1_CI_AS"
  zone_redundant          = false
  read_scale              = false
  storage_account_type    = "Geo"

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"     = "true"
      "jetscale:compute_model" = "serverless"
      "jetscale:previous_sku"  = "GP_Gen5_4"
    }
  )
}
```

**Example: Elastic Pool**

```hcl
# Azure SQL Elastic Pool Consolidation
# Generated by JetScale on 2024-01-15

resource "azurerm_mssql_elasticpool" "app_pool" {
  name                = "app-elastic-pool"
  resource_group_name = var.resource_group_name
  location            = var.location
  server_name         = azurerm_mssql_server.example.name

  # Previous: 12 individual S3 databases (1200 DTUs total, $3,600/month)
  # Optimized: Elastic pool (600 eDTUs shared, $900/month)
  # Cost Reduction: 75% ($2,700/month, $32,400/year)
  #
  # Analysis:
  # - Individual database peaks rarely coincide
  # - Peak concurrent usage: ~300 DTUs
  # - Pool provides 600 eDTUs with 2x headroom
  # - Resource sharing eliminates over-provisioning
  sku {
    name     = "StandardPool"
    tier     = "Standard"
    capacity = 600
  }

  per_database_settings {
    min_capacity = 10
    max_capacity = 100
  }

  max_size_gb = 750

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"         = "true"
      "jetscale:pool_type"         = "consolidation"
      "jetscale:database_count"    = "12"
      "jetscale:previous_total_cost" = "$3600"
    }
  )
}

# Move databases to elastic pool
resource "azurerm_mssql_database" "pooled_databases" {
  count = 12

  name           = "app-database-${count.index + 1}"
  server_id      = azurerm_mssql_server.example.id
  elastic_pool_id = azurerm_mssql_elasticpool.app_pool.id

  # Databases now share pool resources
  # No individual sku_name needed

  tags = var.tags
}
```

## Best Practices

### Monitoring After Changes

After applying JetScale recommendations:

1. **First 24 Hours**: Monitor key metrics closely
   - CPU and DTU/vCore utilization
   - Connection success rate
   - Query performance (execution time)
   - Transaction log usage

2. **Week 1**: Validate performance
   - Compare query execution times
   - Check for resource pressure indicators
   - Monitor connection failures
   - Review application error rates

3. **Week 2-4**: Confirm cost savings
   - Verify Azure billing reflects expected savings
   - Check for unexpected charges
   - Consider reserved capacity for stable workloads

### Testing Strategy

**Pre-Production Testing:**
1. Apply changes to dev/staging environment first
2. Run load tests simulating peak usage
3. Monitor for 48-72 hours under realistic load
4. Validate backup/restore procedures

**Production Rollout:**
1. Schedule changes during maintenance windows
2. Use Azure SQL geo-replication for blue/green deployments
3. Have rollback plan ready
4. Monitor actively during and after change

### Reserved Capacity

**When to Purchase:**
- Stable, long-running databases (> 1 year)
- After right-sizing (don't reserve oversized resources)
- Up to 80% savings vs. pay-as-you-go

**JetScale Recommendations:**
- 1-year or 3-year commitment
- Applied automatically to matching resources
- Size-flexible within same service tier

## Common Optimization Patterns

### Pattern 1: Over-Provisioned Premium Database

**Symptoms:**
- Premium tier with low DTU usage (<40%)
- High availability features not utilized
- Lower latency not critical for application

**JetScale Recommendation:**
- Downgrade to Standard tier
- Typical savings: 60-70%
- Risk: Low (verify latency requirements)

### Pattern 2: Idle Development Databases

**Symptoms:**
- Databases used only during business hours
- Zero activity nights and weekends
- Development or reporting workloads

**JetScale Recommendation:**
- Migrate to Serverless compute tier
- Typical savings: 50-80%
- Risk: Very Low (non-continuous usage)

### Pattern 3: Multiple Small Databases

**Symptoms:**
- Many databases with similar usage patterns
- Individual database peaks don't coincide
- Over-provisioning due to peak capacity per database

**JetScale Recommendation:**
- Consolidate into Elastic Pool
- Typical savings: 60-75%
- Risk: Low (resource sharing reduces waste)

### Pattern 4: Business Critical Over-Provisioning

**Symptoms:**
- Business Critical tier without HA requirements
- Readable secondary replicas not used
- Local SSD performance not required

**JetScale Recommendation:**
- Downgrade to General Purpose
- Typical savings: 60% (2.5x price difference)
- Risk: Medium (verify latency tolerance)

## Troubleshooting

### Recommendation Concerns

**Q: Will downgrading impact performance?**

A: JetScale maintains 2-3x headroom above peak usage. We analyze:
- P95 and P99 DTU/vCore usage over 14 days
- Query performance metrics
- Connection patterns and failures
- Historical spike patterns

Recommendations only proceed if performance risk is Low or Very Low.

**Q: What about serverless cold start delays?**

A: Serverless databases resume in ~2 minutes from paused state. Best for:
- Batch processing and reporting
- Development and testing
- Applications tolerant of occasional cold starts
- Avoid for: Real-time APIs, always-on applications

**Q: Can I test before committing?**

A: Yes! Apply changes to dev/staging first:
1. JetScale generates Terraform for all environments
2. Test in non-production for 48-72 hours
3. Monitor performance metrics
4. Rollback if needed (simple tier change)
5. Apply to production once validated

### Performance Issues After Optimization

**Symptom: Increased query latency**

Possible causes:
- Insufficient DTU/vCore capacity
- Storage I/O throttling
- Connection pool exhaustion

**Resolution:**
1. Check `cpu_percent` and `dtu_consumption_percent` metrics
2. Review slow query logs
3. Increase tier/size by one step if needed
4. Consider Query Performance Insights for optimization

**Symptom: Connection failures**

Possible causes:
- Serverless database in paused state
- Connection pool settings need adjustment
- Resource limits exceeded

**Resolution:**
1. If serverless: Reduce auto-pause delay or switch to provisioned
2. Check `connection_failed` metric
3. Review connection pool configuration
4. Monitor `sessions_percent` for capacity

## Security Considerations

### Encryption

JetScale recommendations preserve:
- Transparent Data Encryption (TDE) settings
- Always Encrypted configuration
- SSL/TLS connection requirements
- Encryption at rest and in transit

### Compliance

Configuration changes maintain compliance:
- Geo-replication settings preserved
- Backup retention policies unchanged
- Threat detection and auditing settings maintained
- Virtual network rules and firewall rules preserved

## Limitations

**Not Currently Supported:**
- Reserved capacity purchase recommendations
- Cross-region replication optimization
- Backup retention policy optimization
- Index and query tuning recommendations

**Roadmap:**
These features are planned for future releases. Currently, JetScale focuses on service tier and compute optimization.

## API Integration

JetScale provides API access for programmatic optimization:

```bash
# List Azure SQL recommendations
GET /api/v1/recommendations?resource_type=azure-sql

# Get specific recommendation details
GET /api/v1/recommendations/{recommendation_id}

# Approve recommendation (generates Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Retrieve generated Terraform
GET /api/v1/recommendations/{recommendation_id}/terraform
```

See our [API Documentation](../api-reference.md) for complete reference.

## Support

Need help with Azure SQL optimization?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: [FAQ](../faq.md)
- **GitHub Issues**: [Report a problem](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

**Related Documentation:**
- [Azure VM Optimization](azure-vm.md)
- [RDS Optimization](rds.md)
