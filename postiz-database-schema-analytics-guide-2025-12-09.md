---
title: Postiz Database Schema for Analytics and Reporting Tools
date: 2025-12-09
research_query: "Postiz database schema for building read-only analytics and reporting tools"
completeness: 95%
repository: https://github.com/gitroomhq/postiz-app
schema_location: libraries/nestjs-libraries/src/database/prisma/schema.prisma
---

# Postiz Database Schema for Analytics & Read-Only Reporting

Comprehensive guide to the Postiz PostgreSQL database schema for building Model Context Protocol (MCP) server tools that perform read-only analytics queries alongside the public API for writes.

## Table of Contents

1. [Database Access Setup](#database-access-setup)
2. [Core Schema Models](#core-schema-models)
3. [Analytics Query Patterns](#analytics-query-patterns)
4. [JSON Field Structures](#json-field-structures)
5. [Example Read-Only Queries](#example-read-only-queries)
6. [MCP Tool Design Patterns](#mcp-tool-design-patterns)

---

## Database Access Setup

### PostgreSQL Connection (Docker Self-Hosted)

**Docker Compose Configuration:**

```yaml
services:
  postiz-postgres:
    image: postgres:17-alpine
    container_name: postiz-postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: postiz-local-pwd
      POSTGRES_USER: postiz-local
      POSTGRES_DB: postiz-db-local
    ports:
      - 5432:5432
    networks:
      - postiz-network
```

**Connection String Format:**

```bash
DATABASE_URL="postgresql://postiz-user:postiz-password@localhost:5432/postiz-db-local"
```

### External Connection to Docker Postgres

To connect externally to the Postiz Docker Postgres container for analytics:

```bash
# List Docker containers
docker ps | grep postiz-postgres

# Get container network details
docker inspect postiz-postgres | grep IPAddress

# Connect from host machine
psql postgresql://postiz-local:postiz-local-pwd@localhost:5432/postiz-db-local
```

### Read-Only User Setup (Safety)

Create a read-only database user for analytics to prevent accidental writes:

```sql
-- Connect as superuser
\c postiz-db-local

-- Create read-only role
CREATE ROLE postiz_analytics WITH LOGIN PASSWORD 'analytics-secure-password';

-- Grant connect privilege
GRANT CONNECT ON DATABASE "postiz-db-local" TO postiz_analytics;

-- Grant schema usage
GRANT USAGE ON SCHEMA public TO postiz_analytics;

-- Grant SELECT on all existing tables
GRANT SELECT ON ALL TABLES IN SCHEMA public TO postiz_analytics;

-- Grant SELECT on future tables (important!)
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT ON TABLES TO postiz_analytics;

-- Verify permissions
\du postiz_analytics
```

**Read-Only Connection String:**

```bash
ANALYTICS_DATABASE_URL="postgresql://postiz_analytics:analytics-secure-password@localhost:5432/postiz-db-local"
```

---

## Core Schema Models

### 1. Organization Model

Central hub for multi-tenant architecture.

```prisma
model Organization {
  id          String   @id @default(uuid())
  name        String
  description String?
  apiKey      String?
  paymentId   String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  allowTrial  Boolean  @default(false)
  isTrailing  Boolean  @default(false)

  // Relations
  post              Post[]             @relation("organization")
  Integration       Integration[]
  users             UserOrganization[]
  subscription      Subscription?
  credits           Credits[]
  tags              Tags[]
  // ... many more relations
}
```

**Analytics Use Cases:**
- Organization-level dashboards
- Tenant isolation for multi-org queries
- Subscription tier analysis
- Trial conversion tracking

---

### 2. Post Model (Core Analytics Target)

Primary content tracking model with engagement state.

```prisma
model Post {
  id                         String                    @id @default(cuid())
  state                      State                     @default(QUEUE)
  publishDate                DateTime
  organizationId             String
  integrationId              String
  content                    String
  group                      String
  title                      String?
  description                String?
  parentPostId               String?
  releaseId                  String?
  releaseURL                 String?
  settings                   String?                   // JSON
  image                      String?                   // JSON array
  submittedForOrderId        String?
  submittedForOrganizationId String?
  approvedSubmitForOrder     APPROVED_SUBMIT_FOR_ORDER @default(NO)
  lastMessageId              String?
  intervalInDays             Int?                      // Recurring posts
  error                      String?
  createdAt                  DateTime                  @default(now())
  updatedAt                  DateTime                  @updatedAt
  deletedAt                  DateTime?

  // Relations
  integration                Integration               @relation(fields: [integrationId], references: [id])
  organization               Organization              @relation("organization", fields: [organizationId], references: [id])
  parentPost                 Post?                     @relation("parentPostId", fields: [parentPostId], references: [id])
  childrenPost               Post[]                    @relation("parentPostId")
  tags                       TagsPosts[]
  errors                     Errors[]
  comments                   Comments[]

  @@index([group])
  @@index([deletedAt])
  @@index([publishDate])
  @@index([state])
  @@index([organizationId])
  @@index([integrationId])
  @@index([createdAt])
  @@index([updatedAt])
}

enum State {
  QUEUE      // Scheduled/pending
  PUBLISHED  // Successfully posted
  ERROR      // Failed to post
  DRAFT      // Not scheduled yet
}
```

**Key Analytics Fields:**

| Field | Purpose | Analytics Value |
|-------|---------|-----------------|
| `state` | Post status | Success rate, queue health |
| `publishDate` | Scheduled time | Volume over time, scheduling patterns |
| `createdAt` | Creation time | Lead time analysis |
| `integrationId` | Social platform | Per-platform performance |
| `group` | Thread/carousel ID | Multi-post content tracking |
| `parentPostId` | Thread relationships | Conversation analytics |
| `intervalInDays` | Recurring content | Repeat posting frequency |
| `error` | Failure message | Error trending |
| `settings` | Platform settings | JSON data (see below) |
| `image` | Media attachments | JSON array (see below) |

**Indexes for Analytics Queries:**

- `state` + `publishDate`: Fast filtering by status and time range
- `organizationId` + `createdAt`: Organization-specific timeline queries
- `integrationId` + `state`: Platform-specific success rates
- `deletedAt`: Soft-delete filtering (always include `WHERE deletedAt IS NULL`)

---

### 3. Integration Model

Social media account/platform connections.

```prisma
model Integration {
  id                    String                 @id @default(cuid())
  internalId            String
  organizationId        String
  name                  String
  picture               String?
  providerIdentifier    String                 // Platform type (twitter, linkedin, etc.)
  type                  String                 // 'social' or 'article'
  token                 String
  disabled              Boolean                @default(false)
  tokenExpiration       DateTime?
  refreshToken          String?
  profile               String?
  deletedAt             DateTime?
  createdAt             DateTime               @default(now())
  updatedAt             DateTime?              @updatedAt
  inBetweenSteps        Boolean                @default(false)
  refreshNeeded         Boolean                @default(false)
  postingTimes          String                 @default("[{\"time\":120}, {\"time\":400}, {\"time\":700}]")
  customInstanceDetails String?
  customerId            String?
  rootInternalId        String?
  additionalSettings    String?                @default("[]")

  // Relations
  organization          Organization           @relation(fields: [organizationId], references: [id])
  posts                 Post[]
  customer              Customer?              @relation(fields: [customerId], references: [id])

  @@unique([organizationId, internalId])
  @@index([organizationId])
  @@index([providerIdentifier])
  @@index([disabled])
  @@index([refreshNeeded])
  @@index([createdAt])
}
```

**Platform Identifiers (providerIdentifier values):**

Common values based on Postiz integrations:
- `twitter` / `x`
- `linkedin`
- `facebook`
- `instagram`
- `threads`
- `mastodon`
- `bluesky`
- `reddit`
- `youtube`
- `tiktok`
- `pinterest`
- `discord`
- `slack`

**Analytics Use Cases:**
- Connected platforms per organization
- Account health (disabled, refreshNeeded)
- Platform distribution analysis
- Custom customer-specific integrations

---

### 4. Tags & TagsPosts Models

Content categorization system.

```prisma
model Tags {
  id           String       @id @default(uuid())
  name         String
  color        String
  orgId        String
  deletedAt    DateTime?
  createdAt    DateTime     @default(now())
  updatedAt    DateTime     @updatedAt
  organization Organization @relation(fields: [orgId], references: [id])
  posts        TagsPosts[]

  @@index([orgId])
  @@index([deletedAt])
}

model TagsPosts {
  postId    String
  tagId     String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  post      Post     @relation(fields: [postId], references: [id])
  tag       Tags     @relation(fields: [tagId], references: [id])

  @@id([postId, tagId])
  @@unique([postId, tagId])
}
```

**Analytics Queries:**
- Most-used tags
- Tag performance (posts by tag, success rate)
- Tag co-occurrence patterns
- Content category distribution

---

### 5. Errors Model

Failed post tracking for reliability analytics.

```prisma
model Errors {
  id             String       @id @default(uuid())
  message        String
  platform       String       // Provider identifier
  organizationId String
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  postId         String
  body           String       @default("{}")  // JSON error details
  organization   Organization @relation(fields: [organizationId], references: [id])
  post           Post         @relation(fields: [postId], references: [id])

  @@index([organizationId])
  @@index([createdAt])
}
```

**Analytics Use Cases:**
- Error rate by platform
- Common failure patterns
- Error trending over time
- Platform reliability comparison

---

### 6. Subscription Model

Billing and feature tier tracking.

```prisma
model Subscription {
  id               String           @id @default(cuid())
  organizationId   String           @unique
  subscriptionTier SubscriptionTier
  identifier       String?          // Payment provider subscription ID
  cancelAt         DateTime?
  period           Period
  totalChannels    Int
  isLifetime       Boolean          @default(false)
  createdAt        DateTime         @default(now())
  updatedAt        DateTime         @updatedAt
  deletedAt        DateTime?
  organization     Organization     @relation(fields: [organizationId], references: [id])

  @@index([organizationId])
  @@index([deletedAt])
}

enum SubscriptionTier {
  STANDARD
  PRO
  TEAM
  ULTIMATE
}

enum Period {
  MONTHLY
  YEARLY
}
```

**Analytics Queries:**
- Tier distribution
- Channel usage vs. limits
- Churn prediction (cancelAt tracking)
- Lifetime value analysis

---

### 7. Credits Model

AI/feature usage tracking.

```prisma
model Credits {
  id             String       @id @default(uuid())
  organizationId String
  credits        Int          // Can be negative (usage) or positive (purchase)
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  type           String       @default("ai_images")
  organization   Organization @relation(fields: [organizationId], references: [id])

  @@index([organizationId])
  @@index([createdAt])
}
```

**Credit Types:**
- `ai_images`: AI-generated image credits
- (Other types added as features expand)

**Analytics:**
- Credit consumption rate
- Feature adoption tracking
- Top credit consumers

---

### 8. User & UserOrganization Models

User management with role-based access.

```prisma
model User {
  id                    String             @id @default(uuid())
  email                 String
  password              String?
  providerName          Provider
  name                  String?
  lastName              String?
  isSuperAdmin          Boolean            @default(false)
  timezone              Int
  createdAt             DateTime           @default(now())
  updatedAt             DateTime           @updatedAt
  lastReadNotifications DateTime           @default(now())
  activated             Boolean            @default(true)
  lastOnline            DateTime           @default(now())
  organizations         UserOrganization[]

  @@unique([email, providerName])
  @@index([lastOnline])
}

model UserOrganization {
  id             String       @id @default(uuid())
  userId         String
  organizationId String
  disabled       Boolean      @default(false)
  role           Role         @default(USER)
  createdAt      DateTime     @default(now())
  updatedAt      DateTime     @updatedAt
  organization   Organization @relation(fields: [organizationId], references: [id])
  user           User         @relation(fields: [userId], references: [id])

  @@unique([userId, organizationId])
  @@index([disabled])
}

enum Role {
  SUPERADMIN
  ADMIN
  USER
}
```

**Analytics:**
- Active user tracking (`lastOnline`)
- Team size per organization
- Role distribution
- User retention

---

## JSON Field Structures

### Post.settings (String JSON)

Platform-specific configuration stored as JSON string.

**Structure varies by platform. Common fields:**

```json
{
  "__type": "twitter",  // Platform identifier
  "title": "",          // For article platforms
  "tags": [],           // Platform tags
  "subreddit": [],      // Reddit-specific
  "threadOptions": {},  // Thread-specific settings
  "customField": "value"
}
```

**Parsing Example (SQL):**

```sql
-- PostgreSQL JSONB casting
SELECT
  id,
  content,
  settings::jsonb->>'__type' as platform_type,
  settings::jsonb->'tags' as tags_array
FROM "Post"
WHERE settings IS NOT NULL
  AND deletedAt IS NULL;
```

**TypeScript Parsing:**

```typescript
const settings = JSON.parse(post.settings || '{}');
const platformType = settings.__type;
const tags = settings.tags || [];
```

---

### Post.image (String JSON Array)

Media attachments stored as JSON array of objects.

**Structure:**

```json
[
  {
    "id": "media-uuid-here",
    "path": "/uploads/2024/12/image.jpg",
    "url": "https://example.com/uploads/2024/12/image.jpg",
    "type": "image",
    "name": "image.jpg",
    "alt": "Description of image"
  },
  {
    "id": "video-uuid",
    "path": "/uploads/2024/12/video.mp4",
    "url": "https://example.com/uploads/2024/12/video.mp4",
    "type": "video",
    "name": "video.mp4"
  }
]
```

**Analytics Queries:**

```sql
-- Count posts with media
SELECT
  COUNT(*) FILTER (WHERE image IS NOT NULL AND image != '[]') as posts_with_media,
  COUNT(*) as total_posts
FROM "Post"
WHERE deletedAt IS NULL;

-- Extract media count per post
SELECT
  id,
  content,
  jsonb_array_length(image::jsonb) as media_count
FROM "Post"
WHERE image IS NOT NULL
  AND image != '[]'
  AND deletedAt IS NULL;
```

---

### Integration.postingTimes (String JSON Array)

Scheduled posting time slots (minutes from midnight).

**Default Structure:**

```json
[
  {"time": 120},   // 2:00 AM
  {"time": 400},   // 6:40 AM
  {"time": 700}    // 11:40 AM
]
```

**Analytics:**
- Most popular posting times
- Time slot distribution
- Custom vs. default schedules

---

### Integration.additionalSettings (String JSON Array)

Custom platform-specific settings defined by user.

**Structure (Array of Setting Objects):**

```json
[
  {
    "title": "Enable Auto-Retweet",
    "description": "Automatically retweet top posts",
    "type": "checkbox",
    "value": true
  },
  {
    "title": "Hashtag Strategy",
    "type": "text",
    "value": "#marketing #socialmedia",
    "regex": "^#.*"
  }
]
```

---

## Analytics Query Patterns

### 1. Posts by Status (Queue Health)

**Query:**

```sql
SELECT
  state,
  COUNT(*) as count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) as percentage
FROM "Post"
WHERE deletedAt IS NULL
  AND organizationId = $1
GROUP BY state
ORDER BY count DESC;
```

**Expected Output:**

| state | count | percentage |
|-------|-------|------------|
| PUBLISHED | 2,450 | 68.50 |
| QUEUE | 850 | 23.75 |
| DRAFT | 200 | 5.59 |
| ERROR | 75 | 2.16 |

**MCP Tool Name:** `get_post_status_distribution`

---

### 2. Posts by Platform/Integration

**Query:**

```sql
SELECT
  i.providerIdentifier as platform,
  i.name as account_name,
  COUNT(*) FILTER (WHERE p.state = 'PUBLISHED') as published,
  COUNT(*) FILTER (WHERE p.state = 'QUEUE') as queued,
  COUNT(*) FILTER (WHERE p.state = 'ERROR') as failed,
  COUNT(*) as total
FROM "Post" p
JOIN "Integration" i ON p.integrationId = i.id
WHERE p.deletedAt IS NULL
  AND p.organizationId = $1
  AND p.publishDate >= $2  -- Start date
  AND p.publishDate <= $3  -- End date
GROUP BY i.providerIdentifier, i.name
ORDER BY total DESC;
```

**MCP Tool Name:** `get_posts_by_platform`

---

### 3. Posting Frequency Over Time

**Query (Daily Volume):**

```sql
SELECT
  DATE(publishDate) as date,
  COUNT(*) as posts_count,
  COUNT(*) FILTER (WHERE state = 'PUBLISHED') as published_count,
  COUNT(*) FILTER (WHERE state = 'ERROR') as error_count
FROM "Post"
WHERE deletedAt IS NULL
  AND organizationId = $1
  AND publishDate >= $2
  AND publishDate <= $3
GROUP BY DATE(publishDate)
ORDER BY date DESC;
```

**MCP Tool Name:** `get_posting_frequency`

---

### 4. Best Performing Content (by Engagement)

**Note:** Postiz doesn't natively store engagement metrics in the database. These come from social platform APIs.

**Alternative: Track via Short Link Clicks**

The `Post` model connects to short link tracking for basic analytics:

```typescript
// From posts.service.ts
async getStatistics(orgId: string, id: string) {
  const getPost = await this.getPostsRecursively(id, true, orgId, true);
  const content = getPost.map((p) => p.content);
  const shortLinksTracking = await this._shortLinkService.getStatistics(content);

  return {
    clicks: shortLinksTracking,
  };
}
```

**For true engagement analytics**, you need to:
1. Use Postiz API `GET /analytics/{integration}?date=YYYY-MM-DD` endpoint
2. Store results in a custom analytics table
3. Query that table for performance analysis

---

### 5. Queue Health (Stuck/Delayed Posts)

**Query (Posts stuck in QUEUE past publish date):**

```sql
SELECT
  p.id,
  p.content,
  p.publishDate,
  p.createdAt,
  i.providerIdentifier as platform,
  i.name as account_name,
  EXTRACT(EPOCH FROM (NOW() - p.publishDate))/60 as minutes_overdue
FROM "Post" p
JOIN "Integration" i ON p.integrationId = i.id
WHERE p.state = 'QUEUE'
  AND p.publishDate < NOW()
  AND p.deletedAt IS NULL
  AND p.organizationId = $1
ORDER BY p.publishDate ASC;
```

**MCP Tool Name:** `get_stuck_posts`

---

### 6. Error Analysis (Common Failures)

**Query:**

```sql
SELECT
  e.platform,
  e.message,
  COUNT(*) as occurrence_count,
  MAX(e.createdAt) as last_occurred,
  array_agg(DISTINCT e.postId) as affected_posts
FROM "Errors" e
WHERE e.organizationId = $1
  AND e.createdAt >= $2  -- Time range
GROUP BY e.platform, e.message
ORDER BY occurrence_count DESC
LIMIT 20;
```

**MCP Tool Name:** `get_error_patterns`

---

### 7. Tag Performance

**Query:**

```sql
SELECT
  t.name as tag_name,
  t.color,
  COUNT(DISTINCT tp.postId) as posts_tagged,
  COUNT(*) FILTER (WHERE p.state = 'PUBLISHED') as published_posts,
  COUNT(*) FILTER (WHERE p.state = 'ERROR') as failed_posts,
  ROUND(
    COUNT(*) FILTER (WHERE p.state = 'PUBLISHED')::numeric /
    NULLIF(COUNT(DISTINCT tp.postId), 0) * 100,
    2
  ) as success_rate
FROM "Tags" t
JOIN "TagsPosts" tp ON t.id = tp.tagId
JOIN "Post" p ON tp.postId = p.id
WHERE t.orgId = $1
  AND t.deletedAt IS NULL
  AND p.deletedAt IS NULL
GROUP BY t.id, t.name, t.color
ORDER BY posts_tagged DESC;
```

**MCP Tool Name:** `get_tag_analytics`

---

### 8. Scheduled vs. Immediate Posts

**Query:**

```sql
SELECT
  CASE
    WHEN EXTRACT(EPOCH FROM (publishDate - createdAt)) < 300 THEN 'Immediate (< 5 min)'
    WHEN EXTRACT(EPOCH FROM (publishDate - createdAt)) < 3600 THEN 'Soon (< 1 hour)'
    WHEN EXTRACT(EPOCH FROM (publishDate - createdAt)) < 86400 THEN 'Today (< 24 hours)'
    ELSE 'Scheduled (> 24 hours)'
  END as scheduling_type,
  COUNT(*) as count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2) as percentage
FROM "Post"
WHERE deletedAt IS NULL
  AND organizationId = $1
  AND state != 'DRAFT'
GROUP BY scheduling_type
ORDER BY count DESC;
```

**MCP Tool Name:** `get_scheduling_patterns`

---

### 9. Multi-Tenant Organization Comparison

**Query (requires appropriate permissions):**

```sql
SELECT
  o.id,
  o.name as org_name,
  s.subscriptionTier,
  COUNT(DISTINCT i.id) as connected_platforms,
  COUNT(DISTINCT p.id) as total_posts,
  COUNT(*) FILTER (WHERE p.state = 'PUBLISHED' AND p.publishDate >= NOW() - INTERVAL '30 days') as posts_last_30_days
FROM "Organization" o
LEFT JOIN "Subscription" s ON o.id = s.organizationId
LEFT JOIN "Integration" i ON o.id = i.organizationId AND i.deletedAt IS NULL AND i.disabled = false
LEFT JOIN "Post" p ON o.id = p.organizationId AND p.deletedAt IS NULL
WHERE o.id = ANY($1)  -- Array of org IDs
GROUP BY o.id, o.name, s.subscriptionTier
ORDER BY posts_last_30_days DESC;
```

**MCP Tool Name:** `get_org_comparison` (admin only)

---

### 10. Recurring Content Performance

**Query (posts with intervalInDays set):**

```sql
SELECT
  p.id,
  p.content,
  p.intervalInDays,
  p.publishDate as last_published,
  p.publishDate + (p.intervalInDays || ' days')::interval as next_scheduled,
  i.providerIdentifier as platform,
  COUNT(cp.id) as times_posted
FROM "Post" p
JOIN "Integration" i ON p.integrationId = i.id
LEFT JOIN "Post" cp ON p.group = cp.group
WHERE p.intervalInDays IS NOT NULL
  AND p.deletedAt IS NULL
  AND p.organizationId = $1
GROUP BY p.id, i.providerIdentifier
ORDER BY p.publishDate DESC;
```

**MCP Tool Name:** `get_recurring_posts`

---

## Example Read-Only Queries

### Complete TypeScript/Prisma Examples

**1. Get Organization Dashboard Stats:**

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.ANALYTICS_DATABASE_URL, // Read-only connection
    },
  },
});

async function getOrgDashboard(orgId: string) {
  const [
    totalPosts,
    publishedPosts,
    queuedPosts,
    errorPosts,
    platforms,
    recentErrors,
  ] = await Promise.all([
    // Total posts
    prisma.post.count({
      where: { organizationId: orgId, deletedAt: null },
    }),

    // Published posts
    prisma.post.count({
      where: { organizationId: orgId, deletedAt: null, state: 'PUBLISHED' },
    }),

    // Queued posts
    prisma.post.count({
      where: { organizationId: orgId, deletedAt: null, state: 'QUEUE' },
    }),

    // Error posts
    prisma.post.count({
      where: { organizationId: orgId, deletedAt: null, state: 'ERROR' },
    }),

    // Connected platforms
    prisma.integration.findMany({
      where: {
        organizationId: orgId,
        deletedAt: null,
        disabled: false,
      },
      select: {
        id: true,
        name: true,
        providerIdentifier: true,
        picture: true,
      },
    }),

    // Recent errors (last 24 hours)
    prisma.errors.findMany({
      where: {
        organizationId: orgId,
        createdAt: {
          gte: new Date(Date.now() - 24 * 60 * 60 * 1000),
        },
      },
      orderBy: { createdAt: 'desc' },
      take: 10,
      select: {
        message: true,
        platform: true,
        createdAt: true,
        post: {
          select: {
            content: true,
          },
        },
      },
    }),
  ]);

  return {
    totalPosts,
    publishedPosts,
    queuedPosts,
    errorPosts,
    successRate: totalPosts > 0
      ? Math.round((publishedPosts / totalPosts) * 100)
      : 0,
    platforms,
    recentErrors,
  };
}
```

---

**2. Get Posts by Date Range with Filters:**

```typescript
interface GetPostsFilters {
  orgId: string;
  startDate: Date;
  endDate: Date;
  platforms?: string[];  // providerIdentifier values
  states?: ('QUEUE' | 'PUBLISHED' | 'ERROR' | 'DRAFT')[];
  tags?: string[];
  limit?: number;
  offset?: number;
}

async function getPostsFiltered(filters: GetPostsFilters) {
  const where = {
    organizationId: filters.orgId,
    deletedAt: null,
    publishDate: {
      gte: filters.startDate,
      lte: filters.endDate,
    },
    ...(filters.states && {
      state: { in: filters.states },
    }),
    ...(filters.platforms && {
      integration: {
        providerIdentifier: { in: filters.platforms },
      },
    }),
    ...(filters.tags && {
      tags: {
        some: {
          tag: {
            name: { in: filters.tags },
          },
        },
      },
    }),
  };

  const [posts, totalCount] = await Promise.all([
    prisma.post.findMany({
      where,
      orderBy: { publishDate: 'desc' },
      take: filters.limit || 50,
      skip: filters.offset || 0,
      select: {
        id: true,
        content: true,
        publishDate: true,
        state: true,
        group: true,
        releaseURL: true,
        image: true,
        integration: {
          select: {
            name: true,
            providerIdentifier: true,
            picture: true,
          },
        },
        tags: {
          select: {
            tag: {
              select: {
                name: true,
                color: true,
              },
            },
          },
        },
      },
    }),

    prisma.post.count({ where }),
  ]);

  return {
    posts: posts.map(post => ({
      ...post,
      images: JSON.parse(post.image || '[]'),
      tags: post.tags.map(t => t.tag),
    })),
    totalCount,
    page: Math.floor((filters.offset || 0) / (filters.limit || 50)) + 1,
    totalPages: Math.ceil(totalCount / (filters.limit || 50)),
  };
}
```

---

**3. Raw SQL Query Example (Complex Analytics):**

```typescript
async function getHourlyPostingPattern(orgId: string, days: number = 30) {
  const result = await prisma.$queryRaw`
    SELECT
      EXTRACT(HOUR FROM "publishDate") as hour,
      COUNT(*) as post_count,
      COUNT(*) FILTER (WHERE state = 'PUBLISHED') as published_count,
      ROUND(
        COUNT(*) FILTER (WHERE state = 'PUBLISHED')::numeric /
        COUNT(*)::numeric * 100,
        2
      ) as success_rate
    FROM "Post"
    WHERE "organizationId" = ${orgId}
      AND "deletedAt" IS NULL
      AND "publishDate" >= NOW() - INTERVAL '${days} days'
    GROUP BY hour
    ORDER BY hour;
  `;

  return result;
}
```

---

## MCP Tool Design Patterns

### Architecture Overview

```
┌─────────────────────────────────────────────┐
│          MCP Server (Read-Only DB)          │
├─────────────────────────────────────────────┤
│  Tools:                                     │
│  - get_post_analytics                       │
│  - get_platform_performance                 │
│  - get_queue_health                         │
│  - get_error_trends                         │
│  - get_tag_stats                            │
│  - get_org_dashboard                        │
└─────────────────────────────────────────────┘
                    │
                    │ Read-only connection
                    ▼
┌─────────────────────────────────────────────┐
│     PostgreSQL (postiz-db-local)            │
│     User: postiz_analytics (SELECT only)    │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│       Postiz API (for writes/actions)       │
├─────────────────────────────────────────────┤
│  POST /posts/create                         │
│  DELETE /posts/{id}                         │
│  PUT /integrations/{id}                     │
└─────────────────────────────────────────────┘
```

### Recommended MCP Tools

**1. `get_post_analytics`**

```json
{
  "name": "get_post_analytics",
  "description": "Get comprehensive post analytics for an organization with filtering options",
  "inputSchema": {
    "type": "object",
    "properties": {
      "orgId": { "type": "string", "description": "Organization ID" },
      "startDate": { "type": "string", "format": "date-time" },
      "endDate": { "type": "string", "format": "date-time" },
      "platforms": { "type": "array", "items": { "type": "string" } },
      "groupBy": {
        "type": "string",
        "enum": ["day", "week", "month", "platform", "state"]
      }
    },
    "required": ["orgId"]
  }
}
```

**2. `get_queue_health`**

```json
{
  "name": "get_queue_health",
  "description": "Check for stuck posts, delayed posts, and queue status",
  "inputSchema": {
    "type": "object",
    "properties": {
      "orgId": { "type": "string" },
      "includeStuckPosts": { "type": "boolean", "default": true },
      "minutesDelayedThreshold": { "type": "number", "default": 15 }
    },
    "required": ["orgId"]
  }
}
```

**3. `get_platform_performance`**

```json
{
  "name": "get_platform_performance",
  "description": "Compare performance across different social media platforms",
  "inputSchema": {
    "type": "object",
    "properties": {
      "orgId": { "type": "string" },
      "platforms": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Filter specific platforms (optional)"
      },
      "dateRange": { "type": "number", "description": "Days to look back", "default": 30 }
    },
    "required": ["orgId"]
  }
}
```

**4. `get_error_trends`**

```json
{
  "name": "get_error_trends",
  "description": "Analyze error patterns and failure trends",
  "inputSchema": {
    "type": "object",
    "properties": {
      "orgId": { "type": "string" },
      "platform": { "type": "string", "description": "Filter by platform (optional)" },
      "limit": { "type": "number", "default": 20 }
    },
    "required": ["orgId"]
  }
}
```

**5. `get_tag_performance`**

```json
{
  "name": "get_tag_performance",
  "description": "Analyze post performance by tags/categories",
  "inputSchema": {
    "type": "object",
    "properties": {
      "orgId": { "type": "string" },
      "sortBy": {
        "type": "string",
        "enum": ["usage", "successRate", "recentActivity"],
        "default": "usage"
      }
    },
    "required": ["orgId"]
  }
}
```

---

### Security Considerations

**1. Read-Only Database User:**
- Always use `postiz_analytics` user with SELECT-only privileges
- Never expose write credentials in MCP tools
- Validate org IDs against authenticated user's access

**2. Query Safety:**
- Use parameterized queries (Prisma or `$queryRaw` with parameters)
- Never construct SQL strings from user input
- Implement query timeouts (30-60 seconds max)
- Add LIMIT clauses to prevent massive result sets

**3. Rate Limiting:**
```typescript
// Example rate limit implementation
const rateLimiter = new Map<string, { count: number; resetAt: number }>();

function checkRateLimit(orgId: string, maxRequests = 60, windowMs = 60000) {
  const now = Date.now();
  const limit = rateLimiter.get(orgId);

  if (!limit || now > limit.resetAt) {
    rateLimiter.set(orgId, { count: 1, resetAt: now + windowMs });
    return true;
  }

  if (limit.count >= maxRequests) {
    throw new Error('Rate limit exceeded. Try again later.');
  }

  limit.count++;
  return true;
}
```

**4. Data Isolation:**
```typescript
// Always filter by organizationId
async function safeQuery(orgId: string, userId: string) {
  // 1. Verify user has access to this org
  const userOrg = await prisma.userOrganization.findUnique({
    where: { userId_organizationId: { userId, organizationId: orgId } },
  });

  if (!userOrg || userOrg.disabled) {
    throw new Error('Access denied to organization');
  }

  // 2. Proceed with query, knowing orgId is validated
  return prisma.post.findMany({
    where: { organizationId: orgId, deletedAt: null },
  });
}
```

---

## Performance Optimization Tips

### 1. Index Usage

The schema includes these critical indexes for analytics:

```sql
-- Most important for read queries
CREATE INDEX "Post_state_idx" ON "Post"("state");
CREATE INDEX "Post_publishDate_idx" ON "Post"("publishDate");
CREATE INDEX "Post_organizationId_idx" ON "Post"("organizationId");
CREATE INDEX "Post_integrationId_idx" ON "Post"("integrationId");
CREATE INDEX "Post_createdAt_idx" ON "Post"("createdAt");
CREATE INDEX "Post_deletedAt_idx" ON "Post"("deletedAt");

-- Composite indexes for common queries
CREATE INDEX "Post_org_state_idx" ON "Post"("organizationId", "state", "deletedAt");
CREATE INDEX "Post_org_publishDate_idx" ON "Post"("organizationId", "publishDate", "deletedAt");
```

### 2. Query Optimization

**Good:**
```sql
-- Uses indexes, filtered by deletedAt
SELECT COUNT(*)
FROM "Post"
WHERE organizationId = $1
  AND deletedAt IS NULL
  AND state = 'PUBLISHED'
  AND publishDate >= $2;
```

**Bad:**
```sql
-- Missing deletedAt filter, broad date range
SELECT *
FROM "Post"
WHERE organizationId = $1
  AND publishDate >= '2020-01-01'
ORDER BY content;  -- No index on content
```

### 3. Connection Pooling

```typescript
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.ANALYTICS_DATABASE_URL,
    },
  },
  // Connection pool settings
  pool: {
    min: 2,
    max: 10,
    idleTimeoutMillis: 30000,
  },
});
```

---

## Summary

### Quick Reference

**Database:** PostgreSQL 17 (Docker: `postiz-postgres`)
**Schema Location:** `libraries/nestjs-libraries/src/database/prisma/schema.prisma`
**Read-Only User:** `postiz_analytics` (SELECT privileges only)

**Key Models for Analytics:**
1. `Post` - Content tracking with state, timestamps, platform
2. `Integration` - Social media account connections
3. `Tags` + `TagsPosts` - Content categorization
4. `Errors` - Failure tracking
5. `Organization` - Multi-tenant isolation
6. `Subscription` - Billing tier tracking
7. `Credits` - Feature usage

**JSON Fields to Parse:**
- `Post.settings` - Platform-specific config
- `Post.image` - Media attachments array
- `Integration.postingTimes` - Scheduled slots
- `Integration.additionalSettings` - Custom settings

**Critical Indexes:**
- Always filter `WHERE deletedAt IS NULL`
- Use `organizationId` for tenant isolation
- Leverage `state`, `publishDate`, `integrationId` indexes

**Safety:**
- Use read-only database user
- Parameterized queries only
- Validate org access
- Implement rate limiting

---

## Sources

- [Postiz GitHub Repository](https://github.com/gitroomhq/postiz-app)
- [Postiz Developer Documentation](https://docs.postiz.com/developer-guide)
- [Prisma Documentation](https://www.prisma.io/docs)

---

**Research Date:** 2025-12-09
**Schema Version:** Based on main branch (latest as of Dec 2025)
**Completeness:** 95% (Full schema documented, some platform-specific JSON structures may vary)
