# Azure VM Optimization

JetScale provides AI-powered cost optimization for Azure Virtual Machines. Our specialized agents analyze your compute workloads to identify right-sizing opportunities, VM family migrations, and cost-effective alternatives.

## Overview

JetScale optimizes Azure VM resources by analyzing:
- **Instance utilization**: CPU, memory, disk, and network usage patterns
- **Cost analysis**: Current spending vs. optimal configuration
- **Workload characteristics**: Resource requirements and performance patterns
- **Pricing options**: Pay-as-you-go, Reserved Instances, Spot VMs

## Supported VM Types

### Standalone Virtual Machines

JetScale optimizes standalone Azure VMs that are not part of Virtual Machine Scale Sets (VMSS).

**VM Architecture:**
```
VirtualMachine (Optimization Target, Billable)
├── OSDisk - Billable
├── DataDisk - Billable (optimized separately)
└── DataDisk - Billable
```

**What We Optimize:**
- **VM size right-sizing**: Change to smaller/larger sizes based on actual usage
- **VM series migration**: Switch between series (B, D, E, F, etc.) based on workload characteristics
- **Generation upgrades**: Migrate to latest generation (v5, v6) for better price/performance
- **Delete unused**: Remove VMs with zero or near-zero activity (100% savings)

**Important:** JetScale currently optimizes standalone Azure VMs only. VMs managed by Virtual Machine Scale Sets, AKS, or other orchestration systems are not yet supported.

## VM Series and Families

JetScale considers all Azure VM series when optimizing:

### General Purpose (B, D)

**B-series** - Burstable VMs
- Use for: Dev/test, low-traffic web servers, small databases, microservices
- Best when: CPU usage is low most of the time with occasional bursts
- Caution: Based on CPU credits, monitor credit balance to avoid throttling
- Sizes: B1s, B1ms, B2s, B2ms, B4ms, B8ms, B12ms, B16ms, B20ms

**D-series** - Balanced compute/memory/network
- Use for: Web servers, application servers, medium databases, mixed workloads
- Latest generations: Dv5/Dsv5 > Dv4/Dsv4 > Dv3/Dsv3
- Available in: Standard HDD, Standard SSD, Premium SSD
- Sizes: D2-D96 (2-96 vCPUs)

### Compute Optimized (F)

**F-series** - High CPU-to-memory ratio
- Use for: Batch processing, application servers, web servers, analytics, gaming
- Best when: CPU utilization is consistently high, memory needs are moderate
- Latest generations: Fv2 (Intel), Fasv5/Famsv5 (AMD)
- Sizes: F2-F72 (2-72 vCPUs)

### Memory Optimized (E, M)

**E-series** - High memory-to-CPU ratio
- Use for: Relational databases, in-memory analytics, SAP applications, caching
- Best when: Memory utilization is high, CPU is moderate
- Latest generations: Ev5/Esv5 > Ev4/Esv4 > Ev3/Esv3
- Sizes: E2-E96 (2-96 vCPUs, up to 672 GB RAM)

**M-series** - Extreme memory (up to 4 TB RAM)
- Use for: Large SQL Server, SAP HANA, in-memory databases
- Cost premium: Significantly higher than E-series
- Mission-critical workloads requiring massive memory
- Sizes: M8ms - M416ms (8-416 vCPUs)

### Storage Optimized (L)

**L-series** - High disk throughput and I/O
- Use for: Big data, SQL databases, NoSQL databases, data warehousing
- NVMe-based local storage
- Best for: I/O-intensive workloads requiring low latency
- Sizes: L4s-L80s (4-80 vCPUs)

### Accelerated Computing (N)

**N-series** - GPU-enabled VMs
- Use for: AI/ML training and inference, graphics rendering, video processing
- Options: NC (NVIDIA Tesla), NV (NVIDIA Tesla M60), ND (NVIDIA Tesla P40/V100)
- Not typically recommended for general cost optimization (specialized workloads)

### VM Sizing Patterns

Sizes follow pattern: `{Series}{Version}{Size}`

Examples:
- **B2s**: B-series, 2 vCPUs, Standard disk support
- **D4s_v5**: D-series v5, 4 vCPUs, Premium SSD support
- **E8as_v5**: E-series v5 AMD, 8 vCPUs, Premium SSD support

