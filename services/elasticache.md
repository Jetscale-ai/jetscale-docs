# ElastiCache Optimization

JetScale provides AI-powered cost optimization for Amazon ElastiCache including both Redis and Memcached clusters. Our specialized agents analyze your caching workloads to identify right-sizing opportunities, node type migrations, and configuration improvements.

## Overview

JetScale optimizes ElastiCache resources by analyzing:
- **Node utilization**: CPU, memory, network, and connection patterns
- **Cache performance**: Hit rates, eviction patterns, swap usage
- **Cost analysis**: Current spending vs. optimal configuration
- **High availability**: Multi-AZ configurations and replication topology

## Supported ElastiCache Types

### Redis Clusters

Redis is our **primary optimization target** for feature-rich caching with persistence and replication.

**Cluster Modes:**

**Redis Cluster Mode Disabled:**
```
ElastiCacheReplicationGroup (Optimization Target)
├── CacheNode (Primary) - Billable
├── CacheNode (Replica) - Billable
└── CacheNode (Replica) - Billable
```

**Redis Cluster Mode Enabled:**
```
ElastiCacheReplicationGroup (Optimization Target)
├── NodeGroup (Shard 1)
│   ├── CacheNode (Primary) - Billable
│   └── CacheNode (Replica) - Billable
├── NodeGroup (Shard 2)
│   ├── CacheNode (Primary) - Billable
│   └── CacheNode (Replica) - Billable
```

**What We Optimize:**
- **Node type right-sizing**: Match actual memory and CPU workload (all nodes in cluster use same type)
- **Graviton migration**: Switch to ARM-based instances (r7g, m7g, r6g, m6g) for 10-40% savings
- **Engine version upgrades**: Recommend latest Redis versions for performance and features
- **Multi-AZ configuration**: Disable Multi-AZ for ~50% savings when high availability isn't critical
- **Reserved node recommendations**: Up to 55% savings with 1-year or 3-year commitment

**Important:** All nodes in a Redis cluster must use the same node type. JetScale sizes for the node with highest resource usage to ensure adequate performance across all cluster members.

### Memcached Clusters

Traditional Memcached clusters are optimized as a collection of independent nodes.

**Cluster Architecture:**
```
ElastiCacheCluster (Optimization Target)
├── CacheNode - Billable
├── CacheNode - Billable
└── CacheNode - Billable
```

**What We Optimize:**
- **Node type right-sizing**: Match memory and CPU to workload
- **Graviton migration**: Switch to ARM-based instances (m7g, m6g) for 10-40% savings
- **Node count optimization**: Adjust number of nodes based on actual usage

**Key Differences from Redis:**
- No replication (each node is independent)
- No persistence (purely in-memory)
- Simpler scaling model (add/remove nodes)
- Generally lower cost than Redis

## ElastiCache Node Types

JetScale considers all AWS ElastiCache node types when optimizing:

### Memory Optimized (R-family)

**R7g (Graviton3)** - Latest generation, ARM-based
- Best price/performance ratio
- 40% better price/performance than R6g
- Use for: Redis with high memory needs, low-latency caching

**R6g (Graviton2)** - Previous ARM generation
- 20% cost savings vs. R5
- Excellent performance
- Use for: Redis requiring large memory footprint

**R5** - Intel-based
- Legacy option (migrate to R6g/R7g for savings)
- Use for: x86-specific requirements (rare)

### General Purpose (M-family)

**M7g (Graviton3)** - Latest generation, balanced
- Best for workloads with balanced CPU and memory needs
- 40% better price/performance than M6g

**M6g (Graviton2)** - ARM-based
- 20% cost savings vs. M5
- Use for: Balanced Redis/Memcached workloads

**M5** - Intel-based
- Legacy option (migrate to M6g/M7g)

### Burstable (T-family)

**T4g (Graviton2)** - Low-cost burstable
- Use for: Development, testing, small caches
- Baseline CPU with burst credits
- Lowest cost option
- Caution: Monitor CPU credit balance

**T3** - Intel-based burstable
- Legacy option (migrate to T4g)

### Node Sizing

Each node type comes in multiple sizes:

