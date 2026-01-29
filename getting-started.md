# Getting Started with JetScale

> Start saving on cloud costs in 5 minutes

## Overview

JetScale is a SaaS platform that automatically discovers cost-saving opportunities in your cloud infrastructure. This guide will walk you through connecting your first cloud account and reviewing your first recommendations.

**Time to first savings: 5 minutes** ⏱️

---

## Step 1: Sign Up for JetScale

**Create Your Account:**
1. Visit [app.jetscale.ai/signup](https://app.jetscale.ai/signup)
2. Enter your email and create a password
3. Verify your email address
4. Complete your profile (company name, role)

**Free Trial:**
- 30 days of full platform access
- Up to $10,000 in identified savings
- No credit card required
- Cancel anytime

---

## Step 2: Connect Your Cloud Account

Choose your cloud provider and follow the setup guide:

### AWS (5 minutes)

**Quick Steps:**
1. Log in to your AWS account
2. Create a cross-account IAM role for JetScale
3. Copy the Role ARN and External ID
4. Paste into JetScale dashboard
5. Click "Connect" and verify access

[Detailed AWS Setup Guide →](aws-setup.md)

**Permissions:** Read-only access to EC2, RDS, EBS, ElastiCache, Cost Explorer

---

### Azure (5 minutes)

**Quick Steps:**
1. Log in to Azure portal
2. Create a Service Principal for JetScale
3. Assign Reader role to your subscription
4. Copy Tenant ID, Subscription ID, and credentials
5. Paste into JetScale dashboard
6. Click "Connect" and verify access

[Detailed Azure Setup Guide →](azure-setup.md)

**Permissions:** Read-only access to Virtual Machines, SQL, Disks, Cache, Cost Management

---

## Step 3: Wait for Discovery (2-5 minutes)

**What's Happening:**
- JetScale scans all regions in your account
- Catalogs compute, database, storage, and caching resources
- Pulls 30 days of usage metrics from CloudWatch/Azure Monitor
- AI analyzes usage patterns and identifies opportunities

**Progress indicators:**
- **Discovering resources** → Finding all cloud resources
- **Analyzing usage** → Collecting 30 days of metrics
- **Generating recommendations** → AI validation in progress
- **Ready** → Recommendations available for review

---

## Step 4: Review Recommendations

**Dashboard Overview:**

You'll see recommendations organized by:
- **Potential savings** (highest to lowest)
- **Risk level** (Low, Medium, High)
- **Resource type** (EC2, RDS, EBS, etc.)

**Recommendation Card:**

Each card shows:
- **Resource name** and type
- **Estimated monthly savings**
- **Risk level** with color coding
- **Quick summary** of the change

**Clicking "View Details" shows:**
- 30-day usage charts (CPU, memory, network)
- Current vs recommended configuration
- Detailed savings calculation
- Performance impact assessment
- Implementation steps and rollback instructions

---

## Step 5: Approve Recommendations

**Select Recommendations:**
1. Review low-risk recommendations first (green badges)
2. Check usage charts to validate AI analysis
3. Select recommendations you want to implement
4. Click "Approve Selected" button

**Bulk Approval:**
- Use filters to find specific types (e.g., "EC2 right-sizing")
- Select multiple recommendations at once
- Preview combined monthly savings
- Approve batch deployment

---

## Step 6: Deploy Changes

**Choose Your Integration:**

### Option 1: GitHub Pull Request (Recommended)

**Setup (one-time):**
1. Connect your GitHub account in Settings
2. Select target repository for Terraform code
3. Configure branch name and reviewers

**Deployment:**
1. JetScale creates a new branch
2. Commits Terraform code for approved recommendations
3. Opens pull request with full documentation
4. Your team reviews and approves
5. Merge PR and apply Terraform changes

[GitHub Integration Guide →](integrations/github.md)

---

### Option 2: Bitbucket Pull Request

Same workflow as GitHub, but for Bitbucket repositories.

[Bitbucket Integration Guide →](integrations/bitbucket.md)

---

### Option 3: Manual Download

**Download Terraform files:**
1. Click "Download Terraform" button
2. Receive ZIP file with all configurations
3. Extract and review files locally
4. Apply using your existing Terraform workflow

```bash
# Example manual deployment
unzip jetscale-recommendations.zip
cd jetscale-recommendations/
terraform init
terraform plan
terraform apply
```

---

## Step 7: Track Savings

**Savings Dashboard:**

After deployment, monitor:
- **Actual savings** vs projected savings
- **Cumulative savings** over time
- **Performance metrics** post-change
- **ROI calculations** for executive reporting

**Continuous Discovery:**
- JetScale scans daily for new opportunities
- Alerts when high-value recommendations appear
- Tracks usage trends to identify growing waste
- Suggests optimizations as your infrastructure evolves

---

## Integrations (Optional)

### Jira Integration

**Automatic ticket creation:**
- Creates Jira tickets for each recommendation batch
- Tracks savings and implementation status
- Links to pull requests and JetScale dashboard
- Updates status when PRs are merged

[Jira Integration Guide →](integrations/jira.md)

---

### Slack Integration

**Real-time notifications:**
- New recommendations available
- Savings milestones reached
- Deployment status updates
- Weekly summary reports

[Slack Integration Guide →](integrations/slack.md)

---

## Team Setup (Optional)

**Invite Team Members:**
1. Go to Settings → Team
2. Click "Invite Member"
3. Enter email and select role:
   - **Admin**: Full access, can approve recommendations
   - **Editor**: Can review and approve recommendations
   - **Viewer**: Read-only access to dashboard

**Role Permissions:**

| Permission | Admin | Editor | Viewer |
|------------|-------|--------|--------|
| View recommendations | ✓ | ✓ | ✓ |
| Approve recommendations | ✓ | ✓ | - |
| Configure integrations | ✓ | - | - |
| Manage team members | ✓ | - | - |
| Connect cloud accounts | ✓ | - | - |

---

## Settings & Preferences

**Customize JetScale:**

**Notification Settings:**
- Email frequency (daily, weekly, monthly)
- Slack channel for notifications
- Savings threshold for alerts ($100, $500, $1000)

**Risk Tolerance:**
- **Conservative**: Only show low-risk recommendations
- **Balanced**: Show low and medium-risk (default)
- **Aggressive**: Show all recommendations including high-risk

**Approval Workflow:**
- **Manual approval**: Review each recommendation individually
- **Auto-approve low-risk**: Automatically approve recommendations < $100/month
- **Custom rules**: Define approval criteria (e.g., "auto-approve EC2 right-sizing")

**Exclusions:**
- Exclude specific resources by tag (e.g., "JetScaleIgnore=true")
- Exclude resource types (e.g., "never touch production RDS")
- Exclude entire accounts or regions

---

## Best Practices

### Start with Low-Risk Recommendations

**Week 1:** Approve low-risk, high-savings recommendations:
- Storage optimization (gp2→gp3 migrations)
- Cleanup unused snapshots and volumes
- Right-size obviously over-provisioned resources

**Week 2-4:** Expand to medium-risk recommendations:
- EC2 instance right-sizing with proper headroom
- Reserved Instance purchases for stable workloads
- Database instance optimization

**Month 2+:** Consider higher-risk optimizations:
- Graviton migrations (requires testing)
- Scheduled shutdown of non-production resources
- Auto Scaling Group optimizations

### Test in Non-Production First

**Safe rollout strategy:**
1. Test recommendations in dev/staging environments first
2. Monitor performance for 1-2 weeks
3. Apply same optimizations to production
4. Gradually expand to more resource types

### Monitor Performance Post-Deployment

**Key metrics to watch:**
- CPU and memory utilization (should stay below 80%)
- Response times and latency (should remain stable)
- Error rates (should not increase)
- Application logs (watch for resource constraints)

**If issues arise:**
- Rollback using provided Terraform instructions
- Contact support: [support@jetscale.ai](mailto:support@jetscale.ai)
- JetScale tracks outcomes to improve future recommendations

---

## Common Questions

### How long until I see savings?

**Immediate savings:**
- Storage optimizations apply within hours
- Instance right-sizing takes 5-10 minutes (create-before-destroy)

**Savings on next bill:**
- Most optimizations reduce your next monthly bill
- Reserved Instances show savings immediately after purchase

### What if I don't like a recommendation?

**You're in control:**
- Reject any recommendation (won't be shown again)
- Defer recommendations for later review
- Provide feedback to improve future suggestions

### Can I undo changes?

**Yes, easily:**
- All Terraform code includes rollback instructions
- Most changes can be reverted in-place
- JetScale provides step-by-step rollback guides
- Support team available to assist

### How accurate are savings estimates?

**Very accurate:**
- 98% accuracy within ±5% variance
- Based on 30 days of actual usage data
- Includes all cost factors (compute, storage, data transfer)
- Accounts for Reserved Instance credits

---

## Need Help?

**Support Channels:**
- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Live Chat**: Available in dashboard (bottom-right corner)
- **Documentation**: [docs.jetscale.ai](https://docs.jetscale.ai)
- **Schedule Demo**: [jetscale.ai/demo](https://jetscale.ai/demo)

**Response Times:**
- **Free Trial**: 24-hour response
- **Pro Plan**: 8-hour response
- **Enterprise**: 2-hour response with dedicated support

---

## Next Steps

**Learn More:**
- [How JetScale Works](how-it-works.md) - Deep dive into the platform
- [AI Analysis](ai-analysis.md) - How our AI validates safety
- [Supported Services](services/README.md) - What JetScale optimizes
- [FAQ](faq.md) - Common questions answered

**Ready to save?**
- [Start Free Trial →](https://app.jetscale.ai/signup)
- [Schedule Demo →](https://jetscale.ai/demo)

---

*Last Updated: January 29, 2025*
