# Supported Services

JetScale provides AI-powered cost optimization for the most critical cloud services across AWS and Azure.

---

## AWS Services

### Amazon RDS (Relational Database Service)
Optimize database instance sizes, storage configurations, and Reserved Instance purchases.

**Optimization Areas:**
- Instance class right-sizing
- Storage type optimization (gp2 → gp3)
- Multi-AZ configuration review
- Reserved Instance recommendations
- Snapshot cleanup

[Learn More →](services/rds.md)

---

### Amazon EC2 (Elastic Compute Cloud)
Right-size compute instances, identify idle resources, and optimize instance purchasing.

**Optimization Areas:**
- Instance type and size recommendations
- Idle instance detection
- Graviton migration opportunities
- Reserved Instance and Savings Plans
- Stop/start scheduling

[Learn More →](services/ec2.md)

---

### Amazon EBS (Elastic Block Store)
Optimize storage volumes with type recommendations and capacity adjustments.

**Optimization Areas:**
- Volume type optimization (gp2 → gp3)
- IOPS and throughput tuning
- Snapshot lifecycle management
- Unused volume detection
- Cold storage migration

[Learn More →](services/ebs.md)

---

### Amazon ElastiCache
Optimize Redis and Memcached clusters for cost and performance.

**Optimization Areas:**
- Node type right-sizing
- Cluster configuration
- Reserved Node recommendations
- Multi-AZ review
- Eviction policy optimization

[Learn More →](services/elasticache.md)

### Amazon S3 (Simple Storage Service)
Optimize storage costs with tiering, lifecycle policies, encryption tuning, and network routing improvements.

**Optimization Areas:**
- Intelligent-Tiering enablement
- Lifecycle rule configuration (Standard → IA → Glacier → Deep Archive)
- Bucket Key for KMS cost reduction
- S3 Gateway Endpoints to eliminate NAT processing fees

[Learn More →](services/s3.md)

### Amazon EKS (Elastic Kubernetes Service)
Right-size node groups, migrate to Graviton, and optimize Spot capacity across your Kubernetes clusters.

**Optimization Areas:**
- Node type right-sizing per node group
- Graviton migration (ARM64) for 10-40% savings
- Spot capacity for fault-tolerant workloads (60-90% savings)
- Idle cluster and node group detection
- Per-node-group analysis with cluster-level aggregation

[Learn More →](services/eks.md)

## Azure Services

### Azure Virtual Machines
Right-size VMs, optimize instance types, and leverage reserved capacity.

**Optimization Areas:**
- VM size recommendations
- B-series burstable instances
- Azure Reserved VM Instances
- Spot instance opportunities
- Shutdown scheduling

[Learn More →](services/azure-vm.md)

---

### Azure SQL Database
Optimize database tiers, DTU allocation, and elastic pool configurations.

**Optimization Areas:**
- Service tier recommendations
- DTU and vCore sizing
- Elastic pool optimization
- Reserved capacity purchases
- Storage optimization

[Learn More →](services/azure-sql.md)

---

## Coming Soon

We're actively developing support for additional services:

### AWS
- **AWS Lambda** - Memory and timeout optimization
- **Amazon DynamoDB** - Capacity mode recommendations
- **Amazon ECS** - Container service right-sizing
- **Elastic Load Balancing** - Load balancer type optimization
- **Amazon CloudFront** - Distribution optimization

### Azure
- **Azure Blob Storage** - Storage tier optimization
- **Azure Functions** - Plan and memory optimization
- **Azure Cosmos DB** - Throughput and capacity recommendations
- **Azure Cache for Redis** - Tier and capacity optimization
- **Azure Kubernetes Service** - Node pool optimization
- **Azure App Service** - Plan tier recommendations

---

## How Service Analysis Works

For each supported service, JetScale:

1. **Discovers Resources**: Automatically identifies all instances across your cloud accounts
2. **Collects Metrics**: Gathers historical usage data (CPU, memory, disk, network)
3. **AI Analysis**: Specialized agents analyze patterns and identify optimization opportunities
4. **Validates Safety**: Ensures recommendations maintain performance and availability
5. **Generates Code**: Creates production-ready Terraform for implementation

---

## Optimization Criteria

Every recommendation is validated against:

- **Performance**: Maintains or improves current performance levels
- **Availability**: Preserves high-availability configurations
- **Headroom**: Includes buffer for traffic spikes and growth
- **Safety**: Easy rollback if needed
- **ROI**: Minimum 20% cost reduction threshold

---

## Support

Questions about supported services?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: [FAQ](faq.md)
- **Feature Requests**: [Request a service](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

*© 2025 JetScale, Inc. All rights reserved.*