| Size | Example (R7g) | vCPUs | Memory |
|------|---------------|-------|--------|
| small | cache.r7g.small | 1 | 1.5 GB |
| medium | cache.r7g.medium | 1 | 3.1 GB |
| large | cache.r7g.large | 1 | 6.4 GB |
| xlarge | cache.r7g.xlarge | 2 | 13.1 GB |
| 2xlarge | cache.r7g.2xlarge | 4 | 26.3 GB |
| 4xlarge | cache.r7g.4xlarge | 8 | 52.8 GB |
| 8xlarge | cache.r7g.8xlarge | 16 | 105.8 GB |
| 12xlarge | cache.r7g.12xlarge | 24 | 159.0 GB |
| 16xlarge | cache.r7g.16xlarge | 32 | 212.2 GB |

## How JetScale Optimizes ElastiCache

### 1. Data Collection

JetScale analyzes multiple data sources:

**CloudWatch Metrics** (14-day rolling window):

**Redis Metrics:**
- `CPUUtilization` - Node CPU usage
- `DatabaseMemoryUsagePercentage` - Memory used for data
- `BytesUsedForCache` - Actual cache data size
- `FreeableMemory` - Available memory
- `NetworkBytesIn` / `NetworkBytesOut` - Network throughput
- `CacheHits` / `CacheMisses` - Cache efficiency
- `CurrConnections` - Active connections
- `SwapUsage` - Memory pressure indicator (should be zero)
- `EngineCPUUtilization` - Redis process CPU (key metric)
- `Evictions` - Items evicted due to memory pressure
- `ReplicationLag` - Replica lag time

**Memcached Metrics:**
- `CPUUtilization` - Node CPU usage
- `BytesUsedForCacheItems` - Memory used for items
- `FreeableMemory` - Available memory
- `NetworkBytesIn` / `NetworkBytesOut` - Network throughput
- `CurrConnections` - Active connections
- `Evictions` - Items evicted due to memory pressure
- `GetHits` / `GetMisses` - Cache efficiency
- `BytesReadIntoMemcached` / `BytesWrittenOutFromMemcached`

**ElastiCache API Data:**
- Node type and engine version
- Number of nodes/shards
- Multi-AZ status
- Automatic failover configuration
- Parameter group configuration
- Maintenance window settings

**Cost Explorer Data:**
- Current monthly spend per cluster
- Historical cost trends
- Reserved node utilization

**AWS Compute Optimizer** (if enabled):
- AWS-generated right-sizing recommendations
- Performance risk assessments

### 2. Analysis

Our AI agents perform deep analysis:

**Memory Utilization:**
- Peak vs. average memory usage
- Eviction patterns (indicates memory pressure)
- Swap usage (critical indicator - should be zero)
- Memory fragmentation patterns
- Headroom calculation for safe downsizing

**CPU Utilization:**
- Engine CPU vs. total CPU (Redis)
- Peak vs. average CPU usage
- CPU credit balance (T-family instances)
- Time-of-day patterns

**Cache Performance:**
- Cache hit rate (target: >95% for healthy cache)
- Eviction rate (indicates insufficient memory)
- Connection patterns
- Network saturation points

**Cost Modeling:**
- Current cost breakdown (compute, data transfer)
- Projected cost for alternative configurations
- Reserved node opportunities
- Multi-AZ premium analysis
- Graviton migration savings

**Risk Evaluation:**
- Headroom calculation (20% buffer above peak recommended)
- Performance degradation probability
- Availability impact assessment
- Failover capability impact

### 3. Recommendations

JetScale generates specific, actionable recommendations:

#### Node Type Right-Sizing

**Example Recommendation:**
```
Resource: production-redis-cache
Current: 2x cache.r5.xlarge (13.1 GB RAM each, 4 vCPUs)
Recommended: 2x cache.r5.large (6.4 GB RAM each, 2 vCPUs)

Cost Impact:
- Current: $500/month
- Projected: $250/month
- Savings: $250/month (50%), $3,000/year

Performance Analysis:
- Peak memory usage: 40% (5.2 GB per node)
- Average memory usage: 30% (3.9 GB per node)
- Peak CPU: 25%
- Cache hit rate: 97.5%
- No evictions detected
- Recommended provides 23% headroom above peak

Risk: Low - Adequate headroom maintained, no memory pressure
```

#### Graviton Migration (Redis)

**Example Recommendation:**
```
Resource: api-cache-cluster
Current: 3x cache.r5.large (Intel-based)
Recommended: 3x cache.r6g.large (Graviton2-based)

Cost Impact:
- Current: $375/month (3 nodes @ $125/month each)
- Projected: $300/month (3 nodes @ $100/month each)
- Savings: $75/month (20%), $900/year

Performance Analysis:
- Same memory: 6.4 GB per node
- Same vCPUs: 2 per node
- Graviton2 offers equivalent or better performance
- Redis 6.2+ fully compatible with ARM64
- No application changes required

Risk: Very Low - Proven Graviton performance with Redis
Note: Requires engine version 5.0.6+ for Graviton support
```

