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

### Send Donation Webhook

Send a donation webhook when a user donates through your platform. This endpoint is idempotent and supports retry logic.

**Endpoint:**

```http
POST /donations-integration/webhook
```

**Request Body:**

| Field                  | Type    | Required | Description                                                                      |
| ---------------------- | ------- | -------- | -------------------------------------------------------------------------------- |
| `campaign`             | string  | No       | Campaign URL slug or ID. If omitted, donation goes to a random featured campaign |
| `email`                | string  | Yes      | Donor email address                                                              |
| `amount`               | number  | Yes      | Donation amount in minor currency units (e.g., kobo for NGN)                     |
| `currency`             | string  | No       | Currency code (default: "NGN")                                                   |
| `displayName`          | string  | No       | Donor display name (use for non-anonymous donations)                             |
| `phoneNumber`          | string  | No       | Donor phone number                                                               |
| `displayLocation`      | string  | No       | Donor location (e.g., "Lagos, Nigeria")                                          |
| `isAnonymous`          | boolean | No       | Whether donation is anonymous (default: false)                                   |
| `platformTip`          | number  | No       | Platform tip amount                                                              |
| `transactionReference` | string  | No       | Unique transaction reference for idempotency                                     |

**Example Request:**

```bash
curl -X POST "https://service-test.dynaraise.com/donations-integration/webhook" \
  -H "x-api-key: dr_live_your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "campaign": "build-schools-nigeria",
    "email": "donor@example.com",
    "amount": 50000,
    "currency": "NGN",
    "displayName": "Jane Smith",
    "displayLocation": "Lagos, Nigeria",
    "transactionReference": "TXN_123456789"
  }'
```

**Response:**

```json
{
  "id": "donation_id",
  "email": "donor@example.com",
  "amount": 50000,
  "currency": "NGN",
  "displayName": "Jane Smith",
  "transactionReference": "TXN_123456789",
  "paymentStatus": "SUCCESSFUL",
  "isSuccessfullyDonated": true,
  "createdAt": "2024-01-01T12:00:00.000Z"
}
```

**Idempotency:**

- If a donation with the same `transactionReference` already exists:
  - **Not successfully donated**: Retries the campaign metrics update and marks as successful
  - **Already successful**: Returns `400 Bad Request` error
- This ensures safe retries without duplicate donations

**Random Campaign Selection:**

- If `campaign` is not provided, the system automatically selects a random **featured** campaign that is:
  - Published and approved
  - Not complete or deactivated
  - Not expired (endDate > now or null)

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
