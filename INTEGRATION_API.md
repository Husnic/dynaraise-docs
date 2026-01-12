# Integration API Endpoints

This document provides detailed information about the Integration API endpoints for campaigns and donations. These endpoints are designed for third-party organizations to integrate with Dynaraise and send donations on behalf of their users.

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Authorization](#authorization)
- [Campaign Integration Endpoints](#campaign-integration-endpoints)
  - [Get Campaigns](#get-campaigns)
  - [Get Campaign by ID](#get-campaign-by-id)
- [Donation Integration Endpoints](#donation-integration-endpoints)
  - [Send Donation Webhook](#send-donation-webhook)
  - [Get Donations](#get-donations)
- [Error Handling](#error-handling)
- [Best Practices](#best-practices)

## Overview

The Integration API allows authorized organizations to:

1. **Browse and search campaigns** to display on their platform
2. **Send donation webhooks** when users donate through their platform
3. **Retrieve donation history** for reporting and reconciliation

All integration endpoints require API key authentication and are scoped to your organization.

## Authentication

All requests must include your API key in the request headers:

```http
x-api-key: dr_live_your_api_key_here
```

The API key automatically identifies your organization, so no additional organization ID is required.

## Authorization

To use the Integration API, your organization must be authorized by a Dynaraise platform administrator:

1. Contact Dynaraise support to request integration access
2. An admin will enable the `canSendIntegrationDonations` flag for your organization
3. Once enabled, you can start using the integration endpoints

**Note**: Attempting to use integration donation endpoints without authorization will result in a `403 Forbidden` error.

## Campaign Integration Endpoints

### Get Campaigns

Retrieve a list of published campaigns with filtering and pagination.

**Endpoint:**

```http
GET /campaigns-integration
```

**Query Parameters:**

| Parameter         | Type    | Required | Description                                                 |
| ----------------- | ------- | -------- | ----------------------------------------------------------- |
| `search`          | string  | No       | Search across campaign title, summary, and URL slug         |
| `isFeatured`      | boolean | No       | Filter by featured campaigns                                |
| `isRamadanGiving` | boolean | No       | Filter Ramadan giving campaigns (urlSlug: 'ramadan-giving') |
| `userId`          | string  | No       | Filter by campaign creator user ID                          |
| `organizationId`  | string  | No       | Filter by organization ID                                   |
| `page`            | number  | No       | Page number (default: 1)                                    |
| `limit`           | number  | No       | Items per page (default: 25, max: 100)                      |

**Example Request:**

```bash
curl -X GET "https://service-test.dynaraise.com/campaigns-integration?search=education&isFeatured=true&page=1&limit=10" \
  -H "x-api-key: dr_live_your_api_key_here"
```

**Response:**

```json
{
  "data": [
    // Array of Campaign objects - see Campaign entity schema below
  ],
  "count": 10,
  "total": 45,
  "page": 1,
  "pageCount": 5
}
```

**Campaign Entity Schema:**

For detailed Campaign entity structure including all fields and relations, see the [Campaign Entity Schema in ENTITIES.md](./ENTITIES.md#campaign).

**Key Fields Returned:**

- Basic info: `id`, `title`, `summary`, `urlSlug`, `type`, `model`
- Financial: `target`, `currency`, `realized`, `totalRealized`, `donationCount`
- Status: `published`, `isComplete`, `isDeactivated`, `isVerified`
- Dates: `endDate`, `createdAt`, `updatedAt`
- Relations: `user`, `organization`, `parentOrganization`, `category`, `bannerImage`

### Get Campaign by ID

Retrieve a single campaign by its ID or URL slug.

**Endpoint:**

```http
GET /campaigns-integration/:campaignId
```

**Path Parameters:**

| Parameter    | Type   | Required | Description             |
| ------------ | ------ | -------- | ----------------------- |
| `campaignId` | string | Yes      | Campaign ID or URL slug |

**Example Request:**

```bash
curl -X GET "https://service-test.dynaraise.com/campaigns-integration/build-schools-nigeria" \
  -H "x-api-key: dr_live_your_api_key_here"
```

**Response:**

```json
{
  // Single Campaign object - see Campaign entity schema in ENTITIES.md
  "id": "01HQXYZ123456789",
  "title": "Build Schools in Rural Nigeria",
  "urlSlug": "build-schools-nigeria"
  // ... other campaign fields
}
```

See [Campaign Entity Schema in ENTITIES.md](./ENTITIES.md#campaign) for complete field details.

## Donation Integration Endpoints

### Webhook Events

The webhook endpoint receives different event types from integration partners. All events follow a common structure with an `event` field indicating the type and a `data` field containing event-specific information.

**Endpoint:**

```http
POST /donations-integration/webhook
```

**Common Request Structure:**

```json
{
  "event": "event.type",
  "timestamp": "2026-01-08T05:42:31Z",
  "data": {
    // Event-specific data
  }
}
```

**Supported Event Types:**

- `donation.completed` - Notification of a successful donation
- `settlement.completed` - Notification of a settlement batch

---

### Event: donation.completed

Sent when a user completes a donation through your platform.

**Request Body:**

```json
{
  "event": "donation.completed",
  "timestamp": "2026-01-08T05:42:31Z",
  "data": {
    "campaignId": "build-schools-nigeria",
    "email": "donor@example.com",
    "amount": 50000,
    "currency": "NGN",
    "displayName": "Jane Smith",
    "displayLocation": "Lagos, Nigeria",
    "transactionReference": "TXN_123456789"
  }
}
```

**Data Fields:**

| Field                  | Type   | Required | Description                                                                      |
| ---------------------- | ------ | -------- | -------------------------------------------------------------------------------- |
| `campaignId`           | string | No       | Campaign URL slug or ID. If omitted, donation goes to a random featured campaign |
| `email`                | string | Yes      | Donor email address                                                              |
| `amount`               | number | Yes      | Donation amount in minor currency units (e.g., kobo for NGN)                     |
| `currency`             | string | No       | Currency code (default: "NGN")                                                   |
| `displayName`          | string | No       | Donor display name (use for non-anonymous donations)                             |
| `displayLocation`      | string | No       | Donor location (e.g., "Lagos, Nigeria")                                          |
| `transactionReference` | string | Yes      | Unique transaction reference for idempotency                                     |

**Example Request:**

```bash
curl -X POST "https://service-test.dynaraise.com/donations-integration/webhook" \
  -H "x-api-key: dr_live_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "event": "donation.completed",
    "timestamp": "2026-01-08T05:42:31Z",
    "data": {
      "campaignId": "build-schools-nigeria",
      "email": "donor@example.com",
      "amount": 50000,
      "currency": "NGN",
      "displayName": "Jane Smith",
      "displayLocation": "Lagos, Nigeria",
      "transactionReference": "TXN_123456789"
    }
  '
```

**Response:**

```json
{
  "id": "donation_id",
  "email": "donor@example.com",
  "amount": 50000,
  "currency": "NGN",
  "displayName": "Jane Smith",
  "processingFees": 2500,
  "transactionReference": "TXN_123456789",
  "paymentStatus": "SUCCESSFUL",
  "isSuccessfullyDonated": true,
  "createdAt": "2024-01-01T12:00:00.000Z"
}
```

**Processing Details:**

- **Idempotency**: If a donation with the same `transactionReference` exists:
  - **Not successfully donated**: Retries the campaign metrics update and marks as successful
  - **Already successful**: Returns `400 Bad Request` error
- **Random Campaign Selection**: If `campaignId` is not provided, the system selects a random **featured** campaign that is:
  - Published and approved
  - Not complete or deactivated
  - Not expired (endDate > now or null)

---

### Event: settlement.completed

Sent when a batch of donations has been settled. This provides a summary of all donations processed in a settlement period.

**Request Body:**

```json
{
  "event": "settlement.completed",
  "timestamp": "2026-01-09T02:15:00Z",
  "data": {
    "settlementId": "SETTLE_20260108",
    "settlementDate": "2026-01-08",
    "currency": "NGN",
    "summary": {
      "grossAmount": 1200000,
      "fees": 0,
      "netAmount": 1200000
    },
    "campaignBreakdown": [
      {
        "campaignId": "01HQXYZ123456789",
        "grossAmount": 600000
      },
      {
        "campaignId": "01RCXN3334456789",
        "grossAmount": 600000
      }
    ],
    "donations": [
      {
        "transactionReference": "TXN_123",
        "amount": 50000
      },
      {
        "transactionReference": "TXN_124",
        "amount": 100000
      }
    ]
  }
}
```

**Data Fields:**

| Field               | Type   | Required | Description                                       |
| ------------------- | ------ | -------- | ------------------------------------------------- |
| `settlementId`      | string | Yes      | Unique settlement identifier                      |
| `settlementDate`    | string | Yes      | Settlement date (ISO 8601 date format)            |
| `currency`          | string | Yes      | Currency code                                     |
| `summary`           | object | Yes      | Settlement summary with gross, fees, and net      |
| `campaignBreakdown` | array  | Yes      | Array of campaign IDs and their gross amounts     |
| `donations`         | array  | Yes      | Array of transaction references and their amounts |

**Example Request:**

```bash
curl -X POST "https://service-test.dynaraise.com/donations-integration/webhook" \
  -H "x-api-key: dr_live_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "event": "settlement.completed",
    "timestamp": "2026-01-09T02:15:00Z",
    "data": {
      "settlementId": "SETTLE_20260108",
      "settlementDate": "2026-01-08",
      "currency": "NGN",
      "summary": {
        "grossAmount": 1200000,
        "fees": 0,
        "netAmount": 1200000
      },
      "campaignBreakdown": [
        {
          "campaignId": "01HQXYZ123456789",
          "grossAmount": 600000
        }
      ],
      "donations": [
        {
          "transactionReference": "TXN_123",
          "amount": 50000
        }
      ]
    }
  }'
```

**Response:**

```json
{
  "id": "settlement_entity_id",
  "settlementId": "SETTLE_20260108",
  "settlementDate": "2026-01-08",
  "currency": "NGN",
  "grossAmount": 1200000,
  "fees": 0,
  "netAmount": 1200000,
  "status": "RECEIVED",
  "createdAt": "2026-01-09T02:15:00.000Z"
}
```

**Settlement Status:**

- `RECEIVED` - Settlement notification received and recorded
- `CONFIRMED` - Settlement has been confirmed (set when duplicate settlement ID is received)

**Validation:**

- All campaigns specified in `campaignBreakdown` must exist in the database
- All donations specified in `donations` array must exist with matching transaction references
- If any campaigns or donations are not found, a `404 Not Found` error is returned with details of missing items

**Idempotency:**

- If a settlement with the same `settlementId` already exists, a `400 Bad Request` error is returned
- Settlements remain in `RECEIVED` status until confirmed by platform administrators

---

## Webhook Events (Outbound)

In addition to receiving webhook events from your platform, Dynaraise will also send webhook events to your configured webhook URL when certain events occur. These outbound webhooks allow you to receive real-time notifications about donations and settlements.

### Configuring Your Webhook URL

Contact Dynaraise support to configure your organization's webhook URL. All outbound webhooks will be sent to this URL with retry logic (3 attempts with exponential backoff).

### Event: donation.received

Sent when a donation is successfully processed for a campaign belonging to your organization (where `parentOrganization` is not "platform").

**Webhook Payload:**

```json
{
  "event": "donation.received",
  "timestamp": "2026-01-12T10:30:00.000Z",
  "organizationId": "01HQXYZ123456789",
  "data": {
    "donationId": "01JXABC123456789",
    "campaignId": "01RCXN3334456789",
    "amount": 50000,
    "processingFees": 2500,
    "charges": 2500,
    "currency": "NGN",
    "donorEmail": "donor@example.com",
    "donorName": "Jane Smith",
    "donorLocation": "Lagos, Nigeria"
  }
}
```

**Headers:**

```http
Content-Type: application/json
X-Webhook-Event: donation.received
X-Organization-Id: 01HQXYZ123456789
```

### Event: settlement.confirmed

Sent when a settlement is confirmed by a platform administrator.

**Webhook Payload:**

```json
{
  "event": "settlement.confirmed",
  "timestamp": "2026-01-12T10:30:00.000Z",
  "organizationId": "01HQXYZ123456789",
  "data": {
    "settlementId": "SETTLE_20260108",
    "settlementDate": "2026-01-08",
    "currency": "NGN",
    "grossAmount": 1200000,
    "fees": 0,
    "netAmount": 1200000,
    "status": "CONFIRMED",
    "confirmedAt": "2026-01-12T10:30:00.000Z"
  }
}
```

**Headers:**

```http
Content-Type: application/json
X-Webhook-Event: settlement.confirmed
X-Organization-Id: 01HQXYZ123456789
```

**Webhook Response:**

Your webhook endpoint should return a `2xx` status code to acknowledge receipt. If a non-2xx status is returned or the request times out (10 seconds), Dynaraise will retry up to 3 times with exponential backoff.

---

### Get Donations

Retrieve donations sent through your integration, with filtering by email and campaign.

**Endpoint:**

```http
GET /donations-integration
```

**Query Parameters:**

| Parameter  | Type   | Required | Description                            |
| ---------- | ------ | -------- | -------------------------------------- |
| `email`    | string | No       | Filter by donor email (partial match)  |
| `campaign` | string | No       | Filter by campaign URL slug or ID      |
| `page`     | number | No       | Page number (default: 1)               |
| `limit`    | number | No       | Items per page (default: 25, max: 100) |

**Example Request:**

```bash
curl -X GET "https://service-test.dynaraise.com/donations-integration?email=donor@example.com&page=1&limit=10" \
  -H "x-api-key: dr_live_your_api_key_here"
```

**Response:**

```json
{
  "data": [
    {
      "id": "donation_id",
      "email": "donor@example.com",
      "displayName": "Jane Smith",
      "displayLocation": "Lagos, Nigeria",
      "phoneNumber": "+2348012345678",
      "amount": 50000,
      "currency": "NGN",
      "transactionReference": "TXN_123456789",
      "paymentStatus": "SUCCESSFUL",
      "createdAt": "2024-01-01T12:00:00.000Z",
      "campaigns": [
        {
          "id": "campaign_id",
          "title": "Build Schools in Rural Nigeria",
          "urlSlug": "build-schools-nigeria",
          "organization": {
            "id": "org_id",
            "name": "Education Foundation"
          }
        }
      ],
      "user": null
    }
  ],
  "count": 10,
  "total": 150,
  "page": 1,
  "pageCount": 15
}
```

**Note**: Only donations sent by your organization (via `sourceOrganization` relation) are returned.

## Error Handling

### Common Error Responses

**401 Unauthorized - Invalid API Key:**

```json
{
  "statusCode": 401,
  "message": "Invalid API key",
  "error": "Unauthorized"
}
```

**403 Forbidden - Not Authorized for Integration:**

```json
{
  "statusCode": 403,
  "message": "unauthorized",
  "error": "Forbidden"
}
```

**404 Not Found - Campaign Not Found:**

```json
{
  "statusCode": 404,
  "message": "Campaign not found or not published",
  "error": "Not Found"
}
```

**404 Not Found - No Eligible Campaigns:**

```json
{
  "statusCode": 404,
  "message": "No eligible campaigns available for donation",
  "error": "Not Found"
}
```

**400 Bad Request - Duplicate Transaction:**

```json
{
  "statusCode": 400,
  "message": "Donation with this transaction reference already exists",
  "error": "Bad Request"
}
```

**422 Unprocessable Entity - Validation Error:**

```json
{
  "statusCode": 422,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "email must be a valid email"
    },
    {
      "field": "amount",
      "message": "amount must be a positive number"
    }
  ]
}
```

## Best Practices

### 1. Transaction References

Always provide a unique `transactionReference` for each donation:

```javascript
const transactionReference = `TXN_${Date.now()}_${userId}_${Math.random()
  .toString(36)
  .substr(2, 9)}`;
```

This enables:

- Idempotent retries
- Easy reconciliation
- Duplicate prevention

### 2. Error Handling

Implement proper error handling with retries:

```javascript
async function sendDonation(donationData, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch("/donations-integration/webhook", {
        method: "POST",
        headers: {
          "x-api-key": API_KEY,
          "Content-Type": "application/json",
        },
        body: JSON.stringify(donationData),
      });

      if (response.ok) {
        return await response.json();
      }

      // Don't retry on 4xx errors (except 429)
      if (
        response.status >= 400 &&
        response.status < 500 &&
        response.status !== 429
      ) {
        throw new Error(`Client error: ${response.status}`);
      }

      // Retry on 5xx or 429
      if (i < retries - 1) {
        await new Promise((resolve) =>
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
      }
    } catch (error) {
      if (i === retries - 1) throw error;
    }
  }
}
```

### 3. Campaign Selection

When displaying campaigns to users:

```javascript
// Fetch featured campaigns
const campaigns = await fetch(
  "/campaigns-integration?isFeatured=true&limit=20",
  {
    headers: { "x-api-key": API_KEY },
  }
);

// Allow users to select a campaign or donate to random featured campaign
const donationData = {
  email: user.email,
  amount: 50000,
  campaign: selectedCampaign?.urlSlug, // Optional
  displayName: user.name,
  transactionReference: generateUniqueRef(),
};
```

### 4. Reconciliation

Regularly fetch your donations for reconciliation:

```javascript
// Fetch all donations for the current month
const startOfMonth = new Date();
startOfMonth.setDate(1);
startOfMonth.setHours(0, 0, 0, 0);

const donations = await fetch(`/donations-integration?page=1&limit=100`, {
  headers: { "x-api-key": API_KEY },
});

// Compare with your internal records
reconcileDonations(donations.data);
```

### 5. Testing

Use the test environment for development:

```javascript
const BASE_URL =
  process.env.NODE_ENV === "production"
    ? "https://api.dynaraise.com"
    : "https://service-test.dynaraise.com";

const API_KEY =
  process.env.NODE_ENV === "production"
    ? process.env.DYNARAISE_LIVE_KEY
    : process.env.DYNARAISE_TEST_KEY;
```

### 6. Rate Limiting

Respect rate limits:

- **Default**: 100 requests per minute
- **Burst**: 20 requests per second

Implement exponential backoff for 429 responses.

### 7. Security

- Never expose API keys in client-side code
- Store keys in environment variables or secrets manager
- Rotate keys periodically
- Use HTTPS for all requests
- Validate webhook signatures (if applicable)

## Support

For integration support:

- **Email**: api-support@dynaraise.com
- **Documentation**: https://docs.dynaraise.com
- **Status Page**: https://status.dynaraise.com