#### Multi-AZ Optimization

**Example Recommendation:**
```
Resource: staging-memcached-cluster
Current: 4x cache.m5.large with Multi-AZ enabled
Recommended: 4x cache.m5.large with Multi-AZ disabled

Cost Impact:
- Current: $480/month (Multi-AZ premium included)
- Projected: $240/month (Single-AZ)
- Savings: $240/month (50%), $2,880/year

Availability Analysis:
- Environment: Staging/Non-Production
- Current: Multi-AZ (automatic failover)
- Projected: Single-AZ (manual recovery)
- Acceptable for staging workload
- RPO/RTO: Tolerable for non-production

Risk: Low - Non-production environment
Note: Multi-AZ provides automatic failover within minutes
```

#### Engine Version Upgrade

**Example Recommendation:**
```
Resource: legacy-redis-cache
Current: 3x cache.r5.large, Redis 5.0.6
Recommended: 3x cache.r5.large, Redis 7.0

Cost Impact:
- Current: $375/month
- Projected: $375/month (same cost)
- Savings: $0/month (performance and security improvements)

Benefits:
- Redis 7.0 features: Functions, ACL improvements, Redis Streams enhancements
- Better memory efficiency (5-10% reduction in memory overhead)
- Improved replication performance
- Security patches and bug fixes
- Same cost, better performance

Risk: Low - Backward compatible, test in staging first
Note: Review Redis 7.0 breaking changes documentation
```

#### Reserved Node Recommendations

**Example Recommendation:**
```
Resource: production-redis-primary
Current: 2x cache.r6g.xlarge, On-Demand pricing
Recommended: 2x cache.r6g.xlarge, 1-Year Reserved (No Upfront)

Cost Impact:
- Current: $600/month (On-Demand)
- Projected: $420/month (Reserved)
- Savings: $180/month (30%), $2,160/year

Analysis:
- Cluster age: 18 months (proven stable workload)
- Utilization: 24/7 production cache
- Reserved term: 1-Year No Upfront
- Break-even: Immediate (monthly savings)
- Commitment: Can sell unused reservations on marketplace

Risk: Very Low - Stable long-running workload
Note: 3-Year Reserved offers 55% savings ($270/month, $390/month savings)
```

#### Memcached Node Count Optimization

**Example Recommendation:**
```
Resource: session-cache-memcached
Current: 6x cache.m5.large (16 GB total capacity)
Recommended: 4x cache.m5.large (16 GB total capacity by using larger nodes)
Alternative: 4x cache.m6g.xlarge (same total memory, Graviton2)

Cost Impact (Alternative):
- Current: $720/month (6 nodes @ $120/month)
- Projected: $640/month (4 nodes @ $160/month)
- Savings: $80/month (11%), $960/year

Performance Analysis:
- Total memory needed: 12 GB (peak)
- Current distribution: 2 GB per node average
- Recommended: 3 GB per node average
- Fewer nodes = reduced operational complexity
- Network performance per node is higher

Risk: Low - Same total capacity, better efficiency
Note: Test connection pooling with fewer nodes
```

### 4. Terraform Generation

For each recommendation, JetScale generates production-ready Terraform code:

**Example: Redis Right-Sizing**

```hcl
# ElastiCache Redis Optimization
# Generated by JetScale on 2024-01-15
# Recommendation ID: rec_elasticache_001

resource "aws_elasticache_replication_group" "production_redis" {
  replication_group_id       = "production-redis-cache"
  replication_group_description = "Production Redis cache"

  # Previous: cache.r5.xlarge ($500/month)
  # Optimized: cache.r5.large ($250/month)
  # Cost Reduction: 50% ($250/month, $3,000/year)
  #
  # Justification:
  # - Peak memory usage: 40% (5.2 GB per node)
  # - Average memory usage: 30% (3.9 GB per node)
  # - Peak CPU: 25%
  # - Cache hit rate: 97.5%
  # - New node type provides 23% headroom above peak
  # - No evictions or swap usage detected
  node_type = "cache.r5.large"

  engine         = "redis"
  engine_version = "7.0"

  # Cluster configuration
  num_cache_clusters         = 2
  automatic_failover_enabled = true
  multi_az_enabled          = true

  # Network configuration
  subnet_group_name  = var.subnet_group_name
  security_group_ids = var.security_group_ids

  # Maintenance and backup
  maintenance_window         = "sun:05:00-sun:06:00"
  snapshot_window           = "03:00-04:00"
  snapshot_retention_limit  = 5

  # Parameter group for configuration
  parameter_group_name = var.parameter_group_name

  # Encryption
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
  auth_token_enabled        = true

  # Logging
  log_delivery_configuration {
    destination      = var.cloudwatch_log_group
    destination_type = "cloudwatch-logs"
    log_format       = "json"
    log_type         = "slow-log"
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"      = "true"
      "jetscale:recommendation" = "rec_elasticache_001"
      "jetscale:previous_type"  = "cache.r5.xlarge"
    }
  )
}
```

