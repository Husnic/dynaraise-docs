# Quick Start Guide

Get started with the Dynaraise Integration API in 5 minutes.

## Prerequisites

- Active Dynaraise organization account
- API key (generated from admin dashboard)
- Development environment with HTTP client (Postman, curl, etc.)

## Step 1: Get Your API Key

1. Log in to your Dynaraise admin dashboard
2. Navigate to **Organization Settings** → **API Keys**
3. Click **"Create API Key"**
4. Give it a name (e.g., "Production API")
5. Copy and save your API key securely (shown only once!)

Your API key will look like:

```
dr_live_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

## Step 2: Import Postman Collection

1. Download the Postman collection: `dynaraise-integration-api.postman_collection.json`
2. Open Postman
3. Click **Import** → **Upload Files**
4. Select the downloaded JSON file
5. Collection will appear in your workspace

## Step 3: Configure Variables

In Postman, set these collection variables:

| Variable          | Value        | Example                     |
| ----------------- | ------------ | --------------------------- |
| `BASE_URL`        | API base URL | `https://api.dynaraise.com` |
| `API_KEY`         | Your API key | `dr_live_a1b2c3...`         |
| `ORGANIZATION_ID` | Your org ID  | `01JBORG123...`             |

## Step 4: Test Your Connection

### Get Organization Details

```bash
curl -X GET https://api.dynaraise.com/organization-admin \
  -H "x-api-key: dr_live_your_api_key_here" \
  -H "x-organization-id: your_organization_id"
```

**Expected Response:**

```json
{
  "id": "your_organization_id",
  "name": "Your Organization",
  "email": "info@yourorg.com",
  "isVerified": "APPROVED"
}
```

## Step 5: Create Your First Campaign

### Get Available Categories

```bash
curl -X GET https://api.dynaraise.com/campaign-categories \
  -H "x-api-key: dr_live_your_api_key_here" \
  -H "x-organization-id: your_organization_id"
```

### Create Campaign

```bash
curl -X POST https://api.dynaraise.com/campaigns-admin/create \
  -H "Content-Type: application/json" \
  -H "x-api-key: dr_live_your_api_key_here" \
  -H "x-organization-id: your_organization_id" \
  -d '{
    "title": "Help Build a School",
    "summary": "Raising funds to build a school for 200 children",
    "story": "<p>Our community desperately needs a school...</p>",
    "target": 5000000,
    "currency": "NGN",
    "country": "NG",
    "type": "CHARITY",
    "payout": "FLEXIBLE",
    "categoryId": "category_id_from_step_above",
    "beneficiary": {
      "firstName": "John",
      "lastName": "Doe",
      "email": "john.doe@example.com",
      "phoneNumber": "+2348012345678",
      "accountNumber": "0123456789",
      "accountName": "John Doe",
      "bankName": "Access Bank",
      "bankCode": "044",
      "bvn": "12345678901"
    }
  }'
```

**Expected Response:**

```json
{
  "id": "01JBCAMP123456789ABCDEFGH",
  "title": "Help Build a School",
  "status": "DRAFT",
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

## Step 6: Set Up Webhooks

### Configure Webhook URL

```bash
curl -X PATCH https://api.dynaraise.com/organization-admin/your_org_id \
  -H "Content-Type: application/json" \
  -H "x-api-key: dr_live_your_api_key_here" \
  -H "x-organization-id: your_organization_id" \
  -d '{
    "webhookUrl": "https://your-domain.com/webhooks/dynaraise"
  }'
```

### Implement Webhook Endpoint

**Node.js/Express Example:**

```javascript
const express = require("express");
const app = express();

app.use(express.json());

app.post("/webhooks/dynaraise", (req, res) => {
  // Acknowledge receipt immediately
  res.status(200).send("OK");

  // Process event
  const { event, data } = req.body;
  console.log(`Received ${event}:`, data);

  // Handle different events
  switch (event) {
    case "campaign.created":
      console.log("New campaign:", data.campaignId);
      break;
    case "donation.received":
      console.log("New donation:", data.amount);
      break;
    case "user.kyc.verified":
      console.log("KYC verified:", data.userId);
      break;
  }
});

