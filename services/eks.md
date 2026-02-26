# EKS Optimization

JetScale provides AI-powered cost optimization for Amazon EKS (Elastic Kubernetes Service) clusters. Our specialized agents analyze node group utilization, instance types, capacity configurations, and pricing to identify right-sizing, Graviton migration, and Spot optimization opportunities.

## Overview

JetScale optimizes EKS clusters by analyzing:
- **Node group utilization**: CPU, memory, network, and disk usage across all nodes
- **Instance type efficiency**: Whether current instance types match actual workload requirements
- **Capacity type**: ON_DEMAND vs. SPOT opportunities for fault-tolerant workloads
- **Cost analysis**: Control plane costs, node group costs, and optimization potential

## Supported EKS Types

### EKS Clusters

JetScale processes each node group within a cluster independently, generating per-node-group recommendations with an aggregated cluster-level view.

**Compute models supported:**
- **Managed Node Groups**: AWS-managed, explicit instance type configuration
- **Karpenter**: Dynamic instance selection with graceful Spot interruption handling
- **Self-Managed Nodes**: EC2 instances launched outside EKS managed node groups

**What We Optimize:**
- **Node type right-sizing**: Match instance types to actual workload needs
- **Graviton migration**: Switch to ARM-based instances for 10-40% savings
- **Spot optimization**: Move fault-tolerant workloads to Spot capacity for 60-90% savings
- **Delete unused**: Remove idle clusters or node groups for 100% savings

### EKS Node Groups

JetScale also optimizes standalone node groups with the same analysis and recommendation types as cluster-level processing.

## Instance Families

### General Purpose (M-series)
- **M7g** (Graviton3, latest): Best price-performance for general workloads
- **M6g** (Graviton2): Proven ARM-based option
- **M5/M5a** (Intel/AMD): x86 compatibility

### Compute Optimized (C-series)
- **C7g** (Graviton3): CPU-intensive containerized workloads
- **C6g** (Graviton2): Batch processing, CI/CD
- **C5/C5a** (Intel/AMD): x86 compute workloads

### Memory Optimized (R-series)
- **R7g** (Graviton3): In-memory caching, real-time analytics
- **R6g** (Graviton2): Memory-heavy containers
- **R5/R5a** (Intel/AMD): x86 memory workloads

### Burstable (T-series)
- **T4g** (Graviton2): Dev/test node groups, low-traffic services
- **T3/T3a** (Intel/AMD): Variable workloads with CPU credit model

## How JetScale Optimizes EKS

### 1. Data Collection

JetScale analyzes multiple data sources:

**CloudWatch Metrics** (1-year lookback):
- CPU utilization (average, maximum, percentiles) per node
- Memory utilization (average, maximum, percentiles) per node
- Network I/O (bytes in/out)
- Disk I/O (operations per second)
- Instance health status checks

Metrics are aggregated across all EC2 instances in a node group, with instance counts noted (e.g., "[3 instances] avg 25%, max 45%").

**EKS API Data**:
- Cluster Kubernetes version and support type
- Node group instance types and scaling configuration (min, max, desired)
- Capacity type (ON_DEMAND or SPOT)
- AMI type (AL2_x86_64, AL2_ARM_64, etc.)
- Labels, taints, and launch template configuration
- Auto Scaling Group linkage

**AWS Pricing Data**:
- On-Demand hourly rates per instance type
- Spot hourly rates (minimum and average)
- EKS control plane cost:
  - Standard support: ~$0.10/hour (~$73/month)
  - Extended support (Kubernetes 1.26 and earlier): ~$0.60/hour (~$438/month)

**Instance linking:**
- Managed Node Groups → EC2 instances via Auto Scaling Group name
- Karpenter nodes → Cluster via `aws:eks:cluster-name` tag, grouped by NodePool
- Self-managed nodes → Cluster via cluster name tag, grouped by instance type

### 2. Analysis

Our AI agents perform deep analysis per node group:

**Utilization Patterns:**
- Peak vs. average CPU and memory usage across all nodes
- Time-of-day and day-of-week patterns
- Node count vs. actual resource consumption

**Cost Modeling:**
- Current cost: `instance_hourly_rate x desired_size x 730 hours/month`
- Control plane cost included in cluster-level totals
- Spot pricing uses conservative minimum rates (not averages)