**Example: Graviton Migration**

```hcl
# ElastiCache Graviton Migration
# Generated by JetScale on 2024-01-15

resource "aws_elasticache_replication_group" "api_cache_graviton" {
  replication_group_id       = "api-cache-cluster"
  replication_group_description = "API response cache - Graviton2"

  # Previous: cache.r5.large (Intel, $125/month per node)
  # Optimized: cache.r6g.large (Graviton2, $100/month per node)
  # Cost Reduction: 20% ($75/month total, $900/year)
  #
  # Performance:
  # - Same memory: 6.4 GB RAM
  # - Same vCPUs: 2 per node
  # - Graviton2 offers equivalent or better performance
  # - 20% cost savings with no performance trade-off
  #
  # Requirements:
  # - Redis engine 5.0.6+ for Graviton support
  # - No application changes required
  node_type = "cache.r6g.large"

  engine         = "redis"
  engine_version = "7.0"

  num_cache_clusters         = 3
  automatic_failover_enabled = true
  multi_az_enabled          = true

  subnet_group_name  = var.subnet_group_name
  security_group_ids = var.security_group_ids

  maintenance_window        = "sun:05:00-sun:06:00"
  snapshot_window          = "03:00-04:00"
  snapshot_retention_limit = 7

  parameter_group_name = var.parameter_group_name

  at_rest_encryption_enabled = true
  transit_encryption_enabled = true

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"    = "true"
      "jetscale:architecture" = "arm64"
      "jetscale:processor"    = "graviton2"
    }
  )
}
```

**Example: Memcached Optimization**

```hcl
# ElastiCache Memcached Optimization
# Generated by JetScale on 2024-01-15

resource "aws_elasticache_cluster" "session_cache" {
  cluster_id = "session-cache-memcached"

  # Previous: 6x cache.m5.large ($720/month)
  # Optimized: 4x cache.m6g.xlarge ($640/month)
  # Cost Reduction: 11% ($80/month, $960/year)
  #
  # Analysis:
  # - Total memory needed: 12 GB peak
  # - Fewer nodes reduce operational complexity
  # - Graviton2 provides 20% per-node savings
  # - Better network performance per node
  node_type = "cache.m6g.xlarge"

  engine         = "memcached"
  engine_version = "1.6.17"

  num_cache_nodes = 4
  az_mode        = "cross-az"

  subnet_group_name  = var.subnet_group_name
  security_group_ids = var.security_group_ids

  maintenance_window = "sun:05:00-sun:06:00"

  parameter_group_name = var.parameter_group_name

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"      = "true"
      "jetscale:previous_nodes" = "6"
      "jetscale:previous_type"  = "cache.m5.large"
    }
  )
}
```

## Best Practices

### Monitoring After Changes

After applying JetScale recommendations:

1. **First 24 Hours**: Monitor key metrics closely
   - Memory utilization and eviction rate
   - CPU utilization (especially EngineCPU for Redis)
   - Cache hit rate (should remain stable)
   - SwapUsage (should be zero)
   - Connection count and latency

2. **Week 1**: Validate performance
   - Application response times
   - Cache miss rate (should not increase)
   - No evictions under normal load
   - Network throughput within limits
   - Replication lag (Redis replicas)

3. **Week 2-4**: Confirm cost savings
   - Verify AWS billing reflects expected savings
   - No unexpected data transfer charges
   - Check Reserved node recommendations

### Testing Strategy

**Pre-Production Testing:**
1. Apply changes to dev/staging environment first
2. Run load tests simulating peak traffic
3. Monitor for 48-72 hours under realistic load
4. Validate cache performance metrics
5. Test failover scenarios (Redis Multi-AZ)

