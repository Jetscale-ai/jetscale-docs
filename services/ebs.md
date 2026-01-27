# EBS Optimization

JetScale provides AI-powered cost optimization for Amazon EBS (Elastic Block Store) volumes. Our specialized agents analyze your storage workloads to identify volume type optimization, IOPS/throughput tuning, and unused volume cleanup opportunities.

## Overview

JetScale optimizes EBS volumes by analyzing:
- **Volume utilization**: IOPS and throughput usage patterns
- **Volume type efficiency**: Cost-effectiveness of current volume type
- **Attachment status**: Identify unused, unattached volumes
- **Cost analysis**: Current spending vs. optimal configuration

## Supported EBS Types

### EBS Volumes

JetScale optimizes EBS volumes attached to EC2 instances or unattached volumes in your account.

**What We Optimize:**
- **Volume type migration**: gp2 → gp3 (20% cost savings), io1 → io2 (better durability)
- **IOPS optimization**: Right-size provisioned IOPS based on actual usage
- **Throughput optimization**: Adjust gp3 throughput to match actual needs
- **Delete unattached volumes**: Remove unused volumes for 100% savings

**Supported Volume Types:**
- **gp3** (General Purpose SSD) - Latest generation, best value
- **gp2** (General Purpose SSD) - Previous generation
- **io2** (Provisioned IOPS SSD) - High performance, 99.999% durability
- **io1** (Provisioned IOPS SSD) - Older generation
- **st1** (Throughput Optimized HDD) - Low-cost, throughput-intensive
- **sc1** (Cold HDD) - Lowest cost, infrequent access

## EBS Volume Types

### General Purpose SSD (gp3)

**Best for:** Most workloads
- **Baseline performance**: 3,000 IOPS, 125 MB/s throughput
- **Scalable**: Up to 16,000 IOPS, 1,000 MB/s throughput
- **Pricing**: Pay for storage + optional additional IOPS/throughput
- **Cost advantage**: ~20% cheaper than gp2 for equivalent performance

**Use cases:**
- Boot volumes
- Virtual desktops
- Development and test environments
- Low-latency interactive applications

### General Purpose SSD (gp2)

**Legacy volume type**
- **Performance**: 3 IOPS per GB (minimum 100 IOPS, maximum 16,000 IOPS)
- **Burst**: Can burst to 3,000 IOPS using burst credits
- **Migration recommended**: Almost always better to migrate to gp3

**Why migrate to gp3:**
- 20% cost savings on storage
- Customize IOPS independently of size
- No burst credit management needed

### Provisioned IOPS SSD (io2)

**Best for:** Mission-critical, I/O-intensive workloads
- **Performance**: Up to 64,000 IOPS (256,000 with Block Express)
- **Durability**: 99.999% (vs. 99.8-99.9% for gp3)
- **Latency**: Sub-millisecond latency
- **Pricing**: Pay for storage + provisioned IOPS

**Use cases:**
- Large relational databases (Oracle, SAP HANA)
- Mission-critical applications requiring highest durability
- Applications needing sustained IOPS performance

### Provisioned IOPS SSD (io1)

**Legacy high-performance volume**
- **Performance**: Up to 64,000 IOPS
- **Durability**: 99.8-99.9% (lower than io2)
- **Migration recommended**: Migrate to io2 for better durability and same cost

### Throughput Optimized HDD (st1)

**Best for:** Throughput-intensive workloads
- **Performance**: 500 MB/s max throughput, 500 IOPS max
- **Pricing**: Lower cost than SSD options
- **Cannot be boot volume**

**Use cases:**
- Big data processing
- Data warehouses
- Log processing
- Large sequential I/O workloads

### Cold HDD (sc1)

**Best for:** Infrequently accessed data
- **Performance**: 250 MB/s max throughput, 250 IOPS max
- **Pricing**: Lowest cost per GB
- **Cannot be boot volume**

**Use cases:**
- Archival storage
- Infrequently accessed data
- Cost-optimized storage for large datasets

## How JetScale Optimizes EBS

### 1. Data Collection

JetScale analyzes multiple data sources:

**CloudWatch Metrics** (14-day rolling window):
- `VolumeReadOps` / `VolumeWriteOps` - IOPS usage
- `VolumeReadBytes` / `VolumeWriteBytes` - Throughput usage
- `VolumeThroughputPercentage` - Percentage of provisioned throughput used
- `VolumeConsumedReadWriteOps` - Total consumed IOPS (gp2/gp3)
- `BurstBalance` - Burst credit balance (gp2 only)