**Workload Classification:**
- Compute-heavy: High CPU, low memory utilization
- Memory-heavy: Low CPU, high memory utilization
- Balanced: Proportional CPU and memory usage
- Idle: Minimal utilization across all metrics

**Safety Checks:**
- Auto Mode clusters are skipped (AWS manages compute automatically)
- Memory reduction >25% requires memory utilization metrics
- CPU reduction >50% requires CPU utilization metrics
- Capacity reductions without metrics are rejected

### 3. Recommendations

JetScale generates recommendations per node group, then aggregates at the cluster level.

#### Right-Sizing

**Example Recommendation:**
```
Resource: production-cluster / api-node-group
Current: 3x m5.xlarge (ON_DEMAND)
  - 4 vCPU, 16 GiB memory per node
  - Monthly cost: $420.48

CloudWatch Analysis (3 nodes):
  - CPU: avg 22%, max 41%
  - Memory: avg 35%, max 52%

Recommended: 3x m5.large (ON_DEMAND)
  - 2 vCPU, 8 GiB memory per node
  - Monthly cost: $210.24

Cost Impact:
  - Savings: $210.24/month (50%), $2,522.88/year

Risk: Low - Peak CPU at 41% fits within 2 vCPU
  Memory at 52% of 16 GiB = 8.3 GiB, fits within 8 GiB with headroom
Implementation Effort: Medium - Rolling node group update required
Restart Needed: Yes (node replacement)
```

#### Graviton Migration

**Example Recommendation:**
```
Resource: web-cluster / frontend-nodes
Current: 4x m5.large (ON_DEMAND)
  - 2 vCPU, 8 GiB memory per node
  - Monthly cost: $280.32

Recommended: 4x m6g.large (ON_DEMAND)
  - 2 vCPU, 8 GiB memory per node (Graviton2, ARM64)
  - Monthly cost: $224.26

Cost Impact:
  - Savings: $56.06/month (20%), $672.72/year

Risk: Low - Containerized workloads are typically ARM-compatible
  Same vCPU and memory specifications
  Better price-performance on Graviton
Implementation Effort: Medium - Requires ARM64-compatible container images
Restart Needed: Yes (node replacement with new AMI)
```

#### Spot Optimization

**Example Recommendation:**
```
Resource: batch-cluster / worker-nodes
Current: 5x c5.xlarge (ON_DEMAND)
  - 4 vCPU, 8 GiB memory per node
  - Monthly cost: $620.50

Workload Analysis:
  - Batch processing, stateless
  - Fault-tolerant (jobs restart on failure)
  - No production-facing traffic

Recommended: 5x c5.xlarge (SPOT)
  - Same specs, Spot capacity
  - Monthly cost: $186.15 (conservative Spot minimum)

Cost Impact:
  - Savings: $434.35/month (70%), $5,212.20/year

Risk: Medium - Spot instances can be reclaimed with 2-minute notice
  Karpenter handles interruptions natively
  Batch jobs should be idempotent and restartable
Implementation Effort: Low (Karpenter) / Medium (Managed Node Groups)
Restart Needed: Yes (node replacement)
```

#### Delete Unused

**Example Recommendation:**
```
Resource: legacy-cluster / test-nodes
Current: 2x t3.medium (ON_DEMAND)
  - Monthly cost: $60.74

CloudWatch Analysis (2 nodes):
  - CPU: avg 0.5%, max 2%
  - Memory: avg 8%, max 12%
  - No network activity in 30+ days

Recommended: Delete
  - Savings: $60.74/month (100%), $728.88/year

Risk: Low - No meaningful workload detected
  Name does not contain "prod" or "production"
  Verify with team before deletion
```

#### Cluster-Level Aggregation

When multiple node groups are optimized:

```
Cluster: production-cluster
Control Plane: $73/month (Standard support)

Node Group Results:
1. api-nodes: Rightsize m5.xlarge → m5.large (-$210.24/month)
2. worker-nodes: MigrateToGraviton m5.large → m6g.large (-$56.06/month)
3. cache-nodes: NoRecommendation (already optimized)

Cluster Action: MultipleOptimizations
Total Cluster Savings: $266.30/month, $3,195.60/year
```

### 4. Terraform Generation

For each recommendation, JetScale generates production-ready Terraform code:

**Example: Node Group Right-Sizing**

