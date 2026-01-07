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
  urlSlug: string; // URL-friendly slug
  summary?: string; // Brief summary (max 255 chars)
  description?: string; // Full campaign story (HTML)
  videoUrl?: string; // Video URL
  target: number; // Fundraising target amount
  currency: string; // Currency code (e.g., "NGN", "USD")
  country: string; // Country code
  address?: string; // Campaign location
  state?: string; // State/region
  type: CampaignType; // Campaign type
  model: CampaignModel; // Campaign model (regular/subscription)
  isSubscription: boolean; // Has subscription option
  isForSelf: boolean; // Campaign for self
  payoutType: CampaignPayout; // Payout model
  published: CampaignPublishedStatus; // Publishing status
  reviewNotes?: string; // Admin review notes

  // Relationships
  user?: User; // User object
  organization?: Organization; // Organization object
  parentOrganization?: Organization; // Parent organization
  category?: Category; // Category object

  // Beneficiary & Account
  beneficiary?: BeneficiaryDetails; // Beneficiary KYC details
  dedicatedAccount?: DedicatedAccount; // Dedicated account info

  // Media
  bannerImage?: File; // Banner image

  // Financial Metrics
  realized: number; // Amount raised (primary currency)
  amountsRaisedBreakdown?: AmountBreakdown; // Multi-currency breakdown
  totalRealized: number; // Total including tips and fees
  totalRealizedBreakdown?: AmountBreakdown;
  platformTip: number; // Platform tips received
  tipsBreakdown?: AmountBreakdown;
  platformCharge: number; // Platform charges
  processingFees: number; // Payment processing fees
  processingFeesBreakdown?: AmountBreakdown;
  transactionCharges: number; // Transaction charges
  transactionChargesBreakdown?: AmountBreakdown;
  manualRealized: number; // Manually added amount
  manualRealizedBreakdown?: AmountBreakdown;
  paidOut: number; // Amount paid out
  paidOutBreakdown?: AmountBreakdown;
  forfeitedAmount: number; // Forfeited amount
  forfeitedBreakdown?: AmountBreakdown;
  partialPayoutStatus?: PaymentStatus; // Partial payout status
  payoutStatus?: PaymentStatus; // Full payout status

  // Donor Metrics
  donationCount: number; // Number of donations
  donationCountBreakdown?: AmountBreakdown;

  // Status Flags
  isVerified: boolean; // Campaign verified
  isZakatable: boolean; // Eligible for Zakat
  isFlagged: boolean; // Flagged for review
  isComplete: boolean; // Campaign completed
  isDeactivated: boolean; // Campaign deactivated
  preFunded: boolean; // Pre-funded campaign
  preFundAmount?: number; // Pre-fund amount
  isPlatform: boolean; // Platform-owned campaign

  // Timestamps
  createdAt: string; // ISO 8601 timestamp
  updatedAt: string; // ISO 8601 timestamp
  publishedAt?: string; // When campaign went live
  completedAt?: string; // When campaign completed
  deactivatedAt?: string; // When campaign deactivated
  endDate?: string; // Campaign end date
}

interface BeneficiaryDetails {
  accountNumber?: string;
  accountName?: string;
  bankName?: string;
  bankCode?: string;
  bvn?: string;
  firstName?: string;
  lastName?: string;
  phoneNumber?: string;
  email?: string;
}

interface DedicatedAccount {
  accountNumber: string;
  bankName: string;
  beneficiaryName: string;
}