**EBS API Data**:
- Volume type (gp3, gp2, io2, io1, st1, sc1)
- Size in GB
- Provisioned IOPS
- Provisioned throughput (gp3 only)
- Attachment status (attached vs. unattached)
- Attached EC2 instance ID
- Volume state (available, in-use, creating, deleting)

**Cost Explorer Data**:
- Current monthly spend per volume
- Historical cost trends

**AWS Compute Optimizer** (if enabled):
- AWS-generated EBS recommendations
- Performance risk assessments

### 2. Analysis

Our AI agents perform deep analysis:

**Utilization Patterns:**
- Peak vs. average IOPS usage
- Peak vs. average throughput usage
- Time-of-day patterns
- Burst credit balance trends (gp2)

**Cost Modeling:**
- Current cost breakdown (storage + IOPS + throughput)
- Projected cost for alternative volume types
- Savings from IOPS/throughput reduction
- Cost of unattached volumes

**Safety Checks:**
- Verify volume is not attached before recommending deletion
- Ensure IOPS/throughput reductions don't impact performance
- Validate migration paths (gp2→gp3, io1→io2)

### 3. Recommendations

JetScale generates specific, actionable recommendations:

#### gp2 to gp3 Migration

**Example Recommendation:**
```
Resource: vol-0abc123 (web-server root volume)
Current: gp2, 100 GB
Recommended: gp3, 100 GB, 3,000 IOPS, 125 MB/s

Cost Impact:
- Current: $10/month
- Projected: $8/month
- Savings: $2/month (20%), $24/year

Performance Analysis:
- Current IOPS: 300 baseline (3 IOPS/GB)
- Peak IOPS usage: 250
- gp3 baseline: 3,000 IOPS (far exceeds current needs)

Risk: Very Low - Performance improvement with cost savings
Note: Can be done online without detaching volume
```

#### IOPS Right-Sizing (io2)

**Example Recommendation:**
```
Resource: vol-0def456 (database volume)
Current: io2, 500 GB, 10,000 IOPS
Recommended: io2, 500 GB, 5,000 IOPS

Cost Impact:
- Current: $40/month (storage) + $650/month (IOPS) = $690/month
- Projected: $40/month (storage) + $325/month (IOPS) = $365/month
- Savings: $325/month (47%), $3,900/year

Performance Analysis:
- Provisioned: 10,000 IOPS
- Peak usage: 3,200 IOPS
- Average usage: 1,800 IOPS
- Recommended: 5,000 IOPS (56% headroom above peak)

Risk: Low - Significant headroom above peak usage maintained
```

#### gp3 Throughput Optimization

**Example Recommendation:**
```
Resource: vol-0ghi789 (log storage volume)
Current: gp3, 1000 GB, 3,000 IOPS, 500 MB/s
Recommended: gp3, 1000 GB, 3,000 IOPS, 250 MB/s

Cost Impact:
- Current: $80/month (storage) + $40/month (extra throughput) = $120/month
- Projected: $80/month (storage) + $10/month (extra throughput) = $90/month
- Savings: $30/month (25%), $360/year

Performance Analysis:
- Provisioned: 500 MB/s
- Peak usage: 180 MB/s
- Average usage: 95 MB/s
- Recommended: 250 MB/s (39% headroom above peak)

Risk: Low - Adequate headroom above peak usage
Note: Throughput can be adjusted online
```

#### io1 to io2 Migration

**Example Recommendation:**
```
Resource: vol-0jkl012 (production database)
Current: io1, 1000 GB, 20,000 IOPS
Recommended: io2, 1000 GB, 20,000 IOPS

Cost Impact:
- Current: $125/month (storage) + $1,300/month (IOPS) = $1,425/month
- Projected: $125/month (storage) + $1,300/month (IOPS) = $1,425/month
- Savings: $0/month (same cost, better durability)

Performance Analysis:
- Same IOPS performance (20,000 IOPS)
- Same throughput characteristics
- Durability: 99.8-99.9% (io1) → 99.999% (io2)
- Better for mission-critical workloads

Risk: Very Low - Same cost, better durability
Benefit: 10-100x lower annual failure rate
```

#### Delete Unattached Volume

