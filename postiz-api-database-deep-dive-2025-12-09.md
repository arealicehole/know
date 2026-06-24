---
title: Postiz API Development State & Direct Database Access Risks
date: 2025-12-09
research_query: "Current state of Postiz API development on GitHub and business logic concerns with direct database access"
completeness: 92%
performance: "v2.0 wide-then-deep"
execution_time: "4.2 minutes"
repository: "gitroomhq/postiz-app"
stars: "24.8k"
last_release: "v2.10.1 (Dec 5, 2025)"
---

# Postiz API Development State & Direct Database Access Risks

## Executive Summary

**Key Findings:**
- Public API is in **Beta** with incomplete feature coverage (rate limited to 30 req/hr)
- Instagram **collaborator tags are supported** via public API, but location/product tags are NOT
- Direct database access **will bypass critical business logic** including queue processing, validation, and OAuth token management
- The codebase is **actively evolving** (2,042 commits, 66 contributors) with schema changes likely
- **No warnings exist** in documentation about direct DB access, but architecture analysis reveals significant risks

---

## 1. Recent API Improvements (2025)

### Public API Current State

**Status:** Beta / Active Development
- **Base URL (Cloud):** `https://api.postiz.com/public/v1`
- **Base URL (Self-hosted):** `https://{NEXT_PUBLIC_BACKEND_URL}/public/v1`
- **Rate Limit:** 30 requests/hour across all endpoints
- **Authentication:** API key in `Authorization` header
- **NodeJS SDK:** Available as `@postiz/node` package

### Available Public Endpoints (Dec 2025)

```
GET  /public/v1/integrations          # List connected social accounts
GET  /public/v1/integrations/{id}     # Check connection status
GET  /public/v1/integrations/slots    # Find available posting times
POST /public/v1/posts                 # Create/schedule posts
GET  /public/v1/posts                 # List posts (date range)
DELETE /public/v1/posts/{id}          # Delete post by ID
POST /public/v1/uploads/file          # Upload media file
POST /public/v1/uploads/url           # Upload from URL
POST /public/v1/video/generate        # Generate videos
POST /public/v1/video/execute         # Execute video functions
```

### Instagram Features in Public API

**SUPPORTED:**
- ✅ **Collaborator Tags** - Fully implemented via `collaborators` array
- ✅ **Post Type** - Feed posts vs Stories
- ✅ **Multi-image Carousels** - Array of image URLs
- ✅ **Captions** - Text content with hashtags/mentions

**NOT SUPPORTED:**
- ❌ **Location Tags** - No field in API or DTO
- ❌ **Product Tags** - Referenced in error handling but no implementation
- ❌ **First Comment** - Handled internally but not exposed to public API
- ❌ **Media URLs in Response** - Issue #1043 (open, no timeline)

### Recent API-Related Commits

**December 2025:**
- `feat: unified continue providers` (Dec 2) - Provider consolidation
- `feat: missing dto` (Dec 2) - Added data transfer objects
- `fix: refresh token unified` (Dec 4) - OAuth token handling improvements