interface AmountBreakdown {
  NGN: number;
  USD: number;
}
```

### Campaign Types

```typescript
enum CampaignType {
  PERSONAL = "PERSONAL", // Personal fundraising
  ORGANIZATION = "ORGANIZATION", // Organization campaign
}
```

### Campaign Published Status

```typescript
enum CampaignPublishedStatus {
  PENDING = "PENDING", // Awaiting approval
  REQUESTED = "REQUESTED", // Review requested
  APPROVED = "APPROVED", // Approved and live
  DECLINED = "DECLINED", // Rejected
  BANNED = "BANNED", // Banned from platform
}
```

### Payout Models

```typescript
enum CampaignPayout {
  PARTIAL = "PARTIAL", // Receive funds anytime (flexible)
  FULL = "FULL", // Only if target is met (all or nothing)
}
```

### Campaign Model

```typescript
enum CampaignModel {
  REGULAR = "REGULAR", // One-time campaign
  SUBSCRIPTION = "SUBSCRIPTION", // Recurring donations
}
```

### Example

```json
{
  "id": "01JBCDEF123456789ABCDEFGH",
  "title": "Help Build a School in Rural Nigeria",
  "urlSlug": "help-build-school-rural-nigeria",
  "summary": "Raising funds to construct a primary school for 200 children",
  "description": "<p>Our community needs a school...</p>",
  "target": 5000000,
  "currency": "NGN",
  "country": "NG",
  "type": "PERSONAL",
  "model": "REGULAR",
  "isSubscription": false,
  "isForSelf": false,
  "payoutType": "PARTIAL",
  "published": "APPROVED",
  "realized": 2500000,
  "totalRealized": 2650000,
  "platformTip": 100000,
  "platformCharge": 50000,
  "donationCount": 150,
  "isVerified": true,
  "isComplete": false,
  "isDeactivated": false,
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

  // Relationships
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
  PLATFORM = "PLATFORM", // Platform user
  ORGANIZATION = "ORGANIZATION", // Organization admin
  ORGANIZATION_USER = "ORGANIZATION_USER", // Organization member
  EMPLOYEE = "EMPLOYEE", // Platform employee
  COLLABORATOR = "COLLABORATOR", // Campaign collaborator
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
  owner?: User; // Owner object
  parentOrganization?: Organization;

  // Media
  logo?: File; // Logo image
  bannerImage?: File; // Banner image

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
  INDIVIDUAL = "INDIVIDUAL", // Individual entity
  ORGANIZATION = "ORGANIZATION", // Registered organization
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

  // Fees & Charges
  processingFees: number; // Payment processing fees
  charges?: number; // Additional charges
  platformTip: number; // Platform tip amount
  platformCharge: number; // Platform charge
  isAddedPlatformTip: boolean; // Whether tip was added
  donorPaidFees: boolean; // Whether donor covered fees

  // Donor Information
  email?: string; // Donor email
  phoneNumber?: string; // Donor phone number
  displayName?: string; // Display name (or "Anonymous")
  displayLocation?: string; // Display location
  followConsent: boolean; // Follow-up consent
  newsletterConsent: boolean; // Newsletter consent

  // Payment
  provider: PaymentProvider; // Payment provider (PAYSTACK, etc.)
  transactionReference?: string; // Payment reference
  paymentLink?: string; // Payment link
  paymentStatus: PaymentStatus; // Payment status
  isSuccessfullyDonated: boolean; // Donation completed
  isDedicatedNuban: boolean; // Via dedicated account

  // Campaign & Subscription
  isPrefund: boolean; // Pre-fund donation
  isSubscription: boolean; // Recurring donation
  campaignCount: number; // Number of campaigns donated to

  // Relationships
  campaigns?: Campaign[]; // Campaign objects
  campaignCategories?: CampaignCategory[]; // Categories
  user?: User; // User object (if registered)
  referrer?: User; // Referrer user
  givingLevel?: CampaignLevel; // Giving level
  subscription?: CampaignSubscription; // Subscription object

  // Timestamps
  createdAt: string; // ISO 8601 timestamp
  updatedAt: string; // ISO 8601 timestamp
}