**Size progression:**
- Standard: 1, 2, 4, 8, 16, 32, 64, 96+ vCPUs
- Burstable: 1, 2, 4, 8, 12, 16, 20 vCPUs

Each step typically doubles vCPU and memory.

## How JetScale Optimizes Azure VMs

### 1. Data Collection

JetScale analyzes multiple data sources:

**Azure Monitor Metrics** (14-day rolling window):
- `Percentage CPU` - VM CPU usage
- `Available Memory Bytes` - Available memory
- `Network In Total` / `Network Out Total` - Network throughput
- `Disk Read Bytes` / `Disk Write Bytes` - Disk I/O
- `Disk Read Operations/Sec` / `Disk Write Operations/Sec` - IOPS

**Azure Resource Manager Data**:
- VM size and state
- OS type (Windows/Linux)
- Creation time
- Location and availability zone
- Tags and resource groups

**Cost Management Data**:
- Current monthly spend per VM
- Historical cost trends
- Reserved Instance coverage

**Azure Advisor** (if available):
- Azure-generated right-sizing recommendations
- Performance risk assessments

### 2. Analysis

Our AI agents perform deep analysis:

**Utilization Patterns:**
- Peak vs. average CPU utilization
- Time-of-day patterns (identify idle periods)
- Day-of-week patterns (weekend vs. weekday load)
- Memory usage trends

**Workload Classification:**
- **Variable CPU**: Consider B-series (burstable)
- **Steady high CPU**: Consider F-series (compute optimized)
- **Balanced**: Consider D-series (general purpose)
- **Memory-intensive**: Consider E-series (memory optimized)

**Cost Modeling:**
- Current monthly cost
- Projected cost for alternative VM sizes
- Savings percentage and annual impact
- Reserved Instance opportunities

**Risk Evaluation:**
- Performance degradation probability
- Headroom calculation (buffer above peak usage)
- Memory safety checks
- Network and storage capacity validation

### 3. Recommendations

JetScale generates specific, actionable recommendations:

#### VM Right-Sizing

**Example Recommendation:**
```
Resource: web-server-prod-01
Current: Standard_D16s_v5 (16 vCPUs, 64 GB RAM)
Recommended: Standard_D8s_v5 (8 vCPUs, 32 GB RAM)

Cost Impact:
- Current: $550/month
- Projected: $275/month
- Savings: $275/month (50%), $3,300/year

Performance Analysis:
- Average CPU: 18%
- Peak CPU: 32%
- Average Memory: 35%
- Recommended provides 2x headroom above peak

Risk: Low - Ample headroom maintained
```

#### Series Migration

**Example Recommendation:**
```
Resource: batch-processor-02
Current: Standard_D8s_v5 (8 vCPUs, 32 GB RAM)
Recommended: Standard_F8s_v2 (8 vCPUs, 16 GB RAM)

Cost Impact:
- Current: $275/month
- Projected: $233/month
- Savings: $42/month (15%), $504/year

Workload Analysis:
- Average CPU: 75%
- Peak CPU: 92%
- Memory usage: Low (28%)
- Workload type: Compute-intensive, memory underutilized

Recommendation: Migrate to F-series (compute optimized)
Risk: Very Low - CPU-bound workload, memory headroom adequate
```

#### Generation Upgrade

**Example Recommendation:**
```
Resource: api-server-01
Current: Standard_D4s_v3 (4 vCPUs, 16 GB RAM)
Recommended: Standard_D4s_v5 (4 vCPUs, 16 GB RAM)

Cost Impact:
- Current: $175/month
- Projected: $165/month
- Savings: $10/month (6%), $120/year

Performance Analysis:
- v5 offers better CPU performance (Intel Xeon Platinum 8370C)
- Same specifications, lower cost
- Better network performance (up to 12.5 Gbps)
- Newer generation efficiency

Risk: Very Low - Same series, newer generation
```

#### Delete Unused VM

**Example Recommendation:**
```
Resource: legacy-test-vm
Current: Standard_B2s (2 vCPUs, 4 GB RAM)
Recommended: Delete/Deallocate

Cost Impact:
- Current: $30/month
- Projected: $0/month
- Savings: $30/month (100%), $360/year

Usage Analysis:
- Average CPU: 0.3%
- Network activity: Near zero
- Created: 14 months ago
- Tags indicate: test environment

Risk: Very Low - Appears unused
Recommendation: Verify with application team before deletion
```

