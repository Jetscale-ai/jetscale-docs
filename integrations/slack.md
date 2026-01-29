# Slack Integration

JetScale integrates with Slack to deliver real-time cost optimization notifications directly to your team channels, keeping everyone informed about new recommendations, deployment status, and cost savings opportunities.

## Overview

The Slack integration allows JetScale to:
- Post notifications for new cost optimization recommendations
- Alert teams about significant savings opportunities
- Notify on deployment status updates
- Share cost savings achievements
- Provide interactive recommendation summaries

## Prerequisites

Before connecting Slack, you'll need:

1. **Slack Workspace** with admin or app installation permissions
2. **Channel Access** to receive JetScale notifications
3. **Workspace Permissions** to install apps

### Supported Workspace Types

- Slack Free Plan
- Slack Pro
- Slack Business+
- Slack Enterprise Grid

## Setup Guide

### Step 1: Connect Slack in JetScale

![Slack Connection Screen](#)
*Screenshot placeholder: JetScale Slack integration page*

1. Navigate to **Settings** → **Integrations** in JetScale
2. Click **Connect Slack**
3. You'll be redirected to Slack's authorization page
4. Select your workspace from the dropdown
5. Click **Allow** to grant permissions

JetScale will request the following permissions:
- **Send messages**: Post notifications to channels
- **View channel information**: List available channels
- **Upload files**: Share detailed reports (optional)

### Step 2: Select Notification Channel

![Channel Selection Screen](#)
*Screenshot placeholder: Slack channel configuration*

1. After authorization, you'll return to JetScale
2. Select your **default notification channel** from the dropdown
3. Click **Save Channel**

You can change the channel at any time or configure multiple channels for different notification types.

### Step 3: Configure Notification Preferences

![Notification Preferences](#)
*Screenshot placeholder: Notification settings*

Customize what gets sent to Slack:

- **New Recommendations** (recommended)
- **High-Value Opportunities** (savings > $500/month)
- **Deployment Status Updates**
- **Weekly Savings Summary**
- **Monthly Cost Reports**

## How It Works

### Notification Types

#### New Recommendation Alerts

When JetScale identifies a cost optimization opportunity:

```
💰 New Cost Optimization Recommendation

Resource: production-postgres-db
Type: RDS Instance Right-Sizing
Current: db.r5.2xlarge ($730/month)
Recommended: db.r5.xlarge ($365/month)

Monthly Savings: $365 (50%)
Annual Savings: $4,380

Performance Impact: Low Risk
✅ 2x headroom above peak usage maintained

[View in JetScale] [View Terraform]
```

#### High-Value Opportunities

For recommendations exceeding your configured savings threshold:

```
🎯 High-Value Optimization Detected!

Resource: customer-aurora-cluster
Type: Aurora I/O-Optimized Migration
Monthly Savings: $1,250 (35%)
Annual Savings: $15,000

This represents 8% of your total cloud costs.

Quick Stats:
• Risk Level: Very Low
• Implementation Time: ~30 minutes
• Downtime Required: None

[Review Recommendation] [Schedule Implementation]
```

#### Deployment Status Updates

When you implement a recommendation:

```
✅ Optimization Deployed Successfully

Resource: api-server-prod
Optimization: EC2 Graviton Migration (m5.xlarge → m6g.xlarge)

Deployment completed at 2:45 PM
Performance metrics: All green ✓
Monitoring active for 48 hours

Expected Monthly Savings: $140
Expected Annual Savings: $1,680

[View Metrics Dashboard]
```

#### Weekly Savings Summary

Every Monday morning (configurable):

```
📊 Weekly Cost Optimization Summary

Total Savings Identified: $2,450/month
Recommendations Implemented: 3
New Recommendations: 7

🏆 Top Opportunities:
1. RDS Instance Right-Sizing: $850/month
2. EC2 Graviton Migration: $420/month
3. EBS Volume Optimization: $380/month

📈 Month-to-Date:
• Implemented Savings: $3,200/month
• Projected Annual Impact: $38,400

[View Full Dashboard] [Review All Recommendations]
```

#### Monthly Cost Reports

First day of each month:

```
📊 Monthly Cloud Cost Report - January 2024

Total Cloud Spend: $28,450
Optimized Savings: $4,200 (14.8% reduction)
Recommendations Implemented: 12

💰 Savings Breakdown:
• RDS Optimization: $1,850/month
• EC2 Right-Sizing: $1,420/month
• EBS Volume Reduction: $930/month

🎯 Remaining Opportunities: $3,800/month

Year-to-Date Savings: $12,600

[View Detailed Report] [Download CSV]
```

## Notification Settings

### Frequency Configuration

Control how often you receive notifications:

| Notification Type | Frequency Options |
|-------------------|-------------------|
| New Recommendations | Real-time, Daily digest, Weekly digest |
| High-Value Opportunities | Real-time (recommended) |
| Deployment Status | Real-time |
| Savings Summary | Weekly, Bi-weekly |
| Cost Reports | Monthly, Quarterly |

### Threshold Configuration

Set minimum savings thresholds to reduce noise:

- **Minimum monthly savings**: $50, $100, $250, $500, $1000
- **Minimum annual savings**: $1000, $2500, $5000, $10000
- **Minimum percentage savings**: 10%, 20%, 30%, 50%

### Channel Routing

Route different notification types to different channels:

**Example Configuration:**
```
#cloud-costs → All recommendations and reports
#engineering-alerts → High-value opportunities only
#devops → Deployment status updates
#leadership → Monthly cost reports only
```

## Slash Commands (Optional)

JetScale provides optional slash commands for interactive queries:

| Command | Description | Example |
|---------|-------------|---------|
| `/jetscale summary` | Current month savings summary | `/jetscale summary` |
| `/jetscale recommendations` | List top 5 recommendations | `/jetscale recommendations` |
| `/jetscale report` | Generate cost report | `/jetscale report last-month` |
| `/jetscale help` | Show available commands | `/jetscale help` |

**Note**: Slash commands require additional permissions during setup. They are entirely optional and can be enabled/disabled independently of notifications.

## Managing the Integration

### View Integration Status

From **Settings** → **Integrations** → **Slack**, you can view:
- Connected workspace name
- Active notification channels
- Recent notifications sent
- Last connection verification time
- Notification preferences

### Test Connection

To verify your Slack integration is working:

1. Navigate to **Settings** → **Integrations** → **Slack**
2. Click **Send Test Message**
3. Check your configured channel for a test notification

### Change Notification Channel

To route notifications to a different channel:

1. Navigate to **Settings** → **Integrations** → **Slack**
2. Click **Change Channel**
3. Select a new channel from the dropdown
4. Click **Update**

**Important**: You must invite `@JetScale` bot to private channels before they appear in the selection dropdown.

### Update Preferences

To modify notification settings:

1. Navigate to **Settings** → **Integrations** → **Slack**
2. Click **Configure Notifications**
3. Toggle notification types on/off
4. Adjust frequency settings
5. Set savings thresholds
6. Click **Save Changes**

### Disconnect Slack

To remove the Slack integration:

1. Navigate to **Settings** → **Integrations** → **Slack**
2. Click **Disconnect**
3. Confirm disconnection

**Important**: Disconnecting will:
- Stop all notifications to Slack
- Remove the JetScale bot from your workspace
- Preserve notification history in JetScale dashboard
- Disable slash commands (if enabled)

You can reconnect at any time by repeating the setup process.

## Troubleshooting

### Connection Failed

**Problem**: "Unable to connect to Slack" error

**Solutions**:
- Verify you have permissions to install apps in your workspace
- Check if your organization requires admin approval for new apps
- Ensure you selected the correct workspace during authorization
- Try disconnecting and reconnecting

### Not Receiving Notifications

**Problem**: Notifications aren't appearing in Slack channel

**Solutions**:
- Verify the JetScale bot is a member of the channel (invite `@JetScale`)
- Check notification preferences are enabled
- Confirm savings thresholds aren't filtering all recommendations
- Send a test message to verify connection
- Check Slack's "Allow apps to send messages" setting

### Bot Not in Channel List

**Problem**: Target channel doesn't appear in dropdown

**Solutions**:
- Public channels appear automatically
- Private channels require you to invite `@JetScale` first
- Type `/invite @JetScale` in the target channel
- Refresh the JetScale integration page after inviting

### Duplicate Notifications

**Problem**: Receiving the same notification multiple times

**Solutions**:
- Check if multiple team members connected separate integrations
- Review notification preferences for duplicate settings
- Verify you didn't configure multiple notification channels for the same type
- Contact support if issue persists

### Slash Commands Not Working

**Problem**: `/jetscale` commands return error

**Solutions**:
- Verify slash commands are enabled in integration settings
- Re-authorize the Slack integration to grant command permissions
- Check command syntax (use `/jetscale help` for reference)
- Ensure bot has necessary permissions in the channel

## Best Practices

### Channel Organization

**Recommended channel structure:**
- **#cloud-costs**: Primary channel for all cost-related notifications
- **#engineering-alerts**: High-priority recommendations for immediate action
- **#devops-deploy**: Deployment status and implementation tracking
- **#leadership-reports**: Monthly summaries and high-level metrics

### Notification Strategy

**For small teams (< 20 people):**
- Single channel for all notifications
- Real-time alerts for high-value opportunities
- Weekly digest for routine recommendations

**For medium teams (20-100 people):**
- Separate channels by notification type
- Real-time for deployments and high-value
- Daily digest for new recommendations

**For large teams (100+ people):**
- Role-based channel routing
- Aggressive threshold filtering
- Weekly digests preferred
- Executive summary channel for leadership

### Integration with Workflows

**Recommendation approval workflow:**
1. JetScale posts recommendation to `#cloud-costs`
2. Team discusses in thread
3. DevOps uses 👍 reaction to indicate approval
4. Engineer implements via JetScale dashboard
5. Status update posted automatically

**Use Slack reactions for tracking:**
- 👀 = Under review
- 👍 = Approved for implementation
- 🚀 = Deployed to staging
- ✅ = Deployed to production
- ❌ = Rejected (document reason in thread)

### Alerting Thresholds

**Conservative approach** (less noise):
- Minimum savings: $500/month
- High-value threshold: $1000/month
- Weekly digests for routine recommendations

**Aggressive approach** (catch everything):
- Minimum savings: $50/month
- High-value threshold: $500/month
- Real-time notifications for all recommendations

**Recommended starting point:**
- Minimum savings: $100/month (monthly recurring)
- High-value threshold: $500/month
- Daily digest for new recommendations
- Real-time for high-value and deployments

## Security Considerations

### Permissions

JetScale requests only the minimum required Slack permissions:
- **Post messages**: Required for all notifications
- **Read channel information**: Required to list available channels
- **Upload files**: Optional, for detailed reports

JetScale cannot:
- Read messages from your channels
- Access private messages or DMs
- Modify channel settings
- Invite or remove users
- Access workspace settings

### Data Privacy

- Slack notifications contain only summary information
- Sensitive credentials never included in messages
- Detailed data available only via authenticated JetScale dashboard links
- All links require active JetScale session

### Audit Trail

Every notification sent is recorded:
- Timestamp and recipient channel
- Notification type and content
- User who configured the integration
- Audit logs available in your JetScale dashboard

## API Reference

For programmatic access to the Slack integration, see our [API Documentation](../api-reference.md#slack-integration).

Key endpoints:
- `POST /api/v2/integrations/slack/me/action/connect` - Initiate OAuth flow
- `GET /api/v2/integrations/slack/channels` - List available channels
- `POST /api/v2/integrations/slack/me/action/configure` - Update preferences
- `POST /api/v2/integrations/slack/notifications/action/send-test` - Send test message
- `POST /api/v2/integrations/slack/me/action/disconnect` - Remove integration

## Support

Need help with Slack integration?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: [FAQ](../faq.md)
- **GitHub Issues**: [Report a problem](https://github.com/Jetscale-ai/jetscale-docs/issues)
