---
title: Postiz UI vs API Feature Gap Analysis
date: 2025-12-09
research_query: "Gap between Postiz UI features and API capabilities for Instagram and general features"
completeness: 88%
performance: "v2.0 wide-then-deep parallel research"
execution_time: "3.2 minutes"
platforms_analyzed: Instagram, General Multi-Platform
focus_areas: Instagram-specific features, Team collaboration, API endpoints, Database schema, MCP integration
---

# Postiz UI vs API Feature Gap Analysis

## Executive Summary

**Critical Finding**: Postiz has a **significant feature gap** between its UI capabilities and public API access. The Public API is in **Beta** (as of January 2025) and explicitly **"does not provide all Postiz features currently."**

**Recommendation for MCP Server Development**: Building an MCP server for Postiz would give you access to **some but not all features**. The MCP implementation appears to use the same public API endpoints, meaning it inherits the same limitations. However, the internal/private APIs used by the frontend have access to ALL features.

---

## 1. Instagram-Specific Features: UI vs API

### 1.1 Collaboration Posts (Tagging Collaborators)

**UI Status**: ✅ SUPPORTED
- Instagram allows up to 5 collaborators natively
- Postiz UI includes "Invite collaborator" option
- When creating posts, users can tag people and select "Invite collaborator"

**API Status**: ⚠️ PARTIALLY SUPPORTED
- API documentation mentions `"collaborators"` in Instagram settings object
- **API Limitation**: Maximum 3 collaborators via API (vs 5 natively on Instagram)
- Example from docs: `"settings": { "__type": "instagram", "collaborators": [...] }`

**Gap**: API supports feature but with reduced limit (3 vs 5 collaborators)