### 4. Terraform Generation

For each recommendation, JetScale generates production-ready Terraform code:

**Example: VM Right-Sizing**

```hcl
# Azure VM Optimization
# Generated by JetScale on 2024-01-15
# Recommendation ID: rec_azurevm_001

resource "azurerm_linux_virtual_machine" "web_server_prod_01" {
  name                = "web-server-prod-01"
  resource_group_name = var.resource_group_name
  location            = var.location

  # Previous: Standard_D16s_v5 ($550/month)
  # Optimized: Standard_D8s_v5 ($275/month)
  # Cost Reduction: 50% ($275/month, $3,300/year)
  #
  # Justification:
  # - Average CPU utilization: 18%
  # - Peak CPU utilization: 32%
  # - Average memory utilization: 35%
  # - New size provides 2x headroom above peak usage
  size = "Standard_D8s_v5"

  admin_username = var.admin_username

  admin_ssh_key {
    username   = var.admin_username
    public_key = var.ssh_public_key
  }

  network_interface_ids = [
    azurerm_network_interface.web_server_nic.id,
  ]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
    disk_size_gb         = 128
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-focal"
    sku       = "20_04-lts-gen2"
    version   = "latest"
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"      = "true"
      "jetscale:recommendation" = "rec_azurevm_001"
      "jetscale:previous_size"  = "Standard_D16s_v5"
    }
  )

  lifecycle {
    create_before_destroy = true
  }
}
```

**Example: Series Migration**

```hcl
# Azure VM Series Migration
# Generated by JetScale on 2024-01-15

resource "azurerm_linux_virtual_machine" "batch_processor_02" {
  name                = "batch-processor-02"
  resource_group_name = var.resource_group_name
  location            = var.location

  # Previous: Standard_D8s_v5 (General Purpose, $275/month)
  # Optimized: Standard_F8s_v2 (Compute Optimized, $233/month)
  # Cost Reduction: 15% ($42/month, $504/year)
  #
  # Workload Analysis:
  # - Average CPU: 75%, Peak CPU: 92%
  # - Memory usage: 28% (underutilized)
  # - Workload type: Compute-intensive
  # - F-series provides higher CPU-to-memory ratio
  size = "Standard_F8s_v2"

  admin_username = var.admin_username

  admin_ssh_key {
    username   = var.admin_username
    public_key = var.ssh_public_key
  }

  network_interface_ids = [
    azurerm_network_interface.batch_processor_nic.id,
  ]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
    disk_size_gb         = 256
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-focal"
    sku       = "20_04-lts-gen2"
    version   = "latest"
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"     = "true"
      "jetscale:series"        = "compute-optimized"
      "jetscale:previous_size" = "Standard_D8s_v5"
    }
  )

  lifecycle {
    create_before_destroy = true
  }
}
```

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

### Reserved Instances & Savings Plans

While JetScale doesn't currently generate RI recommendations, consider them after right-sizing:

**Best practice:**
1. Right-size VMs with JetScale first
2. Run optimized configuration for 30-60 days
3. Purchase Reserved Instances for stable workloads (1-year or 3-year)
4. Never reserve over-sized VMs

**Azure Reserved VM Instances:**
- Up to 72% savings vs. pay-as-you-go
- 1-year or 3-year commitment
- Size flexibility within same VM series
- Payment options: All upfront, Partial upfront, Monthly

### Spot VMs

For non-critical workloads, consider Azure Spot VMs:
- Up to 90% savings vs. pay-as-you-go
- Best for: Batch processing, dev/test, stateless applications
- Risk: Can be evicted with 30-second notice when Azure needs capacity
- JetScale identifies candidates for Spot migration (roadmap feature)

## Common Optimization Patterns

### Pattern 1: Over-Provisioned VMs

**Symptoms:**
- CPU utilization < 20% average
- Memory utilization < 30%
- VM running 24/7 with consistent low usage

**JetScale Recommendation:**
- Downsize by 1-2 VM sizes (e.g., D16 → D8)
- Typical savings: 40-50%
- Risk: Low (2-3x headroom maintained)

### Pattern 2: Wrong VM Series

**Symptoms:**
- Using D-series with consistently high CPU (>70%)
- Using E-series with low memory usage (<40%)
- Using F-series with high memory needs

