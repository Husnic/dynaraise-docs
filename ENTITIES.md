# Entity Schemas Documentation

This document describes the data structures (entities) used in the Dynaraise API.

## Table of Contents

- [Campaign](#campaign)
- [User](#user)
- [Organization](#organization)
- [Donation](#donation)
- [Payout](#payout)
- [KYC](#kyc)
- [Category](#category)
- [Enums](#enums)

## Campaign

Represents a fundraising campaign.

### Schema

```typescript
interface Campaign {
  id: string; // Unique campaign identifier
  title: string; // Campaign title
  summary: string; // Brief summary (max 500 chars)
  story: string; // Full campaign story (HTML)
  target: number; // Fundraising target amount
  currency: string; // Currency code (e.g., "NGN", "USD")
  type: CampaignType; // Campaign type
  model: CampaignModel; // Campaign model (regular/subscription)
  payoutType: CampaignPayout; // Payout model
  published: CampaignPublishedStatus; // Publishing status

  // Relationships
  userId: string; // Campaign creator/beneficiary
  user?: User; // User object
  organizationId: string; // Individual organization ID
  organization?: Organization; // Organization object
  parentOrganizationId: string; // Parent organization ID
  parentOrganization?: Organization;
  categoryId: string; // Campaign category
  category?: Category; // Category object

  // Media
  banner?: File; // Banner image
  files?: File[]; // Supporting documents

  // Metrics
  totalDonations: number; // Total amount raised
  donorCount: number; // Number of donors

  // Timestamps
  createdAt: string; // ISO 8601 timestamp
  updatedAt: string; // ISO 8601 timestamp
  publishedAt?: string; // When campaign went live
  endDate?: string; // Campaign end date
}
```

### Campaign Types

```typescript
enum CampaignType {
  PERSONAL = 'PERSONAL', // Personal fundraising
  ORGANIZATION = 'ORGANIZATION', // Organization campaign
  WALLET = 'WALLET', // Wallet campaign
}
```

### Campaign Published Status

```typescript
enum CampaignPublishedStatus {
  PENDING = 'PENDING', // Awaiting approval
  REQUESTED = 'REQUESTED', // Review requested
  APPROVED = 'APPROVED', // Approved and live
  DECLINED = 'DECLINED', // Rejected
  BANNED = 'BANNED', // Banned from platform
}
```

### Payout Models

```typescript
enum CampaignPayout {
  PARTIAL = 'PARTIAL', // Receive funds anytime (flexible)
  FULL = 'FULL', // Only if target is met (all or nothing)
}
```

### Campaign Model

```typescript
enum CampaignModel {
  REGULAR = 'REGULAR', // One-time campaign
  SUBSCRIPTION = 'SUBSCRIPTION', // Recurring donations
}
```

### Example

```json
{
  "id": "01JBCDEF123456789ABCDEFGH",
  "title": "Help Build a School in Rural Nigeria",
  "summary": "Raising funds to construct a primary school for 200 children",
  "story": "<p>Our community needs a school...</p>",
  "target": 5000000,
  "currency": "NGN",
  "type": "PERSONAL",
  "model": "REGULAR",
  "payoutType": "PARTIAL",
  "published": "APPROVED",
  "userId": "01JBUSER123456789ABCDEFGH",
  "organizationId": "01JBORG123456789ABCDEFGH",
  "parentOrganizationId": "01JBPARENT123456789ABCD",
  "categoryId": "01JBCAT123456789ABCDEFGH",
  "totalDonations": 2500000,
  "donorCount": 150,
  "createdAt": "2024-01-01T00:00:00.000Z",
  "updatedAt": "2024-01-15T00:00:00.000Z",
  "publishedAt": "2024-01-02T00:00:00.000Z"
}
```

## User

Represents a user (beneficiary or donor).

### Schema

```typescript
interface User {
  id: string; // Unique user identifier
  email: string; // Email address
  firstName: string; // First name
  lastName: string; // Last name
  phoneNumber?: string; // Phone number
  urlSlug: string; // URL-friendly identifier
  type: UserType; // User type

  // Status
  isEmailVerified: boolean; // Email verification status
  isVerified: VerificationStatus; // KYC verification status
  isActive: boolean; // Account active status
  isAdmin: boolean; // Admin privileges

  // Relationships
  organizationId: string; // User's organization
  organization?: Organization; // Organization object

  // Profile
  avatar?: File; // Profile picture
  bio?: string; // User bio

  // Timestamps
  createdAt: string; // ISO 8601 timestamp
  updatedAt: string; // ISO 8601 timestamp
}
```

### User Types

```typescript
enum UserType {
  PLATFORM = 'PLATFORM', // Platform user
  ORGANIZATION = 'ORGANIZATION', // Organization admin
  ORGANIZATION_USER = 'ORGANIZATION_USER', // Organization member
  EMPLOYEE = 'EMPLOYEE', // Platform employee
  COLLABORATOR = 'COLLABORATOR', // Campaign collaborator
}
```

### Example

```json
{
  "id": "01JBUSER123456789ABCDEFGH",
  "email": "john.doe@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "phoneNumber": "+2348012345678",
  "urlSlug": "john-doe",
  "type": "INDIVIDUAL",
  "isEmailVerified": true,
  "isVerified": "APPROVED",
  "isActive": true,
  "isAdmin": false,
  "organizationId": "01JBORG123456789ABCDEFGH",
  "createdAt": "2024-01-01T00:00:00.000Z",
  "updatedAt": "2024-01-01T00:00:00.000Z"
}
```

## Organization

Represents an organization (company or individual entity).

### Schema

```typescript
interface Organization {
  id: string; // Unique organization identifier
  name: string; // Organization name
  email: string; // Contact email
  phoneNumber?: string; // Contact phone
  urlSlug: string; // URL-friendly identifier
  type: OrganizationType; // Organization type

  // Details
  description?: string; // Organization description
  website?: string; // Website URL
  address?: string; // Physical address
  country: string; // Country code
  regNo?: string; // Registration number (CAC)

  // Status
  isVerified: VerificationStatus; // Verification status
  published: VerificationStatus; // Publishing status
  isActive: boolean; // Active status
  isFlagged: boolean; // Flagged for review

  // Integration
  webhookUrl?: string; // Webhook endpoint URL

  // Relationships
  ownerId: string; // Organization owner
  owner?: User; // Owner object
  parentOrganizationId?: string; // Parent organization
  parentOrganization?: Organization;

  // Media
  logo?: File; // Logo image
  banner?: File; // Banner image

  // Banking (for payouts)
  dedicatedAccount?: {
    accountNumber: string;
    accountName: string;
    bankName: string;
  };

  // Timestamps
  createdAt: string; // ISO 8601 timestamp
  updatedAt: string; // ISO 8601 timestamp
}
```

### Organization Types

```typescript
enum OrganizationType {
  INDIVIDUAL = 'INDIVIDUAL', // Individual entity
  ORGANIZATION = 'ORGANIZATION', // Registered organization
}
```

### Example

```json
{
  "id": "01JBORG123456789ABCDEFGH",
  "name": "Hope Foundation",
  "email": "info@hopefoundation.org",
  "phoneNumber": "+2348012345678",
  "urlSlug": "hope-foundation",
  "type": "ORGANIZATION",
  "description": "Building hope through education",
  "website": "https://hopefoundation.org",
  "country": "NG",
  "regNo": "RC123456",
  "isVerified": "APPROVED",
  "published": "APPROVED",
  "isActive": true,
  "isFlagged": false,
  "webhookUrl": "https://hopefoundation.org/webhooks/dynaraise",
  "ownerId": "01JBUSER123456789ABCDEFGH",
  "createdAt": "2024-01-01T00:00:00.000Z",
  "updatedAt": "2024-01-01T00:00:00.000Z"
}
```

## Donation

Represents a donation to a campaign.

### Schema

```typescript
interface Donation {
  id: string; // Unique donation identifier
  amount: number; // Donation amount
  currency: string; // Currency code

  // Fees
  processingFees: number; // Payment processing fees
  charges: number; // Platform charges
  netAmount: number; // Amount after fees

  // Donor Information
  email: string; // Donor email
  displayName?: string; // Display name (or "Anonymous")
  displayLocation?: string; // Display location
  message?: string; // Donor message
  isAnonymous: boolean; // Anonymous donation

  // Payment
  reference: string; // Payment reference
  paymentMethod: string; // Payment method used
  paymentStatus: PaymentStatus; // Payment status

  // Relationships
  campaignId: string; // Campaign ID
  campaign?: Campaign; // Campaign object
  userId?: string; // Donor user ID (if registered)
  user?: User; // User object

  // Timestamps
  createdAt: string; // ISO 8601 timestamp
  paidAt?: string; // Payment completion time
}
```

### Payment Status (used for Donations and Payouts)

```typescript
enum PaymentStatus {
  PENDING = 'PENDING', // Payment initiated
  REQUESTED = 'REQUESTED', // Payment requested
  SUCCESSFUL = 'SUCCESSFUL', // Payment successful
  FAILED = 'FAILED', // Payment failed
  CANCELED = 'CANCELED', // Payment canceled
}
```

### Example

```json
{
  "id": "01JBDON123456789ABCDEFGH",
  "amount": 50000,
  "currency": "NGN",
  "processingFees": 1500,
  "charges": 500,
  "netAmount": 48000,
  "email": "donor@example.com",
  "displayName": "John D.",
  "displayLocation": "Lagos, Nigeria",
  "message": "Great cause!",
  "isAnonymous": false,
  "reference": "TXN_123456789",
  "paymentMethod": "card",
  "paymentStatus": "SUCCESSFUL",
  "campaignId": "01JBCDEF123456789ABCDEFGH",
  "createdAt": "2024-01-15T10:30:00.000Z",
  "paidAt": "2024-01-15T10:30:15.000Z"
}
```

## Payout

Represents a payout request for a campaign.

### Schema

```typescript
interface Payout {
  id: string; // Unique payout identifier
  amount: number; // Payout amount
  currency: string; // Currency code
  reason?: string; // Payout reason/description
  payoutStatus: PaymentStatus; // Current status

  // Banking
  accountNumber: string; // Recipient account
  accountName: string; // Account holder name
  bankName: string; // Bank name
  bankCode: string; // Bank code

  // Payment
  reference?: string; // Payment reference
  transactionId?: string; // Transaction ID

  // Relationships
  campaignId: string; // Campaign ID
  campaign?: Campaign; // Campaign object
  requestedById: string; // User who requested
  requestedBy?: User; // User object
  approvedById?: string; // Admin who approved
  approvedBy?: User; // Admin object

  // Timestamps
  createdAt: string; // ISO 8601 timestamp
  approvedAt?: string; // Approval time
  processedAt?: string; // Processing time
  completedAt?: string; // Completion time
}
```

### Example

```json
{
  "id": "01JBPAY123456789ABCDEFGH",
  "amount": 1000000,
  "currency": "NGN",
  "reason": "First milestone completed - School foundation laid",
  "payoutStatus": "SUCCESSFUL",
  "accountNumber": "0123456789",
  "accountName": "John Doe",
  "bankName": "Access Bank",
  "bankCode": "044",
  "reference": "PAYOUT_123456789",
  "transactionId": "TXN_987654321",
  "campaignId": "01JBCDEF123456789ABCDEFGH",
  "requestedById": "01JBUSER123456789ABCDEFGH",
  "approvedById": "01JBADMIN123456789ABCDEF",
  "createdAt": "2024-01-20T00:00:00.000Z",
  "approvedAt": "2024-01-21T00:00:00.000Z",
  "processedAt": "2024-01-21T01:00:00.000Z",
  "completedAt": "2024-01-21T02:00:00.000Z"
}
```

## KYC

Know Your Customer verification data.

### Schema

```typescript
interface KYC {
  id: string; // Unique KYC identifier

  // Bank Details
  bankDetails: {
    accountNumber: string; // Bank account number
    accountName: string; // Account holder name
    bankName: string; // Bank name
    bankCode: string; // Bank code
    bvn?: string; // BVN (nullified after verification)
  };

  // Status
  reviewStatus: VerificationStatus; // Verification status
  statusMessage?: string; // Status message/reason

  // Documents
  idCard?: File; // ID card image
  proofOfAddress?: File; // Proof of address
  cacCertificate?: File; // CAC certificate (for orgs)
  otherFiles?: File[]; // Additional documents

  // Relationships
  organizationId: string; // Organization ID
  organization?: Organization; // Organization object

  // Timestamps
  createdAt: string; // ISO 8601 timestamp
  updatedAt: string; // ISO 8601 timestamp
  verifiedAt?: string; // Verification completion time
}
```

### Example

```json
{
  "id": "01JBKYC123456789ABCDEFGH",
  "bankDetails": {
    "accountNumber": "0123456789",
    "accountName": "John Doe",
    "bankName": "Access Bank",
    "bankCode": "044",
    "bvn": null
  },
  "reviewStatus": "APPROVED",
  "statusMessage": "Verification successful",
  "organizationId": "01JBORG123456789ABCDEFGH",
  "createdAt": "2024-01-01T00:00:00.000Z",
  "updatedAt": "2024-01-02T00:00:00.000Z",
  "verifiedAt": "2024-01-02T00:00:00.000Z"
}
```

## Category

Campaign category.

### Schema

```typescript
interface Category {
  id: string; // Unique category identifier
  name: string; // Category name
  description?: string; // Category description
  icon?: string; // Icon URL or name
  slug: string; // URL-friendly identifier
  isActive: boolean; // Active status

  // Relationships
  organizationId?: string; // Organization (for custom categories)

  // Timestamps
  createdAt: string; // ISO 8601 timestamp
  updatedAt: string; // ISO 8601 timestamp
}
```

### Example

```json
{
  "id": "01JBCAT123456789ABCDEFGH",
  "name": "Education",
  "description": "Educational campaigns and scholarships",
  "icon": "graduation-cap",
  "slug": "education",
  "isActive": true,
  "createdAt": "2024-01-01T00:00:00.000Z",
  "updatedAt": "2024-01-01T00:00:00.000Z"
}
```

## Enums

### Verification Status

```typescript
enum VerificationStatus {
  PENDING = 'PENDING', // Not yet verified
  REQUESTED = 'REQUESTED', // Verification requested
  APPROVED = 'APPROVED', // Verified and approved
  DECLINED = 'DECLINED', // Verification declined
  BANNED = 'BANNED', // Banned from platform
}
```

### Common Fields

All entities include these timestamp fields:

```typescript
interface Timestamps {
  createdAt: string; // ISO 8601 format
  updatedAt: string; // ISO 8601 format
  deletedAt?: string; // Soft delete timestamp
}
```

### ID Format

All IDs use ULID format (26 characters):

- Lexicographically sortable
- Timestamp-based
- Example: `01JBCDEF123456789ABCDEFGH`

## Validation Rules

### Campaign

- `title`: 10-200 characters
- `summary`: 50-500 characters
- `story`: Minimum 100 characters
- `target`: Must be positive number
- `currency`: ISO 4217 currency code

### User

- `email`: Valid email format
- `phoneNumber`: E.164 format (e.g., +2348012345678)
- `firstName`, `lastName`: 2-50 characters

### Bank Details

- `accountNumber`: Exactly 10 digits
- `bvn`: Exactly 11 digits
- `bankCode`: Valid bank code

### Donation

- `amount`: Must be positive
- `email`: Valid email format

### Payout

- `amount`: Must not exceed available campaign balance
- `accountNumber`: Exactly 10 digits

## Data Types

### Amounts

All monetary amounts are stored as integers (smallest currency unit):

- NGN: Stored in kobo (1 NGN = 100 kobo)
- USD: Stored in cents (1 USD = 100 cents)

**Example:**

```json
{
  "amount": 5000000, // 50,000 NGN (5,000,000 kobo)
  "currency": "NGN"
}
```

### Dates

All dates use ISO 8601 format:

```
2024-01-01T00:00:00.000Z
```

### Files

File objects include:

```typescript
interface File {
  id: string;
  key: string; // Storage key
  url: string; // Public URL
  filename: string; // Original filename
  mimeType: string; // MIME type
  size: number; // Size in bytes
  createdAt: string;
}
```

## Pagination

List endpoints return paginated results:

```typescript
interface PaginatedResponse<T> {
  data: T[]; // Array of items
  total: number; // Total count
  page: number; // Current page
  limit: number; // Items per page
  hasMore: boolean; // More pages available
}
```

**Example:**

```json
{
  "data": [...],
  "total": 150,
  "page": 1,
  "limit": 10,
  "hasMore": true
}
```