**Example Recommendation:**
```
Resource: vol-0mno345 (legacy-backup-volume)
Current: gp2, 500 GB, unattached
Recommended: Delete

Cost Impact:
- Current: $50/month
- Projected: $0/month
- Savings: $50/month (100%), $600/year

Volume Analysis:
- State: Available (not attached to any instance)
- Last attached: >180 days ago
- Created: 2 years ago
- Tags: No production tags found

Risk: Low - Volume is unattached and unused
Recommendation: Verify with team before deletion, create snapshot if needed
```

### 4. Terraform Generation

For each recommendation, JetScale generates production-ready Terraform code:

**Example: gp2 to gp3 Migration**

```hcl
# EBS Volume Optimization: gp2 → gp3
# Generated by JetScale on 2024-01-15
# Recommendation ID: rec_ebs_001

resource "aws_ebs_volume" "web_server_root" {
  availability_zone = "us-east-1a"

  # Previous: gp2, 100 GB ($10/month)
  # Optimized: gp3, 100 GB ($8/month)
  # Cost Reduction: 20% ($2/month, $24/year)
  #
  # Performance improvement:
  # - gp2: 300 IOPS baseline (3 IOPS/GB)
  # - gp3: 3,000 IOPS baseline (10x improvement)
  # - Peak usage: 250 IOPS (well within gp3 baseline)
  type = "gp3"
  size = 100

  # gp3 baseline performance (no additional cost)
  iops       = 3000
  throughput = 125

  encrypted  = true
  kms_key_id = var.kms_key_id

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"     = "true"
      "jetscale:recommendation" = "rec_ebs_001"
      "jetscale:previous_type"  = "gp2"
    }
  )
}

# Attachment preserved
resource "aws_volume_attachment" "web_server_root" {
  device_name = "/dev/sda1"
  volume_id   = aws_ebs_volume.web_server_root.id
  instance_id = var.instance_id
}
```

**Example: IOPS Right-Sizing**

```hcl
# EBS Volume IOPS Optimization
# Generated by JetScale on 2024-01-15

resource "aws_ebs_volume" "database_volume" {
  availability_zone = "us-east-1b"

  type = "io2"
  size = 500

  # Previous: 10,000 IOPS ($650/month for IOPS)
  # Optimized: 5,000 IOPS ($325/month for IOPS)
  # Cost Reduction: 50% on IOPS cost ($325/month, $3,900/year)
  #
  # Usage analysis:
  # - Peak IOPS: 3,200
  # - Average IOPS: 1,800
  # - Recommended: 5,000 IOPS (56% headroom above peak)
  iops = 5000

  encrypted  = true
  kms_key_id = var.kms_key_id

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"      = "true"
      "jetscale:previous_iops"  = "10000"
    }
  )
}
```

## Best Practices

### Testing Strategy

**Pre-Production Testing:**
1. Test volume modifications in dev/staging first
2. Monitor performance for 48-72 hours
3. Validate application I/O patterns
4. Check database query performance (if applicable)

**Production Rollout:**
1. Schedule changes during maintenance windows
2. Volume modifications can be done online (no downtime)
3. Some modifications may cause performance impact during migration
4. Monitor CloudWatch metrics actively during change

### Volume Type Migration

**gp2 → gp3 Migration:**
- Can be done online without detaching
- May experience brief performance impact during migration
- Almost always recommended (20% savings, better performance)

**io1 → io2 Migration:**
- Can be done online
- Same cost, better durability (99.999% vs. 99.8-99.9%)
- Strongly recommended for mission-critical workloads

### Unattached Volume Cleanup

**Before deleting unattached volumes:**
1. Verify volume is truly unused
2. Check tags for ownership information
3. Create snapshot if data might be needed
4. Confirm with application team
5. Wait 30 days after snapshot creation before deletion

**Snapshot strategy:**
```bash
# Create snapshot before deletion
aws ec2 create-snapshot \
  --volume-id vol-0abc123 \
  --description "Backup before deletion - Legacy backup volume"

# Wait for snapshot completion
aws ec2 wait snapshot-completed \
  --snapshot-ids snap-0xyz789

# Delete volume after 30-day retention period
aws ec2 delete-volume --volume-id vol-0abc123
```

## Common Optimization Patterns

### Pattern 1: Legacy gp2 Volumes

**Symptoms:**
- Using gp2 volumes
- gp3 available in region
- No special burst requirements

**JetScale Recommendation:**
- Migrate all gp2 → gp3
- Typical savings: 20% on storage costs
- Performance improvement: Higher baseline IOPS (3,000 vs. 3 IOPS/GB)
- Risk: Very Low (can be done online)

### Pattern 2: Over-Provisioned IOPS

