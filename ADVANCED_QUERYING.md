# Advanced Querying Guide

The Dynaraise API uses `@dataui/crud-request` for powerful querying on all list endpoints.

## Quick Start

Install the package:

```bash
npm install @dataui/crud-request
```

Build queries using `RequestQueryBuilder`:

```javascript
import { RequestQueryBuilder } from '@dataui/crud-request';

const queryString = RequestQueryBuilder.create({
  fields: ['id', 'title', 'target', 'realized'],
  search: { $and: [{ published: { $eq: 'APPROVED' } }] },
  join: [{ field: 'category' }, { field: 'user' }],
  sort: [{ field: 'createdAt', order: 'DESC' }],
  page: 1,
  limit: 25,
}).query(false);

// Result: s={"$and":[{"published":{"$eq":"APPROVED"}}]}&join[0]=category&join[1]=user&fields=id,title,target,realized&sort[0]=createdAt,DESC&limit=25&page=1
```

## Search Operators

Use these operators in the `search` parameter:

| Operator   | Description           | Example                                          |
| ---------- | --------------------- | ------------------------------------------------ |
| `$eq`      | Equals                | `{ status: { $eq: "APPROVED" } }`                |
| `$ne`      | Not equals            | `{ status: { $ne: "DECLINED" } }`                |
| `$gt`      | Greater than          | `{ target: { $gt: 1000000 } }`                   |
| `$gte`     | Greater than or equal | `{ target: { $gte: 1000000 } }`                  |
| `$lt`      | Less than             | `{ donorCount: { $lt: 100 } }`                   |
| `$lte`     | Less than or equal    | `{ donorCount: { $lte: 100 } }`                  |
| `$starts`  | Starts with           | `{ title: { $starts: "Help" } }`                 |
| `$ends`    | Ends with             | `{ title: { $ends: "School" } }`                 |
| `$cont`    | Contains              | `{ title: { $cont: "education" } }`              |
| `$excl`    | Excludes              | `{ title: { $excl: "test" } }`                   |
| `$in`      | In array              | `{ status: { $in: ["APPROVED", "PENDING"] } }`   |
| `$notin`   | Not in array          | `{ status: { $notin: ["DECLINED", "BANNED"] } }` |
| `$isnull`  | Is null               | `{ endDate: { $isnull: true } }`                 |
| `$notnull` | Is not null           | `{ endDate: { $notnull: true } }`                |
| `$between` | Between values        | `{ target: { $between: [1000000, 5000000] } }`   |

## Logical Operators

Combine conditions with `$and` or `$or`:

```javascript
// AND condition
search: {
  $and: [{ published: { $eq: 'APPROVED' } }, { target: { $gte: 1000000 } }];
}

// OR condition
search: {
  $or: [{ type: { $eq: 'PERSONAL' } }, { type: { $eq: 'ORGANIZATION' } }];
}

// Nested conditions
search: {
  $and: [
    { published: { $eq: 'APPROVED' } },
    {
      $or: [
        { title: { $cont: 'education' } },
        { summary: { $cont: 'education' } },
      ],
    },
  ];
}
```

## Common Examples

### Filter Approved Campaigns

```javascript
const query = RequestQueryBuilder.create({
  search: { $and: [{ published: { $eq: 'APPROVED' } }] },
  sort: [{ field: 'createdAt', order: 'DESC' }],
  limit: 20,
}).query(false);

// GET /campaigns-admin?s={"$and":[{"published":{"$eq":"APPROVED"}}]}&sort[0]=createdAt,DESC&limit=20
```

### Get High-Value Campaigns with Category

```javascript
const query = RequestQueryBuilder.create({
  search: {
    $and: [{ published: { $eq: 'APPROVED' } }, { target: { $gte: 5000000 } }],
  },
  join: [{ field: 'category' }],
  fields: ['id', 'title', 'target', 'realized'],
  sort: [{ field: 'target', order: 'DESC' }],
  page: 1,
  limit: 10,
}).query(false);
```

### Search Campaigns by Title

```javascript
const query = RequestQueryBuilder.create({
  search: { $and: [{ title: { $cont: 'orphan' } }] },
  join: [{ field: 'category' }, { field: 'user' }],
  limit: 25,
}).query(false);
```

### Get Recent Donations for Campaign

```javascript
const query = RequestQueryBuilder.create({
  search: {
    $and: [
      { campaignId: { $eq: 'campaign_id_here' } },
      { paymentStatus: { $eq: 'SUCCESSFUL' } },
    ],
  },
  sort: [{ field: 'createdAt', order: 'DESC' }],
  limit: 50,
}).query(false);
```

### Get Pending Payouts

```javascript
const query = RequestQueryBuilder.create({
  search: { $and: [{ payoutStatus: { $eq: 'PENDING' } }] },
  join: [{ field: 'campaign' }],
  sort: [{ field: 'createdAt', order: 'ASC' }],
}).query(false);
```

### Filter by Date Range

```javascript
const query = RequestQueryBuilder.create({
  search: {
    $and: [
      { createdAt: { $gte: '2024-01-01T00:00:00.000Z' } },
      { createdAt: { $lt: '2025-01-01T00:00:00.000Z' } },
    ],
  },
  sort: [{ field: 'createdAt', order: 'DESC' }],
}).query(false);
```

## Parameters Reference

### `search`

Filter results with conditions:

```javascript
search: {
  $and: [{ status: { $eq: 'APPROVED' } }];
}
```

### `join`

Include related entities:

```javascript
join: [{ field: 'category' }, { field: 'user' }];
```

### `sort`

Sort results:

```javascript
sort: [{ field: 'createdAt', order: 'DESC' }];
```

### `page` & `limit`

Paginate results:

```javascript
page: 1,
limit: 25
```

## URL Format

The `query(false)` method generates URL query strings:

```javascript
const queryString = RequestQueryBuilder.create({
  search: { $and: [{ title: { $cont: 'orphan' } }] },
  join: [{ field: 'category' }],
  limit: 25,
  page: 1,
  sort: [{ field: 'id', order: 'DESC' }],
}).query(false);

// Result:
// s={"$and":[{"title":{"$cont":"orphan"}}]}&join[0]=category&limit=25&page=1&sort[0]=id,DESC
```

Append to your API endpoint:

```javascript
const url = `${baseUrl}/campaigns-admin?${queryString}`;
```

## Complete Example

```javascript
import { RequestQueryBuilder } from '@dataui/crud-request';

// Build query
const queryString = RequestQueryBuilder.create({
  fields: ['id', 'title', 'summary', 'target', 'realized', 'donorCount'],
  search: {
    $and: [
      { published: { $eq: 'APPROVED' } },
      { target: { $gte: 1000000 } },
      { createdAt: { $gte: '2024-01-01T00:00:00.000Z' } },
    ],
  },
  join: [{ field: 'category' }, { field: 'user' }],
  sort: [{ field: 'realized', order: 'DESC' }],
  page: 1,
  limit: 20,
  resetCache: true,
}).query(false);

// Make request
const response = await fetch(
  `https://api.dynaraise.com/campaigns-admin?${queryString}`,
  {
    headers: {
      'x-api-key': 'dr_live_your_api_key',
      'x-organization-id': 'your_org_id',
    },
  },
);

const data = await response.json();
```

## Learn More

For complete documentation on all features and advanced usage:

**📚 Official Documentation:** https://gid-oss.github.io/dataui-nestjs-crud/requests/

## Support

For questions about querying:

- **Email**: api-support@dynaraise.com
- **Documentation**: https://docs.dynaraise.com