enum PaymentProvider {
  PAYSTACK = "PAYSTACK",
  FLUTTERWAVE = "FLUTTERWAVE",
  STRIPE = "STRIPE",
}
```

### Payment Status (used for Donations and Payouts)

```typescript
enum PaymentStatus {
  PENDING = "PENDING", // Payment initiated
  REQUESTED = "REQUESTED", // Payment requested
  SUCCESSFUL = "SUCCESSFUL", // Payment successful
  FAILED = "FAILED", // Payment failed
  CANCELED = "CANCELED", // Payment canceled
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
  "platformTip": 5000,
  "platformCharge": 1000,
  "isAddedPlatformTip": true,
  "donorPaidFees": true,
  "email": "donor@example.com",
  "phoneNumber": "+2348012345678",
  "displayName": "John Doe",
  "displayLocation": "Lagos, Nigeria",
  "followConsent": true,
  "newsletterConsent": false,
  "provider": "PAYSTACK",
  "transactionReference": "TXN_123456789",
  "paymentStatus": "SUCCESSFUL",
  "isSuccessfullyDonated": true,
  "isDedicatedNuban": false,
  "isPrefund": false,
  "isSubscription": false,
  "campaignCount": 1,
  "createdAt": "2024-01-05T10:30:00.000Z",
  "updatedAt": "2024-01-05T10:30:15.000Z"
}
```

## Payout

Represents a payout request for a campaign.

### Schema

```typescript
interface Payout {
  id: string; // Unique payout identifier
  payoutAmount: number; // Payout amount
  payoutAmountBreakdown?: AmountBreakdown; // Multi-currency breakdown
  currency?: string; // Currency code
  payoutStatus: PaymentStatus; // Current status
  isPartial: boolean; // Partial payout flag

  // Fees & Charges
  transactionCharges: number; // Transaction charges
  processingFees: number; // Processing fees
  platformTip: number; // Platform tip amount
  platformCharge: number; // Platform charge

  // Forfeiture
  forfeitureType?: ForfeitureType; // Forfeiture type
  forfeitureRecipientId?: string; // Recipient of forfeited amount
  forfeitedAmount?: number; // Amount forfeited

  // Relationships
  campaign?: Campaign; // Campaign object

  // Timestamps
  createdAt: string; // ISO 8601 timestamp
  updatedAt: string; // ISO 8601 timestamp
  payoutDate: string; // Payout date
  successfulAt?: string; // Success timestamp
}

enum ForfeitureType {
  PLATFORM = "PLATFORM",
  ORGANIZATION = "ORGANIZATION",
  CAMPAIGN = "CAMPAIGN",
}
```

### Example

```json
{
  "id": "01JBPAY123456789ABCDEFGH",
  "payoutAmount": 1000000,
  "payoutAmountBreakdown": {
    "NGN": 1000000,
    "USD": 0
  },
  "currency": "NGN",
  "payoutStatus": "SUCCESSFUL",
  "isPartial": true,
  "transactionCharges": 50000,
  "processingFees": 15000,
  "platformTip": 20000,
  "platformCharge": 10000,
  "forfeitureType": null,
  "forfeitureRecipientId": null,
  "forfeitedAmount": 0,
  "payoutDate": "2024-01-20T00:00:00.000Z",
  "successfulAt": "2024-01-21T02:00:00.000Z",
  "createdAt": "2024-01-20T00:00:00.000Z",
  "updatedAt": "2024-01-21T02:00:00.000Z"
}
```

## KYC

Know Your Customer verification data.

### Schema

```typescript
interface KYC {
  id: string; // Unique KYC identifier

  // Bank Details
  bankDetails?: IBankDetails; // Bank account details

  // Status
  reviewStatus: VerificationStatus; // Verification status
  statusMessage?: string; // Status message/reason

  // Documents
  idCard?: File; // ID card image
  proofOfAddress?: File; // Proof of address
  cacCertificate?: File; // CAC certificate (for orgs)
  otherFiles?: File[]; // Additional documents

  // Relationships
  organization?: Organization; // Organization object

  // Timestamps
  createdAt: string; // ISO 8601 timestamp
  updatedAt: string; // ISO 8601 timestamp
}

interface IBankDetails {
  accountNumber: string; // Bank account number
  accountName: string; // Account holder name
  bankName: string; // Bank name
  bankCode: string; // Bank code
  bvn?: string; // BVN (nullified after verification)
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
  PENDING = "PENDING", // Not yet verified
  REQUESTED = "REQUESTED", // Verification requested
  APPROVED = "APPROVED", // Verified and approved
  DECLINED = "DECLINED", // Verification declined
  BANNED = "BANNED", // Banned from platform
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