**JetScale Recommendation:**
- Migrate to appropriate series (D → F for CPU, E → D for balanced)
- Typical savings: 10-30%
- Risk: Low to Medium (verify workload characteristics)

### Pattern 3: Legacy Generations

**Symptoms:**
- Using v3 or v4 VMs
- Newer generations available (v5, v6)
- Same cost or lower for better performance

**JetScale Recommendation:**
- Upgrade to latest generation (Dv3 → Dv5, Ev4 → Ev5)
- Typical savings: 5-15% with better performance
- Risk: Very Low (same series, newer hardware)

### Pattern 4: Idle Development VMs

**Symptoms:**
- Dev/test VMs running 24/7
- Used only during business hours
- Low CPU usage overnight and weekends

**JetScale Recommendation:**
- Implement auto-shutdown schedules (Azure feature)
- Downsize to B-series for burstable workloads
- Typical savings: 50-70% combined approach
- Risk: Very Low (non-production environment)

## Troubleshooting

### Recommendation Concerns

**Q: Will downsizing my VM impact performance?**

A: JetScale maintains 2-3x headroom above peak usage. We analyze:
- 99th percentile (P99) CPU utilization over 14 days
- Peak memory usage
- Network saturation points
- Historical spike patterns

Recommendations only proceed if performance risk is Low or Very Low.

**Q: What if I don't have memory metrics?**

A: Azure Monitor provides memory metrics by default for all VMs. If metrics are missing:
1. Verify Azure Monitor agent is installed
2. Check diagnostics settings are enabled
3. Wait 14 days for sufficient historical data

**Q: Can I test without downtime?**

A: Yes! Recommended approach:
1. Use `create_before_destroy = true` in Terraform
2. New VM launches with optimized size
3. Health check passes → traffic shifts to new VM
4. Old VM automatically terminated
5. Typical downtime: <30 seconds during cutover

### Performance Issues After Optimization

**Symptom: Increased latency or response times**

Possible causes:
- Insufficient CPU for peak loads
- Memory pressure
- Network throughput bottleneck

**Resolution:**
1. Check Azure Monitor CPU, memory, network metrics
2. Compare current vs. previous VM performance
3. If needed, increase VM size by one step
4. Simple rollback: revert Terraform to previous state

**Symptom: B-series CPU credit exhaustion**

Possible causes:
- Migrated from standard VM to burstable
- CPU usage higher than B-series baseline
- Insufficient CPU credits for workload pattern

**Resolution:**
1. Check `CPU Credits Remaining` metric
2. If consistently depleted, migrate to D/F-series
3. Consider B-series with higher baseline (e.g., B4ms → B8ms)

## Security Considerations

### VM Configuration

JetScale recommendations preserve:
- Virtual network and subnet associations
- Network Security Groups (NSGs)
- Managed identities
- Key vault references
- Disk encryption settings
- Boot diagnostics configuration

### Image Compatibility

- Uses same OS image/SKU as original VM
- Preserves custom images and configurations
- Maintains OS disk and data disk attachments
- No changes to authentication methods

### Lifecycle Management

Use Terraform lifecycle rules:
```hcl
lifecycle {
  create_before_destroy = true
  prevent_destroy       = var.is_production
}
```

## Limitations

**Not Currently Supported:**
- Virtual Machine Scale Sets (VMSS) optimization
- Spot VM recommendations
- Reserved Instance purchase recommendations
- Auto-shutdown scheduling
- AKS node pool optimization
- Multi-VM orchestration

**Roadmap:**
These features are planned for future releases. Currently, JetScale focuses on standalone Azure VM optimization where immediate cost savings can be achieved through right-sizing and migration.

## API Integration

JetScale provides API access for programmatic optimization:

```bash
# List Azure VM recommendations
GET /api/v1/recommendations?resource_type=azure-vm

# Get specific recommendation details
GET /api/v1/recommendations/{recommendation_id}

# Approve recommendation (generates Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Retrieve generated Terraform
GET /api/v1/recommendations/{recommendation_id}/terraform
```

See our [API Documentation](../api-reference.md) for complete reference.

## Support

Need help with Azure VM optimization?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: [FAQ](../faq.md)
- **GitHub Issues**: [Report a problem](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

**Related Documentation:**
- [Azure SQL Optimization](azure-sql.md)
- [EC2 Optimization](ec2.md)
