# API Reference

> Public REST API for JetScale integrations

## Overview

The JetScale API allows you to programmatically access recommendations, cost data, and platform features. Use this API to build custom integrations, automate workflows, or create internal dashboards.

### Base URL

```
Production: https://api.jetscale.ai/v1
```

### Coming Soon
The JetScale Public API is currently in beta. Full documentation and access will be available in Q2 2025.

Early access is available for Enterprise customers. Contact [sales@jetscale.ai](mailto:sales@jetscale.ai) for details.

---

## Authentication

The JetScale API uses API keys for authentication. You can generate API keys from your account settings.

**Include your API key in the Authorization header:**

```bash
curl https://api.jetscale.ai/v1/recommendations \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Security Best Practices:**
- Never expose API keys in client-side code
- Rotate keys regularly
- Use different keys for different environments
- Revoke unused keys immediately

---

## Rate Limits

| Plan | Requests/Minute | Requests/Hour |
|------|-----------------|---------------|
| Free Trial | 30 | 500 |
| Pro | 100 | 5,000 |
| Enterprise | Custom | Custom |

Rate limit information is included in response headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1706529600
```

---

## Endpoints (Coming Soon)

The following endpoints will be available in the public API:

### Recommendations

```
GET    /v1/recommendations          List all recommendations
GET    /v1/recommendations/:id      Get recommendation details
POST   /v1/recommendations/:id/approve   Approve recommendation
POST   /v1/recommendations/:id/reject    Reject recommendation
```

### Cloud Accounts

```
GET    /v1/accounts                 List connected cloud accounts
GET    /v1/accounts/:id             Get account details
POST   /v1/accounts                 Connect new cloud account
```

### Cost Data

```
GET    /v1/costs/summary            Get cost summary
GET    /v1/costs/trends             Get cost trends
GET    /v1/savings                  Get realized savings
```

### Resources

```
GET    /v1/resources                List discovered resources
GET    /v1/resources/:id            Get resource details
```

---

## Webhooks (Coming Soon)

Register webhooks to receive real-time notifications:

**Events:**
- `recommendation.created` - New recommendation available
- `recommendation.approved` - Recommendation approved
- `recommendation.deployed` - Changes deployed
- `savings.milestone` - Savings milestone reached
- `account.connected` - New cloud account connected

**Webhook Payload Example:**

```json
{
  "event": "recommendation.created",
  "timestamp": "2025-01-29T10:30:00Z",
  "data": {
    "recommendation_id": "rec_abc123",
    "resource_type": "EC2",
    "estimated_savings": 248.16,
    "risk_level": "low"
  }
}
```

---

## SDK Libraries (Roadmap)

Official SDK libraries are planned for:
- Python
- JavaScript/TypeScript
- Go

**Example Usage (Python - Coming Soon):**

```python
from jetscale import JetScaleClient

client = JetScaleClient(api_key="your_api_key")

# Get recommendations
recommendations = client.recommendations.list(
    status="pending",
    min_savings=100
)

# Approve recommendation
client.recommendations.approve("rec_abc123")

# Get cost summary
summary = client.costs.summary(period="last_30_days")
print(f"Total savings: ${summary['total_savings']}")
```

---

## Use Cases

### Custom Dashboards

Build internal dashboards using the JetScale API:
- Pull recommendation data
- Display cost trends
- Show savings metrics
- Embed JetScale insights into existing tools

### Automation Workflows

Automate cost optimization:
- Auto-approve low-risk recommendations
- Trigger deployments based on custom rules
- Send Slack notifications for new recommendations
- Generate weekly cost reports

### Integration with CI/CD

Integrate cost optimization into your deployment pipeline:
- Check for recommendations before deploying
- Block deployments if cost thresholds exceeded
- Automatically implement approved optimizations
- Track cost impact of deployments

---

## Getting Started

**Beta Access:**

The JetScale API is currently in private beta. To request access:

1. **Contact Sales:** [sales@jetscale.ai](mailto:sales@jetscale.ai)
2. **Provide Use Case:** Describe your integration goals
3. **Receive API Key:** Get beta access credentials
4. **Join Beta Program:** Access early documentation and support

**Enterprise Customers:**

All Enterprise plan customers have full API access. Contact your customer success manager for:
- API key generation
- Custom rate limits
- Integration support
- Webhook configuration

---

## Support

**Questions about the API?**
- Email: [api-support@jetscale.ai](mailto:api-support@jetscale.ai)
- Documentation Issues: [Report Here](https://github.com/jetscale-ai/docs/issues)
- Feature Requests: [Request Feature](https://jetscale.ai/feature-requests)

**Related Documentation:**
- [How JetScale Works](how-it-works.md)
- [GitHub Integration](integrations/github.md)

---

*Last Updated: January 29, 2025*
*API Version: 1.0 (Beta)*