app.listen(3000);
```

## Common Workflows

### Complete Campaign Creation Flow

1. **Create Campaign** → Status: `DRAFT`
2. **Upload Banner** → Add campaign image
3. **Upload Documents** → Add supporting files
4. **Request Approval** → Status: `REQUESTED`
5. **Wait for Approval** → Webhook: `campaign.approved`
6. **Campaign Goes Live** → Status: `APPROVED`

### Donation Tracking Flow

1. **User Donates** → Webhook: `donation.received`
2. **Update Campaign Progress** → Fetch donation metrics
3. **Send Thank You** → Email donor
4. **Track Analytics** → Update dashboard

### Payout Request Flow

1. **Check KYC Status** → Must be `APPROVED`
2. **Check Campaign Balance** → Must have funds
3. **Create Payout Request** → Status: `PENDING`
4. **Wait for Approval** → Admin reviews
5. **Payout Processed** → Status: `COMPLETED`

## Testing Tips

### Use Test Mode

For development, use test API keys:

```
dr_test_a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

### Test Webhooks Locally

Use ngrok to expose your local server:

```bash
# Install ngrok
npm install -g ngrok

# Start your local server
node server.js

# Expose it
ngrok http 3000
```

Use the ngrok URL as your webhook URL:

```
https://abc123.ngrok.io/webhooks/dynaraise
```

### Mock Webhook Events

Test your webhook handler with curl:

```bash
curl -X POST http://localhost:3000/webhooks/dynaraise \
  -H "Content-Type: application/json" \
  -H "X-Organization-Id: your_org_id" \
  -d '{
    "event": "donation.received",
    "timestamp": "2024-01-01T00:00:00.000Z",
    "organizationId": "your_org_id",
    "data": {
      "donationId": "test_donation",
      "amount": 50000,
      "currency": "NGN"
    }
  }'
```

## Error Handling

### Common Errors

**401 Unauthorized**

```json
{
  "statusCode": 401,
  "message": "Invalid API key"
}
```

→ Check your API key is correct and active

**422 Validation Error**

```json
{
  "statusCode": 422,
  "message": "Validation failed",
  "errors": [
    {
      "field": "beneficiary.bvn",
      "message": "BVN must be exactly 11 digits"
    }
  ]
}
```

→ Fix the validation errors in your request

**429 Rate Limit**

```json
{
  "statusCode": 429,
  "message": "Too many requests",
  "retryAfter": 60
}
```

→ Wait and retry after the specified seconds

## Next Steps

1. **Read Full Documentation**

   - [API Reference](./README.md)
   - [Webhook Events](./WEBHOOKS.md)
   - [Entity Schemas](./ENTITIES.md)

2. **Explore Postman Collection**

   - Try all available endpoints
   - Customize requests for your use case

3. **Implement Error Handling**

   - Add retry logic
   - Log errors for debugging
   - Monitor API health

4. **Set Up Monitoring**

   - Track API usage
   - Monitor webhook deliveries
   - Set up alerts for failures

5. **Go to Production**
   - Switch to live API keys
   - Update webhook URL to production
   - Test thoroughly before launch

## Support

Need help?

- **Email**: api-support@dynaraise.com
- **Documentation**: https://docs.dynaraise.com
- **Status**: https://status.dynaraise.com

## Checklist

Before going live, ensure:

- [ ] API key is secure and not exposed
- [ ] Webhook endpoint is HTTPS
- [ ] Webhook handler responds within 10 seconds
- [ ] Error handling is implemented
- [ ] Logging is set up
- [ ] Rate limiting is handled
- [ ] Test mode works correctly
- [ ] Production credentials are configured
- [ ] Monitoring is in place
- [ ] Team is trained on API usage

Happy coding! 🚀