**Production Rollout:**
1. Schedule changes during maintenance windows
2. Use blue/green deployments for zero-downtime (create new cluster, switch traffic)
3. Have rollback plan ready
4. Monitor actively during and after change
5. Keep old cluster running for 24 hours before terminating

### Reserved Nodes & Cost Optimization

**When to Purchase Reserved Nodes:**
- Stable, long-running caches (>1 year expected lifetime)
- After right-sizing (don't reserve oversized nodes)
- When commitment savings > opportunity cost

**JetScale Recommendations:**
- We analyze reserved node utilization and suggest optimal purchases
- Consider 1-year over 3-year for flexibility
- No Upfront option provides monthly savings with zero upfront cost
- Partial Upfront offers slightly better savings
- All Upfront offers maximum savings (55% for 3-year Redis)

**Reserved Node Savings:**
- 1-Year No Upfront: ~30% savings
- 1-Year All Upfront: ~35% savings
- 3-Year No Upfront: ~50% savings
- 3-Year All Upfront: ~55% savings

## Common Optimization Patterns

### Pattern 1: Over-Provisioned Redis Cluster

**Symptoms:**
- Memory utilization < 50% consistently
- CPU utilization < 30%
- No evictions
- Cache hit rate >95%

**JetScale Recommendation:**
- Downsize node type by 1-2 sizes
- Typical savings: 50-66%
- Risk: Low (maintain 20%+ headroom)

### Pattern 2: Missed Graviton Opportunities

**Symptoms:**
- Using Intel-based instances (r5, m5, r6i)
- Redis 5.0.6+ or Memcached 1.5.16+
- No ARM-specific dependencies

**JetScale Recommendation:**
- Migrate to Graviton instances (r5 → r6g, m5 → m6g, or latest r7g/m7g)
- Typical savings: 20-40% with same or better performance
- Requirements: Verify engine version compatibility
- Best for: All modern Redis and Memcached deployments

### Pattern 3: Non-Production Multi-AZ

**Symptoms:**
- Dev/staging/test caches with Multi-AZ enabled
- High availability not required for non-production
- Acceptable downtime tolerance

**JetScale Recommendation:**
- Disable Multi-AZ and automatic failover
- Typical savings: 50% on node costs
- Trade-off: Manual recovery vs. automatic failover
- Best for: Development, staging, testing environments

### Pattern 4: Legacy Engine Versions

**Symptoms:**
- Running Redis 5.x or older
- Running Memcached 1.5.x or older
- Missing performance improvements and features

**JetScale Recommendation:**
- Upgrade to latest engine version (Redis 7.x, Memcached 1.6.x)
- Typical savings: 5-10% memory efficiency improvement
- Additional benefits: Security patches, new features, better performance
- Risk: Low to Medium (test for breaking changes)

### Pattern 5: High Eviction Rate

**Symptoms:**
- Evictions > 0 consistently
- Memory utilization at 100%
- Cache hit rate declining
- SwapUsage > 0 (critical indicator)

**JetScale Recommendation:**
- Upsize to larger node type (more memory)
- Alternative: Add more nodes (Memcached) or shards (Redis Cluster Mode)
- Typical cost increase: 50-100% BUT necessary for performance
- Risk: High if not addressed (cache thrashing, poor performance)

**Important:** This is a case where JetScale recommends spending MORE to improve performance and prevent cache failures.

## Troubleshooting

### Recommendation Concerns

**Q: Will downsizing impact cache hit rates?**

A: JetScale maintains 20%+ headroom above peak memory usage. We analyze:
- 99th percentile memory utilization over 14 days
- Eviction patterns
- Cache hit/miss ratios
- SwapUsage (critical indicator of memory pressure)

Recommendations only proceed if:
- No evictions detected in baseline period
- SwapUsage is zero
- Performance risk is Low or Very Low

**Q: What about sudden traffic spikes?**

A: Our analysis includes:
- P99 (99th percentile) metrics, not just averages
- 20% buffer above peak for unexpected spikes
- Historical spike patterns
- Headroom for growth

**Q: How do I test Graviton migration?**

A: Recommended approach:
1. Create new Graviton-based cluster in staging
2. Configure application to use both clusters (A/B test)
3. Compare performance metrics for 48-72 hours
4. Full cutover to Graviton cluster once validated
5. Apply to production after staging success

### Performance Issues After Optimization

**Symptom: Increased cache miss rate**

Possible causes:
- Insufficient memory (too aggressive downsizing)
- Evictions occurring
- Key distribution issues (Redis Cluster Mode)

**Resolution:**
1. Check `Evictions` metric (should be zero)
2. Check `DatabaseMemoryUsagePercentage` (should be <80%)
3. Increase node size if memory pressure detected
4. Review key expiration policies

**Symptom: High CPU utilization**

Possible causes:
- Insufficient CPU for command processing
- Inefficient cache access patterns
- Large key operations

**Resolution:**
1. Check `EngineCPUUtilization` (Redis) or `CPUUtilization` (Memcached)
2. Review slow log for expensive operations
3. Increase node size if CPU consistently >70%
4. Optimize application cache access patterns

**Symptom: SwapUsage > 0**

**CRITICAL ISSUE** - Memory pressure detected

Possible causes:
- Node type has insufficient memory
- Memory leak in application
- Key sizes larger than expected

**Resolution:**
1. Immediately increase node size (memory pressure causes severe performance degradation)
2. Review `BytesUsedForCache` growth trends
3. Check for memory leaks
4. Implement key eviction policies if appropriate

**Symptom: Replication lag increasing (Redis)**

Possible causes:
- High write volume
- Network saturation
- Undersized replicas

**Resolution:**
1. Check `ReplicationLag` metric
2. Verify network throughput (`NetworkBytesOut` from primary)
3. Increase node size if replicas can't keep up
4. Consider Redis Cluster Mode for write distribution

## Security Considerations

### Encryption

JetScale recommendations preserve existing encryption settings:
- At-rest encryption state maintained
- Transit encryption (TLS) state maintained
- Auth tokens preserved (Redis)
- No changes to KMS keys

### Network Security

Configuration preserved:
- VPC and subnet group associations
- Security group rules unchanged
- Private subnet isolation maintained
- No public endpoint changes

### Access Control

- Redis AUTH preserved
- Parameter group associations maintained
- IAM authentication settings unchanged
- Compliance requirements respected

## Engine Compatibility

### Redis Graviton Support

**Minimum versions for Graviton:**
- Redis 5.0.6+: Supports r6g, m6g (Graviton2)
- Redis 6.2+: Recommended for Graviton2
- Redis 7.0+: Full Graviton3 support (r7g, m7g)

**Feature compatibility:**
- All Redis features supported on Graviton
- Same clustering capabilities
- Same replication and persistence
- No application code changes required

### Memcached Graviton Support

**Minimum versions:**
- Memcached 1.5.16+: Supports m6g (Graviton2)
- Memcached 1.6.6+: Recommended for Graviton2

**Feature compatibility:**
- All Memcached features supported
- Same protocol and commands
- No application changes required

## Limitations

**Not Currently Supported:**
- Global Datastore optimization (Redis)
- Backup and restore cost optimization
- Data tiering recommendations (Redis)
- Outpost deployments

**Cluster Mode Enabled Limitations:**
- Cannot change number of shards without data migration
- Resharding is complex and must be planned separately
- JetScale optimizes node types but not shard topology

## API Integration

JetScale provides API access for programmatic optimization:

```bash
# List ElastiCache recommendations
GET /api/v1/recommendations?resource_type=elasticache

# Get specific recommendation details
GET /api/v1/recommendations/{recommendation_id}

# Approve recommendation (generates Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Retrieve generated Terraform
GET /api/v1/recommendations/{recommendation_id}/terraform
```

See our [API Documentation](../api-reference.md) for complete reference.

## IAM Permissions Required

JetScale requires the following IAM permissions for ElastiCache optimization:

**Read permissions (for analysis):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "elasticache:DescribeCacheClusters",
        "elasticache:DescribeReplicationGroups",
        "elasticache:DescribeCacheParameters",
        "elasticache:DescribeCacheEngineVersions",
        "elasticache:ListTagsForResource",
        "cloudwatch:GetMetricStatistics",
        "cloudwatch:GetMetricData",
        "ce:GetCostAndUsage"
      ],
      "Resource": "*"
    }
  ]
}
```

**Note:** JetScale does not require write permissions. All changes are implemented via generated Terraform code that you review and apply through your existing deployment processes.

## Support

Need help with ElastiCache optimization?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: [FAQ](../faq.md)
- **GitHub Issues**: [Report a problem](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

**Related Documentation:**
- [RDS Optimization](rds.md)
- [EC2 Optimization](ec2.md)
- [EBS Optimization](ebs.md)
