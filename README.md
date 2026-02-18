# Dynaraise Integration API Documentation

Welcome to the Dynaraise Integration API documentation. This guide will help you integrate your platform with Dynaraise to manage campaigns, track donations, and handle payouts programmatically.

## Table of Contents

- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Quick Start](#quick-start)
- [Authentication](#authentication)
  - [API Key Format](#api-key-format)
- [Base URL](#base-url)
- [API Endpoints](#api-endpoints)
  - [Campaigns](#campaigns)
  - [Campaign Categories](#campaign-categories)
  - [Donations](#donations)
  - [Payouts](#payouts)
  - [Organization](#organization)
- [Integration API](#integration-api)
- [Advanced Querying](#advanced-querying)
  - [Filtering](#filtering)
  - [Sorting](#sorting)
  - [Pagination](#pagination)
  - [Field Selection](#field-selection)
  - [Joining Relations](#joining-relations)
- [Webhooks](#webhooks)
- [Entity Schemas](#entity-schemas)
- [Error Handling](#error-handling)
  - [Common HTTP Status Codes](#common-http-status-codes)
  - [Error Examples](#error-examples)
- [Rate Limits](#rate-limits)
- [Support](#support)
- [Changelog](#changelog)

## Getting Started

### Prerequisites

1. **Organization Account**: You need an active organization account on Dynaraise
2. **API Key**: Generate an API key from your organization's admin panel
3. **Webhook URL**: Configure a webhook endpoint to receive event notifications

### Quick Start

1. **Get Your API Key**
   - Log in to your Dynaraise admin dashboard
   - Navigate to Organization Settings → API Keys
   - Click "Create API Key"
   - Copy and securely store your API key (it will only be shown once)

2. **Import Postman Collection**
   - Import `dynaraise-integration-api.postman_collection.json` into Postman
   - Set your `API_KEY` and `ORGANIZATION_ID` in the collection variables
   - Start making requests!

3. **Configure Webhooks**
   - Set your webhook URL in Organization Settings
   - Implement webhook handlers for real-time event notifications
   - See [Webhooks Documentation](./WEBHOOKS.md) for details

## Authentication

All API requests must include your API key in the request headers:

```http
x-api-key: dr_live_your_api_key_here
x-organization-id: your_organization_id
```

### API Key Format

- **Live Keys**: `dr_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
- **Test Keys**: `dr_test_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

**Security Best Practices:**

- Never expose your API key in client-side code
- Store API keys securely (environment variables, secrets manager)
- Rotate keys periodically
- Use test keys for development and live keys for production

## Base URL

**Production**: `https://api.dynaraise.com`

**Test Environment**: `https://service-test.dynaraise.com`

## API Endpoints

### Campaigns

#### Create Campaign

Create a new fundraising campaign with beneficiary KYC details via the integration endpoint.

```http
POST /campaigns-admin/integration/create
```

**Request Body:**

```json
{
  "title": "Help Build a School in Rural Nigeria",
  "summary": "We are raising funds to build a school for children in rural communities",
  "description": "<p>Full campaign story with HTML formatting. This is a detailed description of the campaign goals, impact, and how funds will be used.</p>",
  "target": 5000000,
  "currency": "NGN",
  "country": "NG",
  "type": "PERSONAL",
  "model": "REGULAR",
  "payoutType": "PARTIAL",
  "category": "category_id_here",
  "isPublic": true,
  "isSubscription": false,
  "address": "123 School Road, Rural Area",
  "state": "Lagos",
  "endDate": "2025-12-31T23:59:59.000Z",
  "videoUrl": "https://youtube.com/watch?v=example",
  "manualRealized": 0,
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
}
```

**Field Descriptions:**

- `title` (required): Campaign title
- `summary` (optional): Brief campaign summary
- `description` (optional): Full campaign story with HTML formatting
- `target` (required): Fundraising goal in minor currency units (e.g., kobo for NGN)
- `currency` (required): Currency code (e.g., "NGN", "USD")
- `country` (required): Country code (e.g., "NG")
- `type` (required): Campaign type - `PERSONAL` or `ORGANIZATION`
- `model` (required): Campaign model - `REGULAR` or `SUBSCRIPTION`
- `payoutType` (optional): Payout type - `PARTIAL` (flexible) or `FULL` (all-or-nothing)
- `category` (optional): Category ID
- `isPublic` (optional): Whether campaign is publicly visible
- `isSubscription` (optional): Enable recurring donations
- `address` (optional): Campaign location address
- `state` (optional): State/region
- `endDate` (required): Campaign end date (ISO 8601)
- `videoUrl` (optional): YouTube or video URL
- `manualRealized` (optional): Manually added amount
- `beneficiary` (required): Beneficiary KYC details for payouts

**Response:**

```json
{
  "id": "campaign_id",
  "title": "Campaign Title",
  "status": "DRAFT",
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

**Webhooks Triggered:**

- `user.created` - When beneficiary user is created
- `campaign.created` - When campaign is created
- `user.kyc.pending` - When KYC validation is submitted

#### List Campaigns

Get all campaigns for your organization.

```http
GET /campaigns-admin?page=1&limit=10
```

**Query Parameters:**

- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 10, max: 100)
- `status` (optional): Filter by status (DRAFT, PENDING, APPROVED, DECLINED, ACTIVE, COMPLETED)

#### Update Campaign

Update campaign details (only for DRAFT or PENDING campaigns).

```http
PATCH /campaigns-admin/:campaignId
```

**Request Body:**

```json
{
  "title": "Updated Campaign Title",
  "summary": "Updated summary",
  "description": "<p>Updated story with more details about the campaign progress and impact...</p>",
  "target": 6000000,
  "category": "category_id_here",
  "model": "REGULAR",
  "isPublic": true,
  "isSubscription": false,
  "videoUrl": "https://youtube.com/watch?v=updated_video",
  "address": "456 Updated Address",
  "state": "Lagos",
  "endDate": "2025-12-31T23:59:59.000Z",
  "isDeactivated": false,
  "payoutType": "PARTIAL",
  "isForSelf": false,
  "manualRealized": 100000,
  "beneficiary": {
    "firstName": "John",
    "lastName": "Doe",
    "email": "john.doe@example.com",
    "phoneNumber": "+2348012345678",
    "accountNumber": "0123456789",
    "accountName": "John Doe",
    "bankName": "Access Bank",
    "bankCode": "044"
  }
}
```

**Note:** All fields are optional. Only include fields you want to update. BVN is not required for updates.

#### Bank Account Verification Guide

To ensure accurate bank account details when updating user profiles, follow these steps:

**Step 1: Get Available Banks**

First, retrieve the list of supported banks and their codes:

```http
GET /payment/banks
Authorization: Bearer <your_jwt_token>
```

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "name": "Access Bank",
      "code": "044"
    },
    {
      "name": "Guaranty Trust Bank",
      "code": "058"
    },
    {
      "name": "First Bank of Nigeria",
      "code": "011"
    }
    // ... more banks
  ]
}
```

**Step 2: Verify Account Details**

Before updating a user's bank account information, verify the account number and get the correct account name:

```http
POST /payment/verify-account
Content-Type: application/json

{
  "accountNumber": "0123456789",
  "bankCode": "044"
}
```

**Response:**

```json
{
  "account_name": "JOHN DOE",
  "account_number": "0123456789"
}
```

**Step 3: Use Verified Details**

Use the verified account name and bank details when updating the user profile to ensure accuracy and prevent payout issues.

#### Delete Campaign

Delete a campaign (only DRAFT campaigns can be deleted).

```http
DELETE /campaigns-admin/:campaignId
```

#### Request Campaign Approval

Submit campaign for admin review.

```http
POST /campaigns-admin/request-review/:campaignId
```

**Webhooks Triggered:**

- `campaign.approved` - When campaign is approved by admin

#### Upload Campaign Banner

Upload a banner image for the campaign.

```http
POST /campaigns-admin/upload-banner/:campaignId
Content-Type: multipart/form-data
```

**Form Data:**

- `bannerImage`: Image file (jpg, png, webp, max 5MB)

#### Upload Campaign Documents

Upload supporting documents for the campaign.

```http
POST /campaigns-admin/upload-files/:campaignId
Content-Type: multipart/form-data
```

**Form Data:**

- `files`: Multiple files (PDF, images, documents)

### Campaign Categories

#### Get All Categories

```http
GET /campaign-categories
```

**Response:**

```json
[
  {
    "id": "category_id",
    "name": "Education",
    "description": "Educational campaigns",
    "icon": "icon_url"
  }
]
```

### Donations

#### Get Campaign Donations

```http
GET /donations?s={"custom query here"}
```

**Using Advanced Query:**

```javascript
const query = RequestQueryBuilder.create()
  .search({ "campaign.id": "campaign_id_here" })
  .sortBy({ field: "createdAt", order: "DESC" })
  .setLimit(10)
  .setPage(1)
  .query();

// GET /donations?s={custom-query}&sort=createdAt,DESC&limit=10&page=1
```

See [Advanced Querying Guide](./ADVANCED_QUERYING.md) for more details.

**Response:**

```json
{
  "data": [
    {
      "id": "donation_id",
      "amount": 50000,
      "currency": "NGN",
      "email": "donor@example.com",
      "displayName": "Anonymous",
      "displayLocation": "Lagos, Nigeria",
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  ],
  "total": 100,
  "page": 1,
  "limit": 10
}
```

**Webhooks Triggered:**

- `donation.received` - When a new donation is made

#### Get Donation Metrics

```http
GET /donations/campaign/:campaignId/metrics
```

**Response:**

```json
{
  "totalAmount": 2500000,
  "totalDonors": 150,
  "averageDonation": 16666.67,
  "targetProgress": 50.0
}
```

### Payouts

#### Get Payout List

```http
GET /payouts?page=1&limit=10
```

**Query Parameters:**

- `page` (optional): Page number
- `limit` (optional): Items per page
- `campaignId` (optional): Filter by campaign
- `status` (optional): Filter by status (PENDING, APPROVED, PROCESSING, COMPLETED, FAILED)

#### Create Payout Request

Request a payout for a campaign.

```http
PATCH /campaigns-admin/request-payout/:campaignId
```

**Path Parameters:**

- `campaignId`: ID of the campaign to request payout for

**Requirements:**

- Campaign must be APPROVED
- Campaign must have sufficient funds
- Beneficiary KYC must be verified

### Organization

#### Get Organization Details

```http
GET /organization-admin
```

#### Update Webhook URL

```http
PATCH /organization-admin/:organizationId
```

**Request Body:**

```json
{
  "webhookUrl": "https://your-domain.com/webhooks/dynaraise"
}
```

## Advanced Querying

The Dynaraise API supports powerful querying using `@dataui/crud-request`. Build queries with `RequestQueryBuilder`:

```javascript
import { RequestQueryBuilder } from "@dataui/crud-request";

const queryString = RequestQueryBuilder.create({
  fields: ["id", "title", "target", "realized"],
  search: { $and: [{ published: { $eq: "APPROVED" } }] },
  join: [{ field: "category" }],
  sort: [{ field: "createdAt", order: "DESC" }],
  page: 1,
  limit: 25,
}).query(false);

// Use in request
fetch(`${baseUrl}/campaigns-admin?${queryString}`);
```

### Quick Examples

**Filter approved campaigns:**

```javascript
const query = RequestQueryBuilder.create({
  search: { $and: [{ published: { $eq: "APPROVED" } }] },
}).query(false);
// GET /campaigns-admin?s={"$and":[{"published":{"$eq":"APPROVED"}}]}
```

**Search by title:**

```javascript
const query = RequestQueryBuilder.create({
  search: { $and: [{ title: { $cont: "education" } }] },
}).query(false);
```

**High-value campaigns with category:**

```javascript
const query = RequestQueryBuilder.create({
  search: {
    $and: [{ published: { $eq: "APPROVED" } }, { target: { $gte: 5000000 } }],
  },
  join: [{ field: "category" }],
  sort: [{ field: "target", order: "DESC" }],
  limit: 20,
}).query(false);
```

For complete documentation including all operators and examples, see [ADVANCED_QUERYING.md](./ADVANCED_QUERYING.md).

**Official Docs:** https://gid-oss.github.io/dataui-nestjs-crud/requests/

## Integration API

For third-party organizations that want to integrate with Dynaraise and send donations on behalf of their users, we provide dedicated Integration API endpoints.

**Key Features:**

- Browse and search published campaigns
- Send donation webhooks when users donate through your platform
- Retrieve donation history for reporting and reconciliation
- Idempotent webhook handling with retry support
- Random featured campaign selection when no specific campaign is provided

**Authorization Required:**
Your organization must be authorized by a Dynaraise platform administrator to use integration donation endpoints. Contact api-support@dynaraise.com to request access.

**Documentation:**
See [INTEGRATION_API.md](./INTEGRATION_API.md) for complete integration API documentation including:

- Campaign browsing
- Donation webhook handling
- Error handling and best practices
- Code examples and testing guidelines

## Webhooks

See [WEBHOOKS.md](./WEBHOOKS.md) for detailed webhook documentation.

## Entity Schemas

See [ENTITIES.md](./ENTITIES.md) for detailed entity schema documentation.

## Error Handling

All API errors follow this format:

```json
{
  "statusCode": 400,
  "message": "Error message",
  "error": "Bad Request"
}
```

### Common HTTP Status Codes

- `200 OK` - Request successful
- `201 Created` - Resource created successfully
- `400 Bad Request` - Invalid request parameters
- `401 Unauthorized` - Invalid or missing API key
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource not found
- `422 Unprocessable Entity` - Validation error
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error

### Error Examples

**Invalid API Key:**

```json
{
  "statusCode": 401,
  "message": "Invalid API key",
  "error": "Unauthorized"
}
```

**Validation Error:**

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

## Rate Limits

- **Default**: 100 requests per minute per API key
- **Burst**: 20 requests per second

When rate limit is exceeded:

```json
{
  "statusCode": 429,
  "message": "Too many requests",
  "retryAfter": 60
}
```

**Best Practices:**

- Implement exponential backoff for retries
- Cache responses when appropriate
- Use webhooks instead of polling

## Support

For technical support or questions:

- **Email**: api-support@dynaraise.com
- **Documentation**: https://docs.dynaraise.com
- **Status Page**: https://status.dynaraise.com

## Changelog

### Version 1.0.0 (2024-01-01)

- Initial API release
- Campaign management endpoints
- Donation tracking
- Payout requests
- Webhook notifications
