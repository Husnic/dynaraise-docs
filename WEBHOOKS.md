# Webhook Events Documentation

Dynaraise uses webhooks to notify your application about events that happen in real-time. This allows you to build integrations that react to campaign activities, donations, and KYC verifications.

## Table of Contents

- [Overview](#overview)
- [Setup](#setup)
- [Security](#security)
- [Event Types](#event-types)
- [Payload Structure](#payload-structure)
- [Retry Logic](#retry-logic)
- [Best Practices](#best-practices)

## Overview

Webhooks are HTTP POST requests sent to your configured endpoint when specific events occur. Your server should respond with a `2xx` status code to acknowledge receipt.

### Key Features

- **Real-time notifications** - Receive events as they happen
- **Automatic retries** - Failed deliveries are retried up to 3 times
- **Event filtering** - Only receive events relevant to your organization
- **Secure delivery** - Includes organization ID header for verification

## Setup

### 1. Configure Webhook URL

Set your webhook URL in the organization settings:

```http
PATCH /organization-admin/:organizationId
Content-Type: application/json

{
  "webhookUrl": "https://your-domain.com/webhooks/dynaraise"
}
```

### 2. Implement Webhook Endpoint

Your endpoint should:

- Accept POST requests
- Return `200-299` status code within 10 seconds
- Process events asynchronously (don't block the response)

**Example (Node.js/Express):**

```javascript
app.post('/webhooks/dynaraise', async (req, res) => {
  // Acknowledge receipt immediately
  res.status(200).send('OK');

  // Process event asynchronously
  const { event, data, timestamp, organizationId } = req.body;

  try {
    await processWebhookEvent(event, data);
  } catch (error) {
    console.error('Webhook processing error:', error);
  }
});
```

### 3. Verify Webhook Source

Always verify the webhook is from Dynaraise:

```javascript
const organizationId = req.headers['x-organization-id'];

if (organizationId !== process.env.YOUR_ORGANIZATION_ID) {
  return res.status(403).send('Forbidden');
}
```

## Security

### Headers

Each webhook request includes:

```http
Content-Type: application/json
X-Webhook-Event: event.name
X-Organization-Id: your_organization_id
```

### Verification Steps

1. **Check Organization ID** - Verify the `X-Organization-Id` header matches your organization
2. **Validate Timestamp** - Reject events older than 5 minutes to prevent replay attacks
3. **Use HTTPS** - Always use HTTPS for your webhook endpoint
4. **IP Whitelisting** (optional) - Restrict requests to Dynaraise IP addresses

**Example Verification:**

```javascript
function verifyWebhook(req) {
  const orgId = req.headers['x-organization-id'];
  const timestamp = new Date(req.body.timestamp);
  const now = new Date();

  // Check organization ID
  if (orgId !== process.env.YOUR_ORGANIZATION_ID) {
    throw new Error('Invalid organization ID');
  }

  // Check timestamp (reject if older than 5 minutes)
  const ageInMinutes = (now - timestamp) / 1000 / 60;
  if (ageInMinutes > 5) {
    throw new Error('Webhook too old');
  }

  return true;
}
```

## Event Types

### User Events

#### `user.created`

Triggered when a new beneficiary user is created during campaign creation.

**Payload:**

```json
{
  "event": "user.created",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "organizationId": "org_id",
  "data": {
    "userId": "user_id",
    "email": "beneficiary@example.com",
    "organizationId": "individual_org_id"
  }
}
```

#### `user.kyc.pending`

Triggered when KYC validation is submitted to the payment provider.

**Payload:**

```json
{
  "event": "user.kyc.pending",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "organizationId": "org_id",
  "data": {
    "userId": "user_id",
    "email": "beneficiary@example.com",
    "organizationId": "individual_org_id"
  }
}
```

**What to do:**

- Update your records to show KYC is in progress
- Notify the beneficiary that verification is pending
- Expect `user.kyc.verified` or `user.kyc.failed` within 24-48 hours

#### `user.kyc.verified`

Triggered when KYC verification is successful (via payment provider webhook).

**Payload:**

```json
{
  "event": "user.kyc.verified",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "organizationId": "org_id",
  "data": {
    "userId": "user_id",
    "email": "beneficiary@example.com",
    "organizationId": "individual_org_id"
  }
}
```

**What to do:**

- Update beneficiary status to verified
- Enable payout capabilities for the campaign
- Notify beneficiary of successful verification

**Note:** BVN is automatically nullified after successful verification for security.

#### `user.kyc.failed`

Triggered when KYC verification fails.

**Payload:**

```json
{
  "event": "user.kyc.failed",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "organizationId": "org_id",
  "data": {
    "userId": "user_id",
    "email": "beneficiary@example.com",
    "organizationId": "individual_org_id",
    "reason": "BVN validation failed"
  }
}
```

**What to do:**

- Update beneficiary status to failed
- Notify beneficiary to correct their information
- Campaign cannot receive payouts until KYC is verified

### Campaign Events

#### `campaign.created`

Triggered when a new campaign is created.

**Payload:**

```json
{
  "event": "campaign.created",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "organizationId": "org_id",
  "data": {
    "campaignId": "campaign_id",
    "title": "Campaign Title",
    "target": 5000000,
    "currency": "NGN",
    "userId": "user_id"
  }
}
```

#### `campaign.approved`

Triggered when a campaign is approved by admin.

**Payload:**

```json
{
  "event": "campaign.approved",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "organizationId": "org_id",
  "data": {
    "campaignId": "campaign_id",
    "title": "Campaign Title",
    "target": 5000000,
    "summary": "Campaign summary",
    "approvedAt": "2024-01-01T12:00:00.000Z",
    "user": {
      "id": "user_id",
      "email": "beneficiary@example.com"
    }
  }
}
```

**What to do:**

- Update campaign status to active
- Notify beneficiary of approval
- Campaign can now receive donations

### Donation Events

#### `donation.received`

Triggered when a new donation is made to any campaign.

**Payload:**

```json
{
  "event": "donation.received",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "organizationId": "org_id",
  "data": {
    "donationId": "donation_id",
    "campaignId": "campaign_id",
    "amount": 50000,
    "processingFees": 1500,
    "charges": 500,
    "currency": "NGN",
    "donorEmail": "donor@example.com",
    "donorName": "John Doe",
    "donorLocation": "Lagos, Nigeria"
  }
}
```

**What to do:**

- Update campaign progress
- Send thank you message to donor
- Update analytics and metrics

## Payload Structure

All webhook payloads follow this structure:

```typescript
interface WebhookPayload {
  event: string; // Event type (e.g., "user.created")
  timestamp: string; // ISO 8601 timestamp
  organizationId: string; // Your organization ID
  data: Record<string, any>; // Event-specific data
}
```

## Retry Logic

### Automatic Retries

If your endpoint doesn't respond with a `2xx` status code, Dynaraise will retry:

- **Attempt 1**: Immediate
- **Attempt 2**: After 30 seconds
- **Attempt 3**: After 2 minutes

### Timeout

- Each request has a **10-second timeout**
- Respond quickly and process asynchronously

### Failed Deliveries

After 3 failed attempts:

- Event is marked as failed
- No further retries are attempted
- You can view failed webhooks in your dashboard

## Best Practices

### 1. Respond Quickly

```javascript
// ✅ Good - Respond immediately
app.post('/webhooks', async (req, res) => {
  res.status(200).send('OK');
  await processEventAsync(req.body);
});

// ❌ Bad - Blocking response
app.post('/webhooks', async (req, res) => {
  await processEvent(req.body); // Don't wait!
  res.status(200).send('OK');
});
```

### 2. Implement Idempotency

Events may be delivered more than once. Use the event ID or timestamp to prevent duplicate processing:

```javascript
const processedEvents = new Set();

async function processWebhook(payload) {
  const eventId = `${payload.event}-${payload.timestamp}-${payload.data.userId}`;

  if (processedEvents.has(eventId)) {
    console.log('Event already processed');
    return;
  }

  // Process event
  await handleEvent(payload);

  processedEvents.add(eventId);
}
```

### 3. Handle Errors Gracefully

```javascript
app.post('/webhooks', async (req, res) => {
  try {
    // Acknowledge receipt first
    res.status(200).send('OK');

    // Process with error handling
    await processWebhook(req.body);
  } catch (error) {
    // Log error but don't fail the webhook
    console.error('Webhook processing error:', error);
    // Store for manual review
    await saveFailedWebhook(req.body, error);
  }
});
```

### 4. Log All Webhooks

Keep a log of all received webhooks for debugging:

```javascript
app.post('/webhooks', async (req, res) => {
  // Log incoming webhook
  await logWebhook({
    event: req.body.event,
    timestamp: req.body.timestamp,
    payload: req.body,
    headers: req.headers,
  });

  res.status(200).send('OK');
  await processWebhook(req.body);
});
```

### 5. Monitor Webhook Health

Track webhook metrics:

- Success rate
- Processing time
- Failed events
- Retry attempts

### 6. Test Webhooks

Use tools like:

- **ngrok** - Expose local server for testing
- **Postman** - Manually trigger webhook events
- **Webhook.site** - Inspect webhook payloads

## Example Implementation

### Complete Express.js Example

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// Webhook endpoint
app.post('/webhooks/dynaraise', async (req, res) => {
  try {
    // Verify webhook
    verifyWebhook(req);

    // Acknowledge receipt immediately
    res.status(200).send('OK');

    // Process asynchronously
    const { event, data, timestamp } = req.body;

    // Check for duplicates
    if (await isDuplicate(event, timestamp, data)) {
      console.log('Duplicate webhook ignored');
      return;
    }

    // Route to appropriate handler
    switch (event) {
      case 'user.created':
        await handleUserCreated(data);
        break;
      case 'user.kyc.pending':
        await handleKycPending(data);
        break;
      case 'user.kyc.verified':
        await handleKycVerified(data);
        break;
      case 'user.kyc.failed':
        await handleKycFailed(data);
        break;
      case 'campaign.created':
        await handleCampaignCreated(data);
        break;
      case 'campaign.approved':
        await handleCampaignApproved(data);
        break;
      case 'donation.received':
        await handleDonationReceived(data);
        break;
      default:
        console.log('Unknown event:', event);
    }

    // Log success
    await logWebhookSuccess(event, timestamp);
  } catch (error) {
    console.error('Webhook error:', error);
    await logWebhookError(req.body, error);
  }
});

// Event handlers
async function handleUserCreated(data) {
  console.log('New user created:', data.userId);
  // Update your database
  // Send notifications
}

async function handleKycVerified(data) {
  console.log('KYC verified for user:', data.userId);
  // Enable payout features
  // Notify beneficiary
}

async function handleDonationReceived(data) {
  console.log('New donation:', data.donationId);
  // Update campaign progress
  // Send thank you email
  // Update analytics
}

app.listen(3000, () => {
  console.log('Webhook server running on port 3000');
});
```

## Troubleshooting

### Webhook Not Received

1. **Check webhook URL** - Ensure it's publicly accessible
2. **Verify HTTPS** - Use HTTPS, not HTTP
3. **Check firewall** - Ensure port is open
4. **Review logs** - Check your server logs for errors

### Webhook Failing

1. **Response time** - Ensure you respond within 10 seconds
2. **Status code** - Return `200-299` status code
3. **Error handling** - Don't throw errors that prevent response

### Testing Webhooks Locally

Use ngrok to expose your local server:

```bash
ngrok http 3000
```

Then set your webhook URL to the ngrok URL:

```
https://abc123.ngrok.io/webhooks/dynaraise
```

## Support

For webhook-related issues:

- **Email**: support@dynaraise.com
- **Documentation**: https://docs.dynaraise.com/webhooks
- **Status**: Check webhook delivery status in your dashboard