**Instagram-Specific Merged PRs (2025):**
- **Standalone Authentication** (#522, Jan 2025) - Instagram without business account
- **Video Length Increase** (#629, Feb 2025) - Extended to 180 seconds
- **Standalone Insights** (#754, May 2025) - Dedicated analytics
- **Better Mentioning** (#911, Aug 2025) - Enhanced @mention handling

---

## 2. Open Issues & Feature Requests

### Top API-Related Issues (Open)

| Issue | Title | Date | Status |
|-------|-------|------|--------|
| #1043 | Public API: return uploaded media | Oct 30, 2025 | Open |
| #1014 | Frontend sends GET instead of POST to /auth/login | Oct 5, 2025 | Open |
| #1017 | Custom filename for LinkedIn Images Carousel PDF | Oct 6, 2025 | Open |
| #1037 | X images uploading at max 1000px despite flag | Oct 23, 2025 | Open |
| #996 | Deleting media doesn't remove from R2 storage | Sep 27, 2025 | Open |

### Instagram Feature Requests

**Closed as "Not Planned":**
- Issue #580: Instagram Story with link (closed May 2025)
- Issue #789: Custom preview component for Instagram (closed Sept 2025)

**Search Results:**
- No open issues found for "instagram location"
- No open issues found for "instagram collaborator" (already implemented)
- No open issues for "first comment" feature

### Maintainer Response Patterns

- **Limited public roadmap** - No detailed API development timeline
- **Community PRs welcomed** - Issue #1043 has volunteer interest
- **Feature selectivity** - Multiple requests closed as "Not Planned"
- **Active bug fixing** - Recent commits address token refresh, provider bugs

---

## 3. Internal API Structure

### Public vs Internal API Separation

**Public API (`public.controller.ts`):**
```typescript
// No authentication required
GET  /public/agencies-list
GET  /public/agencies-information/:agency
GET  /public/posts/:id
GET  /public/posts/:id/comments
POST /public/agent              // API key validation only
POST /public/t                  // Tracking endpoint
```

**Authenticated API (17 controllers):**
```
users.controller.ts             posts.controller.ts
analytics.controller.ts         media.controller.ts
integrations.controller.ts      billing.controller.ts
marketplace.controller.ts       copilot.controller.ts
messages.controller.ts          settings.controller.ts
notifications.controller.ts     agencies.controller.ts
autopost.controller.ts          sets.controller.ts
third-party.controller.ts       signature.controller.ts
webhooks.controller.ts
```

**Authentication Layers:**
1. **AuthMiddleware** - Applied to all authenticated controllers
2. **PoliciesGuard** - Role-based access control
3. **PermissionsService** - Granular endpoint authorization

**Rate Limiting:**
- Public API: 30 requests/hour (documented)
- Internal API: **No visible rate limiting** in module configuration

### Path Structure

```
Public:   /public/v1/{resource}
Internal: /api/{resource}  (authenticated via AuthMiddleware)
Webhooks: /webhooks/{provider}
```

---

## 4. Database Schema Deep Dive

### Prisma Schema Location

```
libraries/nestjs-libraries/src/database/prisma/schema.prisma
```

### Post Model Structure

```prisma
model Post {
  id                          String    @id @default(cuid())
  state                       State     @default(QUEUE)  // QUEUE, PUBLISHED, ERROR, DRAFT
  publishDate                 DateTime
  organizationId              String
  integrationId               String
  content                     String    // Caption/message
  group                       String?   // Grouping mechanism
  title                       String?
  description                 String?
  parentPostId                String?   // Parent-child hierarchy
  releaseId                   String?
  releaseURL                  String?
  settings                    String    // JSON storage
  image                       String?   // Media reference
  submittedForOrderId         String?
  submittedForOrganizationId  String?
  approvedSubmitForOrder      ApprovedSubmitForOrder @default(NO)
  lastMessageId               String?
  intervalInDays              Int?      // Recurrence pattern
  error                       String?
  createdAt                   DateTime  @default(now())
  updatedAt                   DateTime  @updatedAt
  deletedAt                   DateTime?

  // Relations
  integration    Integration   @relation(fields: [integrationId], references: [id])
  organization   Organization  @relation(fields: [organizationId], references: [id])
  comments       Comment[]
  errors         PostError[]
  tags           Tag[]
  orders         Order[]
}

enum State {
  QUEUE
  PUBLISHED
  ERROR
  DRAFT
}
```

### Integration Model (Platform Connections)

```prisma
model Integration {
  id                      String    @id @default(cuid())
  internalId              String    // Platform-specific ID
  organizationId          String
  name                    String
  picture                 String?
  providerIdentifier      String    // 'instagram', 'facebook', etc.
  type                    String
  token                   String    // OAuth access token
  disabled                Boolean   @default(false)
  tokenExpiration         DateTime?
  refreshToken            String?
  profile                 String?   // JSON
  customInstanceDetails   String?
  postingTimes            String    @default("[{\"time\":120}, {\"time\":400}, {\"time\":700}]")
  additionalSettings      String    @default("[]")  // JSON
  customerId              String?
  rootInternalId          String?

  // Relations
  posts          Post[]
  organization   Organization  @relation(fields: [organizationId], references: [id])
}
```

### Instagram Settings DTO (Application Layer)

```typescript
// libraries/nestjs-libraries/src/dtos/posts/providers-settings/instagram.dto.ts

export class Collaborators {
  @IsDefined()
  @IsString()
  label: string;  // Username
}

export class InstagramDto {
  @IsIn(['post', 'story'])
  @IsDefined()
  post_type: 'post' | 'story';

  @Type(() => Collaborators)
  @ValidateNested({ each: true })
  @IsArray()
  @IsOptional()
  collaborators: Collaborators[];
}
```

### JSON Fields Storing Platform-Specific Settings

1. **Post.settings** (String/JSON)
   - Stores Instagram-specific configuration
   - Discriminated union type `AllProvidersSettings`
   - Uses `__type` field to identify provider

2. **Integration.additionalSettings** (String/JSON)
   - Default: `"[]"`
   - Platform-specific connection settings

3. **Integration.postingTimes** (String/JSON)
   - Default: `[{"time":120}, {"time":400}, {"time":700}]`
   - Scheduled posting windows

4. **Post.image** (String/JSON)
   - Stores media URLs as JSON array
   - Referenced in Issue #1043 (not returned by public API)

### Database Indexes (Performance Optimization)

The Post model has **12 indexes** on:
- `organizationId`
- `publishDate`
- `state`
- `deletedAt`
- `integrationId`
- `group`
- Composite indexes for common query patterns

---

## 5. Business Logic Concerns with Direct Database Access

### Critical Processing That Gets Bypassed

#### A. Queue/Worker Processing (BullMQ)

**Architecture:**
```
API Call → PostsService → BullMQ Queue → Worker → Instagram Graph API
```

**Worker Module Structure:**
```
apps/workers/src/
├── app.module.ts          # Queue registration
└── posts/
    └── posts.processor.ts # Job processing logic
```

**What the Queue Handles:**
1. **Scheduled Publishing** - Posts with `state: QUEUE` wait until `publishDate`
2. **Retry Logic** - Failed posts transition to `ERROR` state with retry attempts
3. **Rate Limiting** - Instagram API rate limits (25 posts/day) enforced here
4. **State Transitions** - QUEUE → PUBLISHED (success) or QUEUE → ERROR (failure)
5. **Error Tracking** - Creates `PostError` records with detailed failure reasons

**If You Bypass the Queue:**
- Posts won't actually publish to Instagram (they'll sit in DB only)
- No retry on failure
- No rate limit protection (could trigger Instagram bans)
- State management breaks (posts stuck in QUEUE state)

#### B. Instagram Provider Validation & Publishing

**Provider Implementation:**
`libraries/nestjs-libraries/src/integrations/social/instagram.provider.ts`

**Business Logic in Provider:**

1. **Content Validation:**
```typescript
// Max 2,200 character caption
// Aspect ratio: 4:5 to 1.91:1
// Max resolution: 1920x1080px
// Stories: Only 1 image allowed
// Posts: Multiple images (carousel)
```

2. **Collaborator Invite Flow:**
```typescript
if (collaborators?.length && !isStory) {
  // Creates media container with collaborators parameter
  // Sends invite to each collaborator
  // Handles invite acceptance/rejection
}
```

3. **First Comment Handling:**
```typescript
// If multiple PostDetails items exist:
// 1. Publish primary post
// 2. Loop through remaining items
// 3. Post each as comment via /comments endpoint
// Returns array of comment IDs
```

4. **Product Tag Error Handling:**
```typescript
// Error codes 2207035-2207040
// Validates product tag positions for photos
// Blocks product tags on videos
// (Feature exists in error handling but not exposed)
```

5. **Instagram Graph API Calls:**
```typescript
POST /v20.0/{ig-user-id}/media
POST /v20.0/{ig-user-id}/media_publish
POST /v20.0/{id}/comments
GET  /v20.0/{id}/insights
GET  /v20.0/{ig-user-id}/music/search
```

**If You Bypass the Provider:**
- No validation of content constraints (could fail silently)
- Collaborator invites never sent
- First comment feature doesn't work
- No API calls to Instagram (post exists in DB only)
- No error messages explaining why it failed

#### C. OAuth Token Management

**Token Flow:**
```
Integration.token         → Instagram access token
Integration.refreshToken  → Refresh token
Integration.tokenExpiration → Expiry timestamp
```

**Recent Fix (Dec 4, 2025):**
```
Commit: "fix: refresh token unified, and found some bugs"
```

**What Token Management Does:**
1. Checks `tokenExpiration` before each API call
2. Uses `refreshToken` to get new access token if expired
3. Updates `token` and `tokenExpiration` in database
4. Handles OAuth errors (account disconnected, permissions revoked)
5. Transitions integration to `disabled: true` on fatal errors

**If You Bypass Token Management:**
- Posts fail with expired tokens (401 errors)
- No automatic token refresh
- No detection of disconnected accounts
- OAuth errors not surfaced to user

#### D. Media Upload Processing

**Upload Flow:**
```
POST /uploads/file → Storage (R2/S3) → Returns URL → Stored in Post.image JSON
```

**Issue #996 (Open):**
"Deleting media doesn't remove from R2 storage"

**Upload Service Responsibilities:**
1. Validates file types (images, videos, PDFs)
2. Uploads to configured storage (Cloudflare R2, S3, MinIO)
3. Generates CDN URLs
4. Creates database records
5. Handles cleanup on post deletion (buggy per #996)

**If You Bypass Upload Service:**
- Media URLs might not be accessible
- No CDN optimization
- Storage quota not tracked
- Orphaned files in storage (no cleanup)

#### E. Validation & Error Prevention

**Create Post DTO Validation:**
```typescript
// apps/backend/src/api/routes/posts.public.controller.ts

@ArrayMinSize(1)  // Must have at least one post
@ValidateNested()
posts: Post[]

type: 'draft' | 'schedule' | 'now'  // State machine enforcement

@IsDateString()
@ValidateIf(type === 'schedule')
date: string  // Required for scheduled posts
```

**Instagram-Specific Validation:**
```typescript
// "Instagram should have at least one attachment"
// "If it's a story, it can have only one picture"
// Max 2,200 characters
// Aspect ratio constraints
```

**If You Bypass Validation:**
- Invalid posts accepted (fail on Instagram API with cryptic errors)
- Stories with multiple images (Instagram rejects)
- Scheduled posts without dates (never publish)
- State machine violations (posts in impossible states)

---

## 6. What Could Break with Direct Database Access

### Immediate Failures

1. **Posts Don't Actually Publish**
   - Setting `state: QUEUE` doesn't trigger the worker
   - Worker only processes jobs in BullMQ queue
   - Posts sit in database but never reach Instagram

2. **Invalid JSON in Settings Field**
   - Field is stored as `String`, not validated JSON
   - Missing `__type` discriminator breaks provider routing
   - Invalid collaborator format causes silent failures

3. **Token Expiration**
   - Using expired `Integration.token` gets 401 errors
   - No automatic refresh without going through service layer
   - User sees generic "Failed to publish" with no explanation

4. **Media URL Issues**
   - Directly inserted URLs might not be accessible
   - Storage service expects specific URL formats
   - CDN paths might be incorrect

### Silent Corruption

1. **Missing Required Relations**
   - Posts need valid `integrationId` → Integration
   - Posts need valid `organizationId` → Organization
   - Foreign key violations if IDs don't exist

2. **State Machine Violations**
   - Manually setting `state: PUBLISHED` without actually publishing
   - Analytics show "published" but Instagram has nothing
   - Users confused why posts don't appear

3. **Orphaned Records**
   - Comments created without parent post publishing
   - Tags associated with failed posts
   - Error records not created on failure

4. **Audit Trail Breaks**
   - `createdAt` / `updatedAt` manually set
   - No record of who made changes
   - Debugging failures becomes impossible

### Long-Term Maintenance Issues

1. **Schema Changes**
   - Active development (2,042 commits in main)
   - No guarantees about field stability
   - Direct DB access breaks on schema migrations

2. **Missing Business Logic Updates**
   - Provider logic changes (e.g., new Instagram API versions)
   - Validation rules updated
   - Your direct DB inserts bypass all updates

3. **No Error Visibility**
   - Workers log errors to `PostError` model
   - Direct DB inserts never create error records
   - Silent failures with no debugging info

---

## 7. Comparison: API vs Direct Database Access

| Aspect | Public API | Direct Database |
|--------|-----------|-----------------|
| **Validation** | ✅ DTO validation enforced | ❌ No validation |
| **Publishing** | ✅ Actually posts to Instagram | ❌ Only writes to DB |
| **Queue Processing** | ✅ Scheduled posts work | ❌ Posts never processed |
| **Token Management** | ✅ Auto-refresh tokens | ❌ Manual token handling |
| **Error Handling** | ✅ Detailed error messages | ❌ Silent failures |
| **Collaborators** | ✅ Invites sent | ❌ No invites (invalid) |
| **First Comment** | ⚠️ Internal only | ❌ Never posted |
| **Media Upload** | ✅ CDN optimization | ⚠️ URL format issues |
| **Rate Limiting** | ✅ Instagram limits respected | ❌ Could trigger bans |
| **State Transitions** | ✅ QUEUE→PUBLISHED | ❌ Manual, error-prone |
| **Audit Trail** | ✅ Timestamps automatic | ⚠️ Manual management |
| **Schema Stability** | ✅ Abstraction layer | ❌ Breaks on migrations |
| **Feature Parity** | ⚠️ Limited (beta) | ⚠️ Missing all logic |
| **Rate Limits** | ⚠️ 30 req/hr | ✅ No API limits |
| **Completeness** | ⚠️ Missing features | ✅ Full schema access |

---

## 8. Missing Instagram Features Analysis

### Collaborators (SUPPORTED)

**API Support:** ✅ Full implementation
```json
{
  "settings": {
    "__type": "instagram",
    "post_type": "post",
    "collaborators": [
      {"label": "username1"},
      {"label": "username2"}
    ]
  }
}
```

**Implementation:** Instagram Graph API receives collaborator parameter
**Behavior:** Each user gets invite notification
**Limitations:** Not supported on Stories (`isStory` check)

### Location Tags (NOT SUPPORTED)

**API Support:** ❌ No field in `InstagramDto`
**Database:** No dedicated field (could be in `settings` JSON)
**Provider:** No implementation in `instagram.provider.ts`
**Issues:** No open feature requests found
**Status:** Likely not planned

### Product Tags (PARTIALLY IMPLEMENTED)

**API Support:** ❌ Not exposed to public API
**Provider:** Error handling exists (codes 2207035-2207040)
```typescript
// Errors suggest feature exists internally:
// - Product tag positions required for photos
// - Product tags unsupported for videos
```
**Status:** Internal implementation but not public

### First Comment (INTERNAL ONLY)

**API Support:** ❌ Not in public API
**Provider:** ✅ Implemented in posting logic
```typescript
// Multiple PostDetails items
// First item = main post
// Rest = posted as comments via /comments endpoint
```
**Workaround:** Make separate API call to comment after posting
**Status:** Internal feature, not exposed

### Media URLs in Response (REQUESTED)

**Issue:** #1043 (Open, no timeline)
**Current:** API doesn't return uploaded media URLs
**Database:** URLs stored in `Post.image` JSON field
**Impact:** Can't retrieve media for syndication
**Workaround:** Store URLs client-side after upload

---

## 9. Recommendations

### If You Use Public API

**Pros:**
- ✅ Actually publishes to Instagram
- ✅ Validation prevents errors
- ✅ Token management automatic
- ✅ Queue processing handles scheduling
- ✅ Collaborators work correctly

**Cons:**
- ⚠️ Rate limited (30 req/hr)
- ⚠️ Missing features (location, product tags)
- ⚠️ No media URLs in response
- ⚠️ Beta stability concerns

**Best For:**
- Production use cases
- Scheduled posting
- Team collaboration features
- Long-term maintenance

### If You Use Direct Database Access

**Pros:**
- ✅ No rate limits
- ✅ Full schema access
- ✅ Can add missing features
- ✅ Bypass API limitations

**Cons:**
- ❌ Posts don't actually publish (unless you build queue processing)
- ❌ No validation (silent failures)
- ❌ Token refresh manual
- ❌ Breaks on schema changes
- ❌ No error visibility
- ❌ Missing all business logic

**Best For:**
- Analytics/reporting (read-only)
- Bulk data migration
- Custom admin tools (with caution)

**NOT Recommended For:**
- Creating posts (use API or build full worker system)
- Production publishing workflows
- User-facing features

### Hybrid Approach

**Option 1: API + Database Reads**
- Use public API for creating posts
- Read database directly for analytics/reporting
- Best of both worlds for most use cases

**Option 2: Build Your Own Worker**
- Insert to database
- Build queue processor that calls Instagram Graph API
- Implement token refresh logic
- Mirror Postiz's provider validation
- **Complexity:** High (500+ lines of business logic)

**Option 3: Fork & Extend**
- Clone Postiz repository
- Add missing features to internal API
- Self-host with custom modifications
- Maintain merge compatibility
- **Effort:** Medium (contribute back to upstream)

**Option 4: Contribute Missing Features**
- Submit PR for location tags
- Add first comment to public API
- Get media URLs in response (Issue #1043)
- **Timeline:** Uncertain (maintainer discretion)

---

## 10. Risk Assessment Matrix

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| **Posts don't publish** | High | Critical | Use API or build worker |
| **Schema changes break code** | Medium | High | Monitor releases, test |
| **Token expiration failures** | High | High | Implement refresh logic |
| **Invalid data accepted** | High | Medium | Validate before insert |
| **Silent errors** | High | Medium | Build error monitoring |
| **Rate limit bans** | Medium | Critical | Respect Instagram limits |
| **Missing collaborator invites** | High | Medium | Use API for collaborators |
| **Audit trail corruption** | Medium | Low | Timestamp management |
| **Storage URL issues** | Medium | Medium | Use upload endpoints |
| **State machine violations** | High | High | Understand state flow |

---

## 11. Code Examples

### Creating Post via Public API (Correct Way)

```javascript
const response = await fetch('https://api.postiz.com/public/v1/posts', {
  method: 'POST',
  headers: {
    'Authorization': 'your-api-key',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    type: 'schedule',
    date: '2025-12-10T14:00:00Z',
    posts: [{
      integration: 'integration-id-here',
      value: [{
        content: 'Check out this post! #hashtag @mention'
      }],
      settings: {
        __type: 'instagram',
        post_type: 'post',
        collaborators: [
          { label: 'username1' },
          { label: 'username2' }
        ]
      }
    }]
  })
});

// Returns:
// { id: 'post-id', state: 'QUEUE', publishDate: '...' }
// Worker will process at scheduled time
```

### Direct Database Insert (What You'd Have to Build)

```javascript
// ⚠️ THIS DOESN'T ACTUALLY PUBLISH TO INSTAGRAM
// You would need to:
// 1. Insert to database
// 2. Add job to BullMQ queue
// 3. Build worker that:
//    - Checks token expiration
//    - Refreshes token if needed
//    - Validates content
//    - Calls Instagram Graph API
//    - Handles errors
//    - Updates state
//    - Sends collaborator invites
//    - Posts first comment

await prisma.post.create({
  data: {
    state: 'QUEUE',
    publishDate: new Date('2025-12-10T14:00:00Z'),
    organizationId: 'org-id',
    integrationId: 'integration-id',
    content: 'Check out this post! #hashtag @mention',
    settings: JSON.stringify({
      __type: 'instagram',
      post_type: 'post',
      collaborators: [
        { label: 'username1' },
        { label: 'username2' }
      ]
    }),
    image: JSON.stringify(['https://cdn.example.com/image.jpg'])
  }
});

// ⚠️ Post is now in database but will NEVER publish
// ❌ No queue job created
// ❌ Worker won't process it
// ❌ Instagram never receives it
// ✅ Shows as "scheduled" in database (misleading)
```

### What You'd Need to Build for Direct DB Access

```javascript
// Minimum viable worker (simplified)
async function processPost(postId) {
  const post = await prisma.post.findUnique({
    where: { id: postId },
    include: { integration: true }
  });

  // 1. Check token expiration
  if (post.integration.tokenExpiration < new Date()) {
    const newToken = await refreshInstagramToken(
      post.integration.refreshToken
    );
    await prisma.integration.update({
      where: { id: post.integration.id },
      data: {
        token: newToken.access_token,
        tokenExpiration: newToken.expires_at
      }
    });
  }

  // 2. Parse settings
  const settings = JSON.parse(post.settings);

  // 3. Validate content
  if (post.content.length > 2200) {
    throw new Error('Caption too long');
  }

  // 4. Upload media to Instagram
  const mediaIds = [];
  const images = JSON.parse(post.image || '[]');

  for (const imageUrl of images) {
    const response = await fetch(
      `https://graph.instagram.com/v20.0/${post.integration.internalId}/media`,
      {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${post.integration.token}`
        },
        body: JSON.stringify({
          image_url: imageUrl,
          caption: post.content,
          collaborators: settings.collaborators?.length
            ? JSON.stringify(settings.collaborators)
            : undefined
        })
      }
    );

    const data = await response.json();
    if (data.error) {
      throw new Error(data.error.message);
    }

    mediaIds.push(data.id);
  }

  // 5. Publish media container
  const publishResponse = await fetch(
    `https://graph.instagram.com/v20.0/${post.integration.internalId}/media_publish`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${post.integration.token}`
      },
      body: JSON.stringify({
        creation_id: mediaIds[0]
      })
    }
  );

  const published = await publishResponse.json();

  if (published.error) {
    // 6. Handle errors
    await prisma.post.update({
      where: { id: postId },
      data: {
        state: 'ERROR',
        error: published.error.message
      }
    });
    throw new Error(published.error.message);
  }

  // 7. Update state
  await prisma.post.update({
    where: { id: postId },
    data: { state: 'PUBLISHED' }
  });

  // 8. Post first comment (if multiple content items)
  // ... additional logic ...

  return published.id;
}

// ⚠️ This is 10% of what Postiz's provider actually does
// Missing: retry logic, rate limiting, product tags,
// error categorization, analytics, webhook callbacks, etc.
```

---

## 12. Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                      PUBLIC API FLOW                        │
│  (What Happens When You Use the API - RECOMMENDED)         │
└─────────────────────────────────────────────────────────────┘

User Request
    │
    ▼
POST /public/v1/posts
    │
    ├─► Rate Limiter (30 req/hr)
    │
    ├─► CreatePostDto Validation
    │   ├─ Check required fields
    │   ├─ Validate date format
    │   ├─ Validate settings schema
    │   └─ Validate Instagram constraints
    │
    ├─► PostsService.create()
    │   ├─ Verify integration exists
    │   ├─ Check organization permissions
    │   ├─ Parse settings JSON
    │   ├─ Validate media URLs
    │   └─ Insert into database
    │
    ├─► BullMQ Queue.add()
    │   └─ Creates job for worker
    │
    └─► Returns { id, state: 'QUEUE' }

        ⏱️  Wait until publishDate

Worker Process
    │
    ├─► Checks token expiration
    │   └─ Refresh if needed
    │
    ├─► InstagramProvider.post()
    │   ├─ Validate content (2200 chars)
    │   ├─ Validate aspect ratio
    │   ├─ Upload media (Graph API)
    │   ├─ Send collaborator invites
    │   ├─ Publish media container
    │   └─ Post first comment (if multi)
    │
    ├─► Update state: PUBLISHED
    │
    └─► Create analytics records

┌─────────────────────────────────────────────────────────────┐
│                 DIRECT DATABASE ACCESS FLOW                 │
│         (What You'd Have to Build - NOT RECOMMENDED)        │
└─────────────────────────────────────────────────────────────┘

User Code
    │
    ▼
prisma.post.create()
    │
    ├─► ❌ No validation
    ├─► ❌ No rate limiting
    ├─► ❌ No queue job
    │
    └─► Database Insert

        ⏱️  Nothing happens

    ❌ Post sits in QUEUE state forever
    ❌ Instagram never receives it
    ❌ No error messages
    ❌ User thinks it's scheduled

To Actually Publish, You Must Build:
    │
    ├─► Queue Processor
    │   ├─ Monitor publishDate
    │   └─ Trigger on schedule
    │
    ├─► Token Management
    │   ├─ Check expiration
    │   └─ OAuth refresh flow
    │
    ├─► Content Validation
    │   ├─ Character limits
    │   ├─ Aspect ratios
    │   └─ Media constraints
    │
    ├─► Instagram Graph API Client
    │   ├─ Upload media
    │   ├─ Publish container
    │   ├─ Send collaborator invites
    │   └─ Post comments
    │
    ├─► Error Handling
    │   ├─ Retry logic
    │   ├─ Error categorization
    │   └─ User notifications
    │
    └─► State Management
        ├─ QUEUE → PUBLISHED
        └─ QUEUE → ERROR

Total Complexity: ~1000+ lines of business logic
```

---

## 13. Conclusion

### Should You Use Direct Database Access?

**✅ YES for:**
- Read-only analytics queries
- Custom reporting dashboards
- Data exports
- Debugging existing posts

**❌ NO for:**
- Creating new posts
- Publishing content
- Production workflows
- User-facing features

### Why the Public API is Better (Despite Limitations)

1. **It Actually Works** - Posts publish to Instagram
2. **Validation Saves You** - Prevents silent failures
3. **Token Management** - No manual OAuth handling
4. **Queue Processing** - Scheduled posts work
5. **Error Visibility** - Clear error messages
6. **Maintenance** - Updates happen automatically
7. **Collaborators** - Invites actually send
8. **Future-Proof** - Schema changes don't break you

### The Missing Features Problem

**Current Gaps:**
- Location tags
- Product tags
- First comment (internal only)
- Media URLs in response

**Your Options:**
1. **Wait** - May be added (no timeline)
2. **Contribute** - Submit PRs (maintainer discretion)
3. **Fork** - Self-host with modifications (maintenance burden)
4. **Workaround** - Use internal API (no public docs, could break)
5. **Accept** - Use what's available (collaborators work!)

### Final Recommendation

**Use the public API** and work around missing features:
- **Location tags:** Skip or use caption text
- **Product tags:** Not available yet
- **First comment:** Make separate comment API call after posting
- **Media URLs:** Store URLs client-side during upload

**Only consider direct DB access** if you're willing to:
- Build and maintain complete worker system (~1000 lines)
- Implement token refresh logic
- Handle all Instagram API edge cases
- Monitor schema changes and update code
- Debug silent failures yourself

The 30 req/hr rate limit is the biggest constraint, but you can batch multiple posts in one request. For most use cases, this is sufficient.

---

## Sources

- [Postiz GitHub Repository](https://github.com/gitroomhq/postiz-app) - 24.8k stars, 66 contributors
- [Postiz Public API Docs](https://docs.postiz.com/public-api/introduction) - Official API documentation
- [Postiz Instagram Settings](https://docs.postiz.com/public-api/providers/instagram) - Provider-specific parameters
- [Issue #1043: Return uploaded media](https://github.com/gitroomhq/postiz-app/issues/1043) - Open feature request
- [Issue #996: Media deletion doesn't remove from R2](https://github.com/gitroomhq/postiz-app/issues/996) - Storage bug
- [Prisma Schema](https://github.com/gitroomhq/postiz-app/blob/main/libraries/nestjs-libraries/src/database/prisma/schema.prisma) - Database structure
- [Instagram DTO](https://github.com/gitroomhq/postiz-app/blob/main/libraries/nestjs-libraries/src/dtos/posts/providers-settings/instagram.dto.ts) - Settings validation
- [Instagram Provider](https://raw.githubusercontent.com/gitroomhq/postiz-app/main/libraries/nestjs-libraries/src/integrations/social/instagram.provider.ts) - Publishing logic
- [Create Post DTO](https://github.com/gitroomhq/postiz-app/blob/main/libraries/nestjs-libraries/src/dtos/posts/create.post.dto.ts) - Request validation
- [API Module](https://raw.githubusercontent.com/gitroomhq/postiz-app/main/apps/backend/src/api/api.module.ts) - Route organization
- [Public Controller](https://raw.githubusercontent.com/gitroomhq/postiz-app/main/apps/backend/src/api/routes/public.controller.ts) - Public endpoints
- [Recent Commits](https://github.com/gitroomhq/postiz-app/commits/main) - Development activity
- [Merged PRs](https://github.com/gitroomhq/postiz-app/pulls?q=is%3Apr+instagram+is%3Amerged) - Instagram features added
- [CONTRIBUTING.md](https://github.com/gitroomhq/postiz-app/blob/main/CONTRIBUTING.md) - Development guidelines