**Symptoms:**
- Using io2 or io1 volumes
- Peak IOPS usage < 50% of provisioned
- High IOPS costs relative to storage costs

**JetScale Recommendation:**
- Reduce provisioned IOPS to 2x peak usage
- Typical savings: 30-50% on IOPS costs
- Risk: Low (maintain 2x headroom)

### Pattern 3: Unattached Volumes Accumulation

**Symptoms:**
- Multiple unattached volumes in account
- Volumes from terminated instances
- Old backup volumes no longer needed

**JetScale Recommendation:**
- Create snapshots if needed
- Delete unattached volumes
- Typical savings: 100% on deleted volumes
- Risk: Low if properly verified

### Pattern 4: Unnecessary gp3 IOPS/Throughput

**Symptoms:**
- gp3 volumes with >3,000 IOPS or >125 MB/s throughput
- Actual usage below baseline performance
- Paying for unused performance

**JetScale Recommendation:**
- Reduce to baseline (3,000 IOPS, 125 MB/s)
- Typical savings: 10-30% on volume costs
- Risk: Very Low if usage is well below baseline

## Troubleshooting

### Recommendation Concerns

**Q: Will migrating from gp2 to gp3 cause downtime?**

A: No. EBS volume modifications are done online without detaching the volume. The volume remains available during the modification, though there may be brief performance impacts during the migration process.

**Q: How long does volume modification take?**

A: Volume modifications typically complete within minutes to hours depending on:
- Volume size
- Current I/O load
- Type of modification (type change, IOPS change, size change)

You can monitor progress via AWS Console or CLI.

**Q: Can I roll back a volume modification?**

A: You cannot directly roll back, but you can modify the volume again after the current modification completes. There's a cooldown period (typically 6 hours) between modifications.

**Q: What if I reduce IOPS too much?**

A: You can increase IOPS again after the modification completes. Monitor CloudWatch metrics for:
- `VolumeQueueLength` (should stay low)
- `VolumeThroughputPercentage` (should not hit 100%)
- Application latency metrics

### Performance Issues After Optimization

**Symptom: Increased application latency after IOPS reduction**

Possible causes:
- IOPS reduced below actual peak needs
- `VolumeQueueLength` increasing
- Application experiencing I/O waits

**Resolution:**
1. Check CloudWatch `VolumeReadOps` and `VolumeWriteOps`
2. Verify `VolumeQueueLength` metric
3. Increase IOPS if queue length consistently > 0
4. Allow 6-hour cooldown between modifications

**Symptom: gp2 burst credit depletion after migration**

Possible causes:
- Migrated to gp3 but workload still needs burst capability
- Baseline IOPS insufficient for workload

**Resolution:**
- gp3 doesn't use burst credits - performance is consistent
- If seeing performance issues, increase gp3 IOPS above 3,000 baseline
- Cost: $0.065 per provisioned IOPS per month

## Security Considerations

### Encryption

JetScale recommendations preserve:
- EBS encryption status
- KMS key associations
- Encryption in-transit settings

### Volume Attachments

- Attached volumes: Only type/IOPS/throughput modifications
- Unattached volumes: Deletion recommendations (with safety checks)
- Volume attachments preserved in Terraform

### Snapshots

Before deleting volumes:
- Always create snapshot if data might be needed
- Tag snapshots appropriately
- Set snapshot lifecycle policies
- Consider snapshot copies to other regions for DR

## Limitations

**Not Currently Supported:**
- Snapshot optimization (lifecycle policies, cross-region copies)
- Multi-attach volumes
- Volume size reduction (AWS limitation)
- Snapshot-to-volume restore optimization

**Volume Modification Limits:**
- 6-hour cooldown between modifications
- Cannot modify volumes <6 hours old
- Cannot decrease volume size (AWS limitation)

## API Integration

JetScale provides API access for programmatic optimization:

```bash
# List EBS recommendations
GET /api/v1/recommendations?resource_type=ebs

# Get specific recommendation details
GET /api/v1/recommendations/{recommendation_id}

# Approve recommendation (generates Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Retrieve generated Terraform
GET /api/v1/recommendations/{recommendation_id}/terraform
```

See our [API Documentation](../api-reference.md) for complete reference.

## Support

Need help with EBS optimization?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: [FAQ](../faq.md)
- **GitHub Issues**: [Report a problem](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

**Related Documentation:**
- [EC2 Optimization](ec2.md)
- [RDS Optimization](rds.md)
- [ElastiCache Optimization](elasticache.md)