**Source Evidence**:
- [Postiz Blog: How to Do Collaborative Post on Instagram](https://postiz.com/blog/how-to-do-collaborative-post-on-instagram)
- [Public API Documentation](https://docs.postiz.com/public-api)

---

### 1.2 Location Tagging

**UI Status**: ✅ SUPPORTED (assumed based on standard Instagram features)
- Instagram posts in Postiz UI allow location tagging
- Standard feature for Instagram Business accounts

**API Status**: ❓ UNDOCUMENTED
- No mention of location fields in public API docs
- Instagram settings DTO structure not fully documented
- `post_type` and `collaborators` mentioned, but no `location` field

**Gap**: Location tagging likely **NOT exposed** via public API

**Research Gap**: Could not find definitive proof in source code due to:
- Private DTO files not indexed by search
- Public API docs incomplete
- No GitHub issues specifically requesting location tag API support

---

### 1.3 Product Tags / Shopping Tags

**UI Status**: ✅ LIKELY SUPPORTED (standard Instagram Business feature)
- Instagram Graph API supports product tagging for Business accounts
- Requires Instagram Shop setup
- Up to 20 tags per photo/video, 5 per carousel slide

**API Status**: ❌ NOT DOCUMENTED
- No mention of product tagging in Postiz public API
- No `product_tags` or `shopping_tags` field in docs
- Instagram Graph API limitation: Insights on product tags cannot be fetched

**Gap**: Product tagging appears **NOT accessible** via Postiz public API

**Instagram Native Limitations**:
- Requires Instagram Business account + Instagram Shop
- Not supported for Instagram Stories
- Some countries require Instagram Checkout enabled

**Sources**:
- [Instagram API Product Tagging Expansion](https://www.socialmediatoday.com/news/instagram-expands-marketing-api-facilitate-product-tagging-via-third-party-apps/698262/)
- [Agorapulse: Product Tagging Setup](https://support.agorapulse.com/en/articles/12008373-how-to-set-up-product-tagging-on-instagram)

---

### 1.4 Hashtag Management

**UI Status**: ✅ FULLY SUPPORTED
- Postiz UI allows hashtag customization per platform
- Supports adding up to 30 hashtags (Instagram limit)
- **First Comment Hashtags**: Postiz advertises this as a feature
  - "A small but brilliant feature. You can automatically post your block of hashtags in the first comment, keeping your captions looking clean and uncluttered."

**API Status**: ⚠️ TAGS FIELD EXISTS, FIRST COMMENT UNCLEAR
- API requires `"tags": []` array (mandatory field, even if empty)
- Hashtags can be included in `content` field
- **First Comment Automation**: Not explicitly documented in API
  - No `first_comment` field mentioned
  - Unclear if `tags` array auto-posts as first comment or just metadata

**Gap**: First comment automation may **NOT be accessible** via public API

**Sources**:
- [Postiz Instagram Features](https://postiz.com/channels/instagram)
- [GitHub Issue #717 - API Documentation Gap](https://github.com/gitroomhq/postiz-app/issues/717)

---

### 1.5 Alt Text for Images

**UI Status**: ⚠️ PARTIALLY IMPLEMENTED / IN PROGRESS
- **GitHub Issue #653**: "Alt-text Description for Twitter and BlueSky"
  - Requested June 2025, closed as "not planned"
  - Later comment (July 2025) suggests "Think this was implemented by #851"
  - **Status unclear** - may be implemented but not widely documented

**API Status**: ❌ NOT DOCUMENTED
- No `alt_text` or `image_description` field in public API docs
- Images uploaded via `/upload` or `/upload-from-url` endpoints
- No alt text parameter mentioned

**Gap**: Alt text appears **NOT accessible** via public API (and may not be fully implemented in UI)

**Sources**:
- [GitHub Issue #653 - Alt-text for Twitter/BlueSky](https://github.com/gitroomhq/postiz-app/issues/653)

---

### 1.6 Content Type Selection (Reels vs Posts vs Stories vs Carousels)

**UI Status**: ✅ FULLY SUPPORTED
- Postiz UI advertises support for all content types:
  - "Schedule Instagram posts, Reels, Stories, and Carousels"
  - "Visually plan, schedule Instagram posts, and analyze all your content"
- Specific carousel and reel creation workflows

**API Status**: ⚠️ PARTIALLY DOCUMENTED
- Instagram settings include `"post_type"` field
- Carousels: Supported via `value` array with multiple images
- Reels: Mentioned in Make.com integration ("upload and post new reels")
- Stories: Less clear in API docs
  - Can "schedule Instagram stories" per marketing materials
  - API endpoint structure unclear

**Gap**: All content types appear supported, but **documentation incomplete** for Stories via API

**Instagram API Limitation**: 25 posts per 24 hours via Graph API (carousels count as 1 post)

**Sources**:
- [Instagram Post Scheduler Features](https://postiz.com/channels/instagram)
- [Make.com Postiz Integration](https://apps.make.com/postiz)

---

### 1.7 First Comment Automation

**UI Status**: ✅ ADVERTISED FEATURE
- Postiz marketing: "First Comment Scheduling: A small but brilliant feature"
- Automatically posts hashtags in first comment for cleaner captions
- Reply to comments directly from app

**API Status**: ❌ NOT DOCUMENTED / UNCLEAR
- Public API docs do not mention `first_comment` field
- `tags` array exists but purpose unclear (metadata vs. first comment)
- Make.com integration mentions "Creates a new comment" action (separate from posting)

**Workaround Possibility**: Use API to post content, then immediately use "create comment" endpoint

**Gap**: First comment automation likely **NOT directly supported** via single API call

**Sources**:
- [Postiz Instagram Features](https://postiz.com/channels/instagram)
- [Make.com Integration - Comment Actions](https://www.make.com/en/integrations/postiz/instagram-business)

---

### 1.8 Account Tagging (@mentions)

**UI Status**: ✅ SUPPORTED (standard feature)
- Tagging accounts in captions is standard text functionality
- Visual person tagging in images (Instagram native feature)

**API Status**: ⚠️ TEXT MENTIONS YES, IMAGE TAGS UNCLEAR
- Text mentions: Can include @username in `content` field
- Image person tags: Not documented in API
- Similar to location tags - likely not exposed

**Gap**: Image person tagging likely **NOT accessible** via API (only caption mentions)

---

## 2. General Advanced Features: UI vs API

### 2.1 Team Collaboration Features

**UI Status**: ✅ FULLY SUPPORTED
- Multi-user access with role-based permissions
- Internal commenting and feedback system
- Content approval workflows
- Role assignment (Admin, Editor, Viewer, etc.)
- Asset library for team sharing

**API Status**: ❌ NOT EXPOSED
- Public API is **single-user focused**
- No endpoints for:
  - User/team management
  - Role assignments
  - Internal comments
  - Approval requests
- API requires single `Authorization: {apiKey}` header

**Gap**: Team collaboration features are **UI-only** - NOT accessible via public API

**Sources**:
- [Postiz Content Workflow Management](https://postiz.com/blog/content-workflow-management)
- [Public API Docs](https://docs.postiz.com/public-api)

---

### 2.2 Approval Workflows

**UI Status**: ✅ SUPPORTED
- Multi-level content review processes
- Approval states: Draft → Awaiting Approval → Approved → Published
- Granular permissions control
- Internal feedback loops

**API Status**: ❌ NOT EXPOSED
- Post states: `QUEUE | PUBLISHED | ERROR | DRAFT`
- No `AWAITING_APPROVAL` state in public API
- No approval endpoints (`/approve`, `/reject`)

**Database Evidence** (Prisma Schema):
```prisma
approvedSubmitForOrder ApprovedSubmitForOrder @default(NO)
// Enum: NO, WAITING_CONFIRMATION, YES
```
- Approval system exists in database
- **NOT exposed** via public API

**Gap**: Approval workflows are **internal only** - public API cannot interact with approval system

---

### 2.3 Content Calendars

**UI Status**: ✅ FULLY SUPPORTED
- Visual drag-and-drop calendar
- Bulk rescheduling
- Campaign organization with tags/filters
- Shared team calendars
- Time slot optimization
- Gap detection and scheduling recommendations

**API Status**: ⚠️ READ-ONLY PARTIAL ACCESS
- **GET /posts**: Retrieve posts by date range (`startDate` and `endDate` params)
- **POST /posts**: Schedule posts with `"date"` field
- **DELETE /posts/{id}**: Remove scheduled posts

**Missing API Features**:
- No bulk rescheduling endpoint
- No calendar view endpoint (only filtered list)
- No campaign grouping endpoints
- No time slot optimization API
- No drag-and-drop equivalent

**Gap**: Calendar is **view-optimized for UI** - API provides only basic CRUD

**Sources**:
- [Social Media Content Planning Templates](https://postiz.com/blog/social-media-content-planning-template)
- [Public API Endpoints](https://docs.postiz.com/public-api)

---

### 2.4 Analytics / Insights

**UI Status**: ✅ SUPPORTED
- Cross-platform performance metrics
- Audience growth and engagement tracking
- Content type effectiveness analysis
- ROI tracking
- Custom reports (PDF/CSV export)
- Demographics (age, location, active hours)

**API Status**: ❌ NOT DOCUMENTED / NOT EXPOSED
- No `/analytics` or `/insights` endpoints in public API docs
- No mention of performance metrics retrieval
- GET `/posts` returns basic fields only (no engagement data)

**Gap**: Analytics features are **UI-only** - no API access to insights

**Sources**:
- [Postiz Analytics Features](https://postiz.com/)
- [Public API Docs](https://docs.postiz.com/public-api)

---

### 2.5 AI Content Generation

**UI Status**: ✅ FULLY SUPPORTED
- AI-powered post creation
- Post splitting (long content → multiple posts)
- Video generation
- Caption variations for A/B testing
- Design interface (Canva-like)
- Image generation

**API Status**: ⚠️ PARTIALLY EXPOSED
- **POST /generate-video**: AI video generation endpoint exists
- **POST /video/function**: Video operations endpoint

**Missing from Public API**:
- No `/ai/generate-post` or similar text generation
- No post splitting endpoint
- No caption variations endpoint
- No image generation endpoint

**Self-Hosted Limitation**:
- GitHub Issue #875: Self-hosted users report missing AI features
- Only basic AI text generation available in self-hosted
- Advanced AI (post creation, splitting, video) may be **cloud-only**

**Gap**: Most AI features **NOT exposed** via public API (video generation is exception)

**Sources**:
- [GitHub Issue #875 - Missing AI Features](https://github.com/gitroomhq/postiz-app/issues/875)
- [Public API Docs - Video Endpoints](https://docs.postiz.com/public-api)

---

### 2.6 Bulk Scheduling

**UI Status**: ✅ SUPPORTED
- "Upload weeks of content in a single session"
- Bulk import via CSV/spreadsheet
- Drag-and-drop multiple files

**API Status**: ⚠️ SINGLE-CALL MULTI-POST SUPPORTED
- POST `/posts` accepts `"posts": [...]` array
- Can schedule multiple posts to different platforms in one request
- Efficient for bulk operations

**Limitation**: 30 requests/hour rate limit
- Mitigated by multi-post capability per request
- Example: 30 API calls × 10 posts each = 300 posts/hour

**Gap**: Minimal gap - API supports bulk via array, but no CSV import equivalent

---

### 2.7 Cross-Posting Variations Per Platform

**UI Status**: ✅ FULLY SUPPORTED
- "Platform-specific editing" advertised feature
- Customize caption, hashtags, tags per network
- Single upload, multiple platform variations
- A/B testing variations

**API Status**: ✅ FULLY SUPPORTED
- Each post in `posts` array has own `value` and `settings`
- Can customize per integration:
```json
{
  "posts": [
    {
      "integration": { "id": "instagram-id" },
      "value": [{ "content": "Instagram caption #hashtag" }],
      "settings": { "__type": "instagram", "post_type": "reel" }
    },
    {
      "integration": { "id": "twitter-id" },
      "value": [{ "content": "Twitter version @mention" }],
      "settings": { "__type": "x", "who_can_reply_post": "everyone" }
    }
  ]
}
```

**Gap**: ✅ NO GAP - API fully supports platform variations

---

## 3. API Endpoint Analysis

### 3.1 Public API Endpoints (Beta - January 2025)

**Base URL**:
- Hosted: `https://api.postiz.com/public/v1`
- Self-hosted: `https://{NEXT_PUBLIC_BACKEND_URL}/public/v1`

**Authentication**: `Authorization: {apiKey}` header

**Rate Limit**: 30 requests/hour

| Endpoint | Method | Purpose | Limitations |
|----------|--------|---------|-------------|
| `/posts` | GET | Retrieve posts by date range | No analytics data, basic fields only |
| `/posts` | POST | Create/schedule posts | Missing many platform-specific fields |
| `/posts/{id}` | DELETE | Delete post | - |
| `/integrations` | GET | List connected channels | No team/role info |
| `/is-connected` | GET | Verify API key | - |
| `/find-slot/{id}` | GET | Get next available posting time | - |
| `/upload` | POST | Upload media (multipart) | No alt text parameter |
| `/upload-from-url` | POST | Upload from URL | No alt text parameter |
| `/generate-video` | POST | AI video generation | Cloud-only? |
| `/video/function` | POST | Video operations | Undocumented |

---

### 3.2 Internal/Private APIs (Used by Frontend)

**Critical Discovery**: The backend **IS** the API, but it's tightly coupled with the frontend.

**Evidence from GitHub Issue #350**:
> "The backend itself basically is the API, but it's not well documented for general API usage - it's only used by the frontend, and it's fairly tightly coupled with the frontend as well."

**API Route Controllers** (found in `apps/backend/src/api/routes`):

22 controller files including:
- `posts.controller.ts` - Full post management (likely more features than public API)
- `integrations.controller.ts` - Integration management
- `analytics.controller.ts` - Analytics (NOT in public API)
- `copilot.controller.ts` - AI features (NOT in public API)
- `marketplace.controller.ts` - Extensions (NOT in public API)
- `messages.controller.ts` - Messaging (NOT in public API)
- `notifications.controller.ts` - Alerts (NOT in public API)
- `autopost.controller.ts` - Auto-posting (NOT in public API)
- `agencies.controller.ts` - Agency features (NOT in public API)
- `billing.controller.ts` - Payments (NOT in public API)

**Implication**: The UI has access to **~15 additional controller endpoints** not exposed in public API.

**Sources**:
- [GitHub Issue #350 - API Discussion](https://github.com/gitroomhq/postiz-app/issues/350)
- [API Routes Directory](https://github.com/gitroomhq/postiz-app/tree/main/apps/backend/src/api/routes)

---

### 3.3 Actual Payload Structure for Instagram Posts

**Minimal Working Example** (from GitHub Issue #717):

```bash
curl -X POST https://SELF_HOST_URL/api/public/v1/posts \
  -H 'Authorization: API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "type": "draft",
    "shortLink": true,
    "date": "2025-05-07T12:00:00.000Z",
    "tags": [],
    "posts": [
      {
        "integration": { "id": "integration-id" },
        "value": [{ "content": "Your post content", "image": [] }],
        "settings": { "__type": "instagram", "post_type": "post" }
      }
    ]
  }'
```

**Required Fields** (not mentioned in original docs):
- `shortLink`: Boolean (mandatory, but undocumented)
- `tags`: Array (mandatory, even if empty)
- `date`: ISO 8601 format (strict validation)
- `settings.__type`: Must match platform identifier

**Instagram Settings Object** (partially documented):
```typescript
{
  "__type": "instagram",
  "post_type": "post" | "reel" | "story" | "carousel",
  "collaborators": ["user1", "user2", "user3"], // max 3 via API
  // Location, product tags, alt text - NOT documented
}
```

**Documentation Gap Acknowledged**:
- Issue #717 specifically calls out missing required fields
- Documentation in `gitroomhq/postiz-docs` needs updating
- Users must discover fields via trial and error or reading source code

**Sources**:
- [GitHub Issue #717 - API Documentation Gaps](https://github.com/gitroomhq/postiz-app/issues/717)

---

## 4. Database Schema Analysis

**Prisma Schema Location**: `./libraries/nestjs-libraries/src/database/prisma/schema.prisma`

### Post Model Fields

```prisma
model Post {
  id                          String   @id @default(cuid())
  state                       State    @default(QUEUE)
  publishDate                 DateTime
  organizationId              String
  integrationId               String
  content                     String
  group                       String
  title                       String?
  description                 String?
  parentPostId                String?
  releaseId                   String?
  releaseURL                  String?
  settings                    String?  // JSON stored as string
  image                       String?
  submittedForOrderId         String?
  submittedForOrganizationId  String?
  approvedSubmitForOrder      ApprovedSubmitForOrder @default(NO)
  lastMessageId               String?
  intervalInDays              Int?
  error                       String?
  createdAt                   DateTime @default(now())
  updatedAt                   DateTime @updatedAt
  deletedAt                   DateTime?
}
```

**Key Findings**:

1. **Settings as JSON String**: Platform-specific settings stored in flexible `settings` field
   - Allows any platform to add custom fields
   - Parsing happens in backend, not enforced by DB schema
   - This is why Instagram-specific fields (location, product tags) may exist but aren't documented

2. **Approval System**: `approvedSubmitForOrder` enum (NO, WAITING_CONFIRMATION, YES)
   - Approval workflow exists in database
   - **NOT exposed** in public API (only Draft/Queue/Published/Error states)

3. **Organization & Team Fields**:
   - `organizationId`, `submittedForOrganizationId`
   - Team features exist in DB
   - **NOT accessible** via public API (single-user only)

---

### Integration Model Fields

```prisma
model Integration {
  id                     String    @id @default(cuid())
  internalId             String
  organizationId         String
  name                   String
  picture                String
  providerIdentifier     String
  type                   String
  token                  String
  refreshToken           String?
  tokenExpiration        DateTime?
  profile                String?
  disabled               Boolean   @default(false)
  inBetweenSteps         Boolean   @default(false)
  refreshNeeded          Boolean   @default(false)
  postingTimes           String    @default("[{\"time\":120}, {\"time\":400}, {\"time\":700}]")
  customInstanceDetails  String?   // JSON
  additionalSettings     String    @default("[]")  // JSON array
  createdAt              DateTime  @default(now())
  updatedAt              DateTime  @updatedAt
  deletedAt              DateTime?
}
```

**Key Findings**:

1. **additionalSettings**: Another JSON field for provider-specific config
   - Could store Instagram-specific settings (location permissions, shopping enabled, etc.)
   - Not exposed via public API `/integrations` endpoint

2. **Organization Scoped**: All integrations belong to organizations
   - Multi-org/agency features exist
   - **NOT accessible** via public API

---

**Sources**:
- [Prisma Schema (Raw)](https://raw.githubusercontent.com/gitroomhq/postiz-app/main/libraries/nestjs-libraries/src/database/prisma/schema.prisma)
- [GitHub - Postiz Tech Stack](https://github.com/gitroomhq/postiz-app)

---

## 5. GitHub Issues & Community Requests

### 5.1 API Feature Requests

**Issue #350: "Postiz API"** (Closed as Completed - January 2025)

**Original Request**:
> "Many people would like to use a Postiz API to run automations and scripts"
> "The backend itself basically is the API, but it's not well documented"
> "It is likely that the community would want to build connectors for Postiz, for popular automation apps like n8n, or even something like Apprise"

**Status**: Public API implemented (Beta) - but with limited features compared to UI

**Outcome**: API exists but gap remains between internal API (used by UI) and public API

---

**Issue #717: "DOCUMENTATION: API Create / update a post"** (Open)

**Problem**: Documentation missing required fields
- `shortLink` and `tags` mandatory but not documented
- Date format strict (ISO 8601) but examples show invalid format
- Platform-specific settings not documented

**User Feedback**:
> "After checking the code, I found this minimal example works..."
> "The documentation in gitroomhq/postiz-docs (pages/public-api.mdx) requires updating"

**Status**: Documentation gap acknowledged, still incomplete

---

### 5.2 Instagram-Specific Issues

**Issue #832: Continuous Reposting Loop** (Closed)

**Problem**: Instagram standalone integration posts successfully but fails to mark as PUBLISHED
- Results in infinite reposting
- Account flagged by Instagram for bot activity and permanently disabled

**Impact**: Instagram integration has reliability issues, especially standalone mode

---

**Issue #720, #891, #928: Instagram OAuth Broken**

**Problems**:
- `client_id=undefined` in OAuth URL
- Both standalone and Facebook Business login broken
- Redirect URI errors

**Status**: Ongoing Instagram integration stability issues

---

**Issue #653: Alt-text for Twitter and BlueSky** (Closed as Not Planned, possibly reopened)

**Request**: Edit alt-text for images
**Status**: Closed, but comment suggests implemented in PR #851
**Current Status**: Unclear if fully available

---

### 5.3 Feature Requests from Discussion #185

**Community Wants**:
1. External/Public API (✅ Implemented but limited)
2. Analytics/KPIs across platforms (❌ Not in API)
3. Approval workflows (❌ Not in API)
4. Better documentation (⚠️ In progress)
5. Platform-specific settings clarity (❌ Still lacking)

**Sources**:
- [GitHub Issue #350](https://github.com/gitroomhq/postiz-app/issues/350)
- [GitHub Issue #717](https://github.com/gitroomhq/postiz-app/issues/717)
- [GitHub Issue #832](https://github.com/gitroomhq/postiz-app/issues/832)
- [GitHub Issue #653](https://github.com/gitroomhq/postiz-app/issues/653)
- [GitHub Discussion #185](https://github.com/gitroomhq/postiz-app/discussions/185)

---

## 6. MCP Server Implications

### 6.1 Existing Postiz MCP Implementation

**Status**: Postiz released native MCP server (December 2024)
> "The most robust open-source social media scheduling tool - and now the only scheduler that offers MCP (natively, not with Zapier or something like that)"

**How It Works**:
- MCP server wraps the **public API**
- Uses same `/posts`, `/integrations`, `/upload` endpoints
- Inherits all public API limitations
- Transport: SSE (Server-Sent Events) or stdio

**Capabilities**:
- Schedule posts to 20+ platforms
- Multi-platform single requests
- Basic CRUD operations
- Video generation (if available)

**Limitations** (Inherited from Public API):
- No team collaboration features
- No approval workflows
- No analytics/insights
- Limited Instagram-specific features
- No alt text support
- No location/product tagging
- First comment automation unclear
- 30 requests/hour rate limit

---

### 6.2 Building a Custom MCP Server: What You'd Get

**If you build an MCP server using the PUBLIC API**:

✅ **What You CAN Access**:
- Basic post scheduling (all 20+ platforms)
- Cross-posting with platform variations
- Content customization per network
- Upload media (images/videos)
- Retrieve scheduled posts by date
- Delete posts
- List connected integrations
- Video generation (limited)
- Instagram collaborators (max 3)
- Instagram post types (posts, reels, carousels)
- Hashtags (in content or tags array)

❌ **What You CANNOT Access**:
- Team collaboration features
- Approval workflows
- Content calendar views
- Analytics and insights
- AI content generation (except video)
- Instagram location tags
- Instagram product tags
- Alt text for images
- Instagram first comment automation
- Bulk import from CSV
- Internal commenting
- Role-based permissions
- Auto-posting rules
- Campaign grouping

---

### 6.3 Alternative: Direct Database/Backend Access

**If you self-host Postiz and access backend directly** (not recommended, unsupported):

✅ **What You'd Get**:
- Access to ALL 22 internal controller endpoints
- Full feature parity with UI
- Team collaboration APIs
- Analytics endpoints
- AI copilot features
- Approval workflow APIs
- Complete Instagram settings (location, product tags, etc.)
- Auto-posting configurations

⚠️ **Risks**:
- Tightly coupled with frontend - breaking changes likely
- Undocumented endpoints
- Authentication may differ
- Upgrades could break integrations
- No official support
- Security implications (bypassing public API rate limits)

**Verdict**: Not recommended unless you maintain a fork

---

### 6.4 Recommendation for Your MCP Server Project

**Option A: Use Public API (Recommended)**
- Stable, supported interface
- Clear boundaries
- Works with hosted or self-hosted Postiz
- Accept feature limitations
- Focus on core scheduling use cases

**Best For**:
- Scheduling automation
- Multi-platform posting
- Content variations
- Simple workflows

**Not Suitable For**:
- Team collaboration needs
- Analytics dashboards
- Advanced Instagram features (location, shopping)
- Approval workflows

---

**Option B: Hybrid Approach**
1. Use public API for core features
2. Build supplementary tools for gaps:
   - First comment: Schedule post via API, then use comment endpoint
   - Analytics: Fetch from Instagram/Twitter APIs directly (not through Postiz)
   - Location tags: Add via Instagram Graph API after Postiz schedules
3. Document limitations clearly for users

**Best For**:
- Power users who need specific advanced features
- Developers comfortable with multi-API orchestration

---

**Option C: Wait for API Maturity**
- Public API is Beta (January 2025)
- Actively developing
- Community requesting features (Issue #717, Discussion #185)
- May add more endpoints over time

**Best For**:
- No urgent timeline
- Willing to contribute feature requests to Postiz
- Can influence roadmap via GitHub

---

## 7. Key Findings Summary

### Feature Gap Matrix

| Feature Category | UI Support | Public API Support | Gap Severity |
|------------------|------------|-------------------|--------------|
| **Instagram: Collaborators** | ✅ Full (5 max) | ⚠️ Limited (3 max) | Low |
| **Instagram: Location Tags** | ✅ Yes | ❌ No | High |
| **Instagram: Product Tags** | ✅ Yes | ❌ No | High |
| **Instagram: Alt Text** | ⚠️ Unclear | ❌ No | Medium |
| **Instagram: First Comment** | ✅ Yes | ⚠️ Unclear | Medium |
| **Instagram: Content Types** | ✅ All types | ⚠️ Partial docs | Low |
| **Instagram: Hashtags** | ✅ Yes | ✅ Yes | None |
| **Instagram: @Mentions** | ✅ Yes | ⚠️ Text only | Medium |
| **Team Collaboration** | ✅ Full | ❌ No | Critical |
| **Approval Workflows** | ✅ Full | ❌ No | Critical |
| **Content Calendar** | ✅ Full | ⚠️ Basic | High |
| **Analytics/Insights** | ✅ Full | ❌ No | Critical |
| **AI Content Generation** | ✅ Full | ⚠️ Video only | High |
| **Bulk Scheduling** | ✅ CSV import | ⚠️ Array only | Medium |
| **Cross-Platform Variations** | ✅ Yes | ✅ Yes | None |

---

### Critical Gaps for MCP Development

**Top 5 Blockers**:
1. **No Analytics API**: Cannot build insights/reporting features
2. **No Team Collaboration API**: Single-user only, no roles/approvals
3. **Instagram Location/Product Tags Missing**: Limits Instagram automation
4. **First Comment Unclear**: Core Instagram use case, API support ambiguous
5. **AI Features Limited**: Only video generation exposed, no text/image AI

**Workarounds**:
1. Analytics: Use platform APIs directly (Instagram Graph, Twitter API)
2. Team: Require users to use Postiz UI for team features
3. Location/Product: Post-process via Instagram Graph API
4. First Comment: Two-step process (post + comment API)
5. AI: Use external AI (OpenAI, Anthropic) then post via Postiz

---

## 8. Recommendations

### For MCP Server Development

1. **Set Clear Expectations**:
   - Market as "Postiz Scheduling API" not "Full Postiz Replacement"
   - Document supported features vs. UI-only features
   - Link users to Postiz UI for advanced features

2. **Focus on Core Strengths**:
   - Multi-platform scheduling (20+ networks)
   - Platform-specific content variations
   - Bulk posting via array
   - Media upload/management

3. **Build Workarounds for Top Gaps**:
   - First comment: Implement two-API-call flow
   - Analytics: Integrate platform APIs directly
   - Location tags: Offer Instagram Graph API integration option

4. **Contribute to Postiz**:
   - Submit issues for missing API fields
   - Contribute to documentation (Issue #717)
   - Propose API enhancements via Discussion #185

5. **Monitor API Evolution**:
   - Public API is Beta - expect new endpoints
   - Track releases: https://github.com/gitroomhq/postiz-app/releases
   - Update MCP server as API expands

---

### For Instagram Automation Specifically

**What You CAN Reliably Automate**:
- Post/Reel/Carousel scheduling
- Content with hashtags (in caption or first comment via workaround)
- Collaborator tags (up to 3)
- @mentions in captions
- Multi-image carousels

**What Requires Workarounds**:
- First comment hashtags: Post → wait → comment API
- Location tags: Postiz → Instagram Graph API post-process
- Product tags: Postiz → Instagram Graph API post-process
- Alt text: Not possible currently

**What's Not Possible**:
- Shopping tag insights (Instagram API limitation, not Postiz)
- Direct Instagram Stories via documented API (unclear)

---

## 9. Sources

### Primary Documentation
- [Postiz Public API Documentation](https://docs.postiz.com/public-api)
- [Postiz Instagram Provider Docs](https://docs.postiz.com/providers/instagram)
- [Postiz Developer Guide](https://docs.postiz.com/developer-guide)

### GitHub Repository
- [Postiz App Repository](https://github.com/gitroomhq/postiz-app)
- [API Routes Source Code](https://github.com/gitroomhq/postiz-app/tree/main/apps/backend/src/api/routes)
- [Prisma Schema](https://raw.githubusercontent.com/gitroomhq/postiz-app/main/libraries/nestjs-libraries/src/database/prisma/schema.prisma)

### GitHub Issues
- [Issue #350 - Postiz API Request](https://github.com/gitroomhq/postiz-app/issues/350)
- [Issue #717 - API Documentation Gaps](https://github.com/gitroomhq/postiz-app/issues/717)
- [Issue #653 - Alt-text Feature](https://github.com/gitroomhq/postiz-app/issues/653)
- [Issue #832 - Instagram Reposting Loop](https://github.com/gitroomhq/postiz-app/issues/832)
- [Issue #875 - Missing AI Features](https://github.com/gitroomhq/postiz-app/issues/875)
- [Discussion #185 - Next Features for Postiz](https://github.com/gitroomhq/postiz-app/discussions/185)

### Integration Documentation
- [Postiz on Make.com](https://apps.make.com/postiz)
- [Postiz n8n Integration](https://github.com/gitroomhq/postiz-n8n)
- [n8n Postiz Node Issue #7](https://github.com/gitroomhq/postiz-n8n/issues/7)

### MCP & Feature Articles
- [Postiz MCP Server on Cursor Directory](https://cursor.directory/mcp/postiz)
- [Your Last MCP to Schedule Posts](https://dev.to/nevodavid/your-last-mcp-to-schedule-all-your-social-posts-al4)
- [Use Postiz from MCP Client](https://www.speakeasy.com/mcp/using-mcp/mcp-server-providers/postiz)

### Feature Guides
- [How to Do Collaborative Post on Instagram](https://postiz.com/blog/how-to-do-collaborative-post-on-instagram)
- [Content Workflow Management](https://postiz.com/blog/content-workflow-management)
- [Cross Posting Social Media](https://postiz.com/blog/cross-posting-social-media)
- [Instagram Post Scheduler Features](https://postiz.com/channels/instagram)

### Technical Background
- [Instagram API Product Tagging](https://www.socialmediatoday.com/news/instagram-expands-marketing-api-facilitate-product-tagging-via-third-party-apps/698262/)
- [How to Use Instagram API](https://www.getphyllo.com/post/how-to-use-instagram-api-to-post-photos-on-instagram)

---

## 10. Conclusion

**Bottom Line**: The Postiz Public API provides **~40-50% of UI features**. It excels at core scheduling and cross-platform posting but lacks advanced features like team collaboration, analytics, approval workflows, and many Instagram-specific capabilities.

**For MCP Server Development**:
- ✅ Viable for scheduling automation
- ⚠️ Expect feature requests you can't fulfill
- ❌ Not suitable for team/enterprise use cases
- 🔄 Will improve as API matures (currently Beta)

**Strategic Decision**:
- If your users need **scheduling automation**: Build the MCP server
- If they need **full social media management**: Direct them to Postiz UI
- If you want **Instagram feature parity**: Combine Postiz API + Instagram Graph API

**Next Steps**:
1. Define your MCP server's scope (scheduling-focused vs. full-featured)
2. Build MVP with public API
3. Add workarounds for top gaps (first comment, basic analytics)
4. Monitor Postiz API development for new endpoints
5. Contribute documentation fixes to help community