```hcl
# EKS Node Group Optimization: Right-Sizing
# Generated by JetScale on 2024-01-15
# Recommendation ID: rec_eks_001

resource "aws_eks_node_group" "api_nodes" {
  cluster_name    = "production-cluster"
  node_group_name = "api-nodes"
  node_role_arn   = var.node_role_arn
  subnet_ids      = var.subnet_ids

  # Previous: m5.xlarge ($420.48/month)
  # Optimized: m5.large ($210.24/month)
  # Cost Reduction: 50% ($210.24/month, $2,522.88/year)
  #
  # Usage analysis:
  # - CPU: avg 22%, max 41% (fits within 2 vCPU)
  # - Memory: avg 35%, max 52% of 16 GiB = 8.3 GiB (fits within 8 GiB)
  instance_types = ["m5.large"]

  scaling_config {
    desired_size = 3
    max_size     = 5
    min_size     = 2
  }

  update_config {
    max_unavailable = 1
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"        = "true"
      "jetscale:recommendation"   = "rec_eks_001"
      "jetscale:previous_type"    = "m5.xlarge"
    }
  )
}
```

**Example: Graviton Migration**

```hcl
# EKS Node Group Optimization: Graviton Migration
# Generated by JetScale on 2024-01-15

resource "aws_eks_node_group" "frontend_nodes" {
  cluster_name    = "web-cluster"
  node_group_name = "frontend-nodes"
  node_role_arn   = var.node_role_arn
  subnet_ids      = var.subnet_ids

  # Previous: m5.large, x86_64 ($280.32/month)
  # Optimized: m6g.large, ARM64 ($224.26/month)
  # Cost Reduction: 20% ($56.06/month, $672.72/year)
  instance_types = ["m6g.large"]

  # ARM64 AMI required for Graviton instances
  ami_type = "AL2_ARM_64"

  scaling_config {
    desired_size = 4
    max_size     = 6
    min_size     = 2
  }

  update_config {
    max_unavailable = 1
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"        = "true"
      "jetscale:previous_type"    = "m5.large"
      "jetscale:migration_type"   = "graviton"
    }
  )
}
```

## Best Practices

### Testing Strategy

**Pre-Production Testing:**
1. Test instance type changes on non-production node groups first
2. Validate container image ARM64 compatibility before Graviton migration
3. Run Spot node groups alongside ON_DEMAND for a trial period
4. Monitor pod scheduling and resource requests/limits alignment

**Production Rollout:**
1. Use rolling updates (`max_unavailable = 1`) to minimize disruption
2. Monitor pod evictions and rescheduling during node replacement
3. Validate application health checks pass on new node types
4. Keep previous node group configuration for quick rollback

### Graviton Migration Checklist

Before migrating to Graviton (ARM64) instances:
1. Verify all container images have ARM64 or multi-arch builds
2. Check for x86-specific binary dependencies in containers
3. Update AMI type to `AL2_ARM_64` in node group configuration
4. Test with a small node group before migrating all nodes
5. Monitor application performance for 48-72 hours post-migration

### Spot Instance Guidelines

**Good candidates for Spot:**
- Batch processing and data pipelines
- CI/CD build workers
- Stateless microservices with multiple replicas
- Development and testing workloads
- Karpenter-managed nodes (handles interruptions natively)

**Avoid Spot for:**
- Single-node managed groups (no redundancy during reclaim)
- Stateful workloads (databases, persistent queues)
- Production-critical services with strict SLAs
- Workloads with long initialization times

### Capacity Planning

- JetScale uses conservative Spot pricing (minimum, not average) for cost estimates
- Node group scaling configuration (min/max/desired) is preserved in recommendations
- Control plane costs are included in cluster-level totals
- Extended support clusters (Kubernetes 1.26 and earlier) incur higher control plane costs (~$438/month vs ~$73/month)

## Common Optimization Patterns

### Pattern 1: Over-Provisioned Node Groups

**Symptoms:**
- CPU utilization consistently below 30%
- Memory utilization below 40%
- Nodes have significant unused capacity

**JetScale Recommendation:**
- Right-size to smaller instance types
- Typical savings: 30-50%
- Risk: Low (verify peak usage fits new capacity)

### Pattern 2: Graviton Migration Opportunity

**Symptoms:**
- Running x86 instances (m5, c5, r5)
- Containerized workloads (typically ARM-compatible)
- No x86-specific binary dependencies

**JetScale Recommendation:**
- Migrate to Graviton equivalents (m6g, c7g, r6g, t4g)
- Typical savings: 10-40%
- Risk: Low for most containerized workloads

### Pattern 3: ON_DEMAND Batch Workers

**Symptoms:**
- Batch processing or CI/CD node groups
- Fault-tolerant, stateless workloads
- Running on ON_DEMAND capacity

**JetScale Recommendation:**
- Switch to Spot capacity
- Typical savings: 60-90%
- Risk: Medium (Spot can be reclaimed, jobs must be restartable)

### Pattern 4: Legacy Kubernetes Version

**Symptoms:**
- Kubernetes version 1.26 or earlier
- Extended support charges (~$438/month control plane)
- Standard support clusters cost ~$73/month

**JetScale Observation:**
- Extended support adds ~$365/month to control plane costs
- Upgrading Kubernetes version reduces control plane costs by ~83%
- JetScale includes this cost in cluster-level totals for visibility

## Troubleshooting

### Recommendation Concerns

**Q: Will right-sizing cause application downtime?**

A: Node group updates use rolling replacement. With `max_unavailable = 1`, one node is replaced at a time. Pods are evicted and rescheduled on remaining nodes during the transition. Ensure your deployments have Pod Disruption Budgets (PDBs) configured for graceful handling.

**Q: How does JetScale handle Karpenter-managed nodes?**

A: JetScale discovers Karpenter nodes by grouping EC2 instances that share the same cluster tag but aren't part of a managed node group. These are grouped by NodePool tag and analyzed as synthetic node groups. Karpenter's native Spot interruption handling makes it a strong candidate for Spot optimization.

**Q: What if memory metrics are unavailable?**

A: If CloudWatch Container Insights is not enabled, JetScale won't have memory utilization data. In this case, recommendations that reduce memory capacity by more than 25% are rejected as a safety measure. Enabling Container Insights provides more accurate and aggressive optimization opportunities.

**Q: Can I roll back a node group change?**

A: Yes. Update the node group configuration back to the previous instance type and desired size. The rolling update process will replace nodes with the original configuration.

### Performance Issues After Optimization

**Symptom: Pods in Pending state after right-sizing**

Possible causes:
- New instance type has insufficient CPU or memory for pod resource requests
- Node capacity doesn't match pod scheduling requirements

**Resolution:**
1. Check pod resource requests against new node capacity
2. Verify `kubectl describe node` shows allocatable resources
3. Adjust pod requests/limits or increase node size if needed
4. Consider adding more nodes instead of larger nodes

**Symptom: Spot interruptions causing service disruption**

Possible causes:
- Single-replica deployments on Spot nodes
- No Pod Disruption Budget configured
- Long graceful shutdown periods

**Resolution:**
1. Run multiple replicas across nodes for redundancy
2. Configure Pod Disruption Budgets
3. Use Karpenter for automatic Spot interruption handling
4. Consider mixed ON_DEMAND + SPOT node groups for critical services

## Limitations

**Not Currently Supported:**
- EKS Auto Mode clusters (AWS manages compute)
- Fargate profile optimization
- Cluster autoscaler configuration tuning
- Pod-level resource optimization (requests/limits)
- Multi-cluster orchestration
- Reserved Instance recommendations for EKS nodes

**EKS-Specific Constraints:**
- All nodes in a managed node group must use the same instance type
- Graviton migration requires ARM64-compatible container images
- Spot optimization rejected for single-node managed groups
- CloudWatch Container Insights needed for memory metrics

## API Integration

JetScale provides API access for programmatic optimization:

```bash
# List EKS recommendations
GET /api/v1/recommendations?resource_type=eks

# Get specific recommendation details
GET /api/v1/recommendations/{recommendation_id}

# Approve recommendation (generates Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Retrieve generated Terraform
GET /api/v1/recommendations/{recommendation_id}/terraform
```

See our [API Documentation](../api-reference.md) for complete reference.

## Support

Need help with EKS optimization?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: [FAQ](../faq.md)
- **GitHub Issues**: [Report a problem](https://github.com/Jetscale-ai/jetscale-docs/issues)

**Related Documentation:**
- [EC2 Optimization](ec2.md)
- [RDS Optimization](rds.md)
- [EBS Optimization](ebs.md)
- [ElastiCache Optimization](elasticache.md)
- [S3 Optimization](s3.md)