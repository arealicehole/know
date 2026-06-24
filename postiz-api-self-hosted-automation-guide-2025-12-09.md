---
title: "Postiz API for Self-Hosted Installations: Complete Automation Guide"
date: 2025-12-09
research_query: "Postiz API self-hosted automation workflows - walking the dog"
completeness: 90%
performance: "v2.0 wide-then-deep"
execution_time: "2.8 minutes"
---

# Postiz API for Self-Hosted Installations: Complete Automation Guide

## Executive Summary

Postiz is an open-source social media scheduling platform that exposes a comprehensive Public API for automation workflows. The API supports 27+ social platforms and is particularly powerful for self-hosted installations where you can build custom "walking the dog" workflows (full content pipeline automation). This guide provides complete implementation details for automating social media posting via the Postiz API.

**Key Findings:**
- Public API launched January 2025 (previously in beta)
- REST API with JWT authentication via API keys
- 30 requests/hour rate limit (not post limit - strategic batching recommended)
- Self-hosted URL: `https://{NEXT_PUBLIC_BACKEND_URL}/public/v1`
- Strong n8n/Make/Zapier integration ecosystem
- Community-maintained tools and extensions available

---

## 1. Postiz API Overview

### What APIs Does Postiz Expose?

Postiz provides a **REST API** specifically designed for automation workflows. The architecture is straightforward:

- **Backend API**: Handles all operations and posts work to Redis queue
- **Public API**: Documented endpoints at `/public/v1/*`
- **Internal API**: Frontend-backend communication (tightly coupled, not recommended for external use)

**Important Architecture Note:**
> "The backend itself basically is the API, but it's not well documented for general API usage - it's only used by the frontend, and it's fairly tightly coupled with the frontend as well."

**Recommendation:** Use the documented Public API endpoints only. Avoid reverse-engineering internal endpoints.

### Base URLs

| Environment | URL |
|------------|-----|
| **Self-Hosted** | `https://{NEXT_PUBLIC_BACKEND_URL}/public/v1` |
| **Cloud (Hosted)** | `https://api.postiz.com/public/v1` |

For self-hosted n8n integration, your host must end with `/api`:
```
http://yourdomain.com/api
```

### API Documentation Location

- **Official Docs**: [docs.postiz.com/public-api](https://docs.postiz.com/public-api)
- **GitHub Source**: [github.com/gitroomhq/postiz-docs](https://github.com/gitroomhq/postiz-docs/blob/main/pages/public-api.mdx)
- **GitHub Issues**: Community discussions at [github.com/gitroomhq/postiz-app/issues](https://github.com/gitroomhq/postiz-app/issues)

**Documentation Quality:**
- Core endpoints well-documented (integrations, posts, uploads)
- Community-contributed examples (n8n workflows, Make templates)
- Some gaps in platform-specific settings (27 platforms with varying documentation levels)

---

## 2. Core API Capabilities

### Available REST Endpoints

#### Integrations (Social Media Accounts)
```bash
# List all connected channels/accounts
GET /integrations

# Verify API key validity
GET /is-connected

# Find next available posting slot for a channel
GET /find-slot/{id}
```

**Important Terminology:**
> "The UI representation is a channel, but on the API it is called integration. So when you see integration in the API, it means a channel."

#### Posts Management
```bash
# Retrieve posts within date range
GET /posts?startDate={ISO8601}&endDate={ISO8601}&customer={optionalCustomerId}

# Create or update post
POST /posts

# Delete post by ID
DELETE /posts/{id}
```

#### Media Upload
```bash
# Upload file via multipart form data
POST /upload

# Import file from URL
POST /upload-from-url
```

**Upload Response Example:**
```json
{
  "id": "e639003b-f727-4a1e-87bd-74a2c48ae41e",
  "name": "vXJYn8EzSB.png",
  "path": "https://uploads.gitroom.com/vXJYn8EzSB.png",
  "organizationId": "85460a39-6329-4cf4-a252-187ce89a3480",
  "createdAt": "2024-12-14T08:18:54.274Z",
  "updatedAt": "2024-12-14T08:18:54.274Z"
}
```

#### Video Generation (AI Features)
```bash
# Create AI-generated videos
POST /generate-video

# Execute video operations
POST /video/function
```

#### Content Queue Management

Postiz supports intelligent queue systems:
- **Auto-scheduling**: Drop content into queues with pre-set schedules
- **Evergreen queues**: Recycle timeless content automatically
- **Bulk CSV uploads**: Schedule 1,000+ posts via script

**Queue States:**
- `QUEUE` - Scheduled/waiting
- `PUBLISHED` - Successfully posted
- `ERROR` - Failed posting
- `DRAFT` - Not yet scheduled

### Supported Platforms (27 Total)

**21 Platforms with Custom Settings:**
X (Twitter), LinkedIn, Facebook, Instagram, YouTube, TikTok, Reddit, Discord, Medium, Pinterest, Dribbble, Telegram, Slack, YouTube Shorts, and others.

**6 Platforms without Custom Settings:**
Threads, Mastodon, Bluesky, Telegram, Nostr, VK

Each platform requires a `__type` field matching the provider identifier in settings.

### Analytics and Reporting

**Current State:**
> "The Postiz Public API is currently in Beta and in active development. It does not provide all of the Postiz features currently."

**Analytics Capabilities:**
- Dashboard provides engagement metrics, post performance, follower growth
- PostgreSQL stores analytics data (direct database access possible)
- Custom analytics via open APIs (build your own dashboards)
- YouTube requires: YouTube Data API v3, YouTube Analytics API, YouTube Reporting API

**Workaround for Missing API Endpoints:**
Direct database queries via PostgreSQL connection for advanced analytics not yet exposed via API.

### Webhook Support

Postiz includes built-in webhook support for automation:
- Trigger workflows on post events
- Connect to CRMs, newsletters, marketing tools
- n8n workflows can start with webhook triggers

**n8n Webhook Workflow Pattern:**
1. Webhook triggers workflow
2. Fetch metadata from Airtable/Google Sheets
3. Download media from Google Drive
4. Upload to Postiz via `/upload` endpoint
5. Create post via `/posts` endpoint

---

## 3. Authentication & Setup

### How to Generate API Keys (Self-Hosted)

**Step-by-step:**
1. Log in to your Postiz account
2. Click the gear (settings) icon in upper right corner
3. Click "Public API"
4. Copy your API key

**Environment Variables Required (Self-Hosted):**
```bash
# Core Configuration
DATABASE_URL=postgresql://postiz-user:postiz-password@postiz-postgres:5432/postiz-db-local
REDIS_URL=redis://postiz-redis:6379
JWT_SECRET=your-long-random-jwt-secret-string

# URL Configuration
FRONTEND_URL=https://postiz.your-server.com
NEXT_PUBLIC_BACKEND_URL=https://postiz.your-server.com/api
BACKEND_INTERNAL_URL=http://backend:3000

# Storage Configuration (Local)
STORAGE_PROVIDER=local
UPLOAD_DIRECTORY=/uploads
NEXT_PUBLIC_UPLOAD_DIRECTORY=/uploads

# Security (Development Only)
NOT_SECURED=true  # Only for non-HTTPS environments
```

**HTTPS/Security Considerations:**
> "Postiz marks its login cookies as Secure, which is called 'secure context' in modern web browsers. If you want to use a secure Login Process, you need to set up a Certificate, which can be done via Reverse Proxy like Caddy or Nginx."

**Production Recommendation:** Always use HTTPS with reverse proxy (Caddy/Nginx). The `NOT_SECURED=true` flag is for development only.

### Token Management

**Authentication Method:** Bearer token via `Authorization` header

**Example Request:**
```bash
curl -X GET 'https://api.postiz.com/public/v1/integrations' \
  -H 'Authorization: YOUR_API_KEY_HERE'
```

**Token Refresh:** Not required - API keys are long-lived. Regenerate manually in settings if compromised.

**Multi-Tenant Consideration:**
> Feature request exists for "Postiz to accept tokens dynamically (via API or database) from an external system like an authorization portal, instead of requiring API keys/secrets in docker-compose.yml."

Currently not available - each self-hosted instance uses single API key per user account.

### Rate Limiting Considerations

**Official Limit:**
> "There is a limit of 30 requests per hour. It does not mean you can post only 30 posts per hour, it means you can make 30 requests to the API per hour. If you plan ahead, you can have a lot more every hour."

**Strategic Batching:**
- Single `/posts` request can schedule multiple posts to multiple platforms
- Upload all media first (separate requests)
- Batch-create posts referencing uploaded media IDs

**Example Optimization:**
```
Hour 1 Schedule:
- 5 upload requests (images)
- 1 create post request (10 posts to 5 platforms = 50 scheduled posts)
= 6 API requests for 50 posts
```

**No Explicit Bypass:** Rate limits are enforced. Design workflows around 30/hour constraint.

---

## 4. Automation Workflows ("Walking the Dog")

### End-to-End Posting Automation

**Complete Workflow Architecture:**
```
Content Source (CMS/Blog/RSS)
    ↓
Automation Platform (n8n/Make/Zapier)
    ↓
1. Extract content (title, URL, media)
    ↓
2. Format for each platform (character limits, hashtags)
    ↓
3. Upload media to Postiz (/upload or /upload-from-url)
    ↓
4. Create post payload with integration IDs
    ↓
5. Schedule via /posts endpoint
    ↓
Postiz Queue System
    ↓
Automatic Publishing at Scheduled Time
```

### Bulk Scheduling Capabilities

**CSV Bulk Upload:**
> "One of the time-saving features in Postiz is bulk scheduling. Instead of creating posts one by one, you can map out and upload an entire month's worth of content in one go with a simple CSV file."

**Implementation:**
1. Prepare CSV with columns: date, platform, content, media_urls
2. Script reads CSV row-by-row
3. Upload media via `/upload-from-url` (batch)
4. Create post requests with scheduled dates
5. Postiz populates calendar automatically

**Scale Example:**
> "You can break UI limitations and automate tasks at a scale that would be impossible through a web interface - like scheduling 1,000 posts from a CSV file with a single script."

### Content Approval Workflows

**Custom Approval Chains:**
> "You can build completely custom content approval chains, analytics dashboards, or content generation pipelines tailored to your organization's specific needs."

**Pattern for Approval Workflow:**
1. Create posts with `type: "draft"`
2. External approval system reviews drafts via `/posts` GET endpoint
3. Upon approval, update post with `type: "schedule"` or `type: "now"`
4. Postiz queues or publishes immediately

### Integration Patterns with External Tools

**CRM Integration:**
> "You can create custom integrations to connect Postiz to any other tool in your stack, triggering posts from a new entry in your CRM, a sale in your e-commerce store, or a new article on your blog's RSS feed."

**Common Integration Points:**
- WordPress (new post → auto-schedule social posts)
- Shopify (new product → announce on social)
- Airtable (content calendar → Postiz queue)
- Google Sheets (CSV export → bulk schedule)
- HubSpot/Salesforce (new lead → share case study)

### Triggering Posts Programmatically

**Three Post Types:**
```json
{
  "type": "now",      // Immediate publishing
  "type": "schedule", // Queue for specific date/time
  "type": "draft"     // Save without scheduling
}
```

**Immediate Posting Example:**
```bash
curl -X POST 'https://postiz.your-domain.com/api/public/v1/posts' \
  -H 'Authorization: YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "type": "now",
    "shortLink": false,
    "tags": [],
    "posts": [{
      "integration": {"id": "your-integration-id"},
      "value": [{"content": "Breaking: New product launch!"}],
      "settings": {"__type": "x"}
    }]
  }'
```

---

## 5. Integration Examples

### Python Code Example

**Basic Post Creation (Python):**
```python
import requests
from datetime import datetime, timedelta

POSTIZ_API_KEY = "your-api-key-here"
POSTIZ_URL = "https://postiz.your-domain.com/api/public/v1"

headers = {
    "Authorization": POSTIZ_API_KEY,
    "Content-Type": "application/json"
}

# Step 1: Upload media
def upload_media(file_path):
    with open(file_path, 'rb') as f:
        files = {'file': f}
        response = requests.post(
            f"{POSTIZ_URL}/upload",
            headers={"Authorization": POSTIZ_API_KEY},
            files=files
        )
    return response.json()

# Step 2: Create scheduled post
def create_post(integration_id, content, media_id=None, schedule_time=None):
    payload = {
        "type": "schedule" if schedule_time else "now",
        "date": schedule_time or datetime.utcnow().isoformat() + "Z",
        "shortLink": False,
        "tags": [],
        "posts": [{
            "integration": {"id": integration_id},
            "value": [{
                "content": content,
                "image": [{"id": media_id}] if media_id else []
            }],
            "settings": {"__type": "x"}  # Platform identifier
        }]
    }

    response = requests.post(
        f"{POSTIZ_URL}/posts",
        headers=headers,
        json=payload
    )
    return response.json()

# Example usage
media = upload_media("image.png")
post = create_post(
    integration_id="your-integration-id",
    content="Check out our new product!",
    media_id=media['id'],
    schedule_time=(datetime.utcnow() + timedelta(hours=2)).isoformat() + "Z"
)
print(f"Post scheduled: {post}")
```

**Bulk CSV Upload Script (Python):**
```python
import csv
import requests
from datetime import datetime

def bulk_schedule_from_csv(csv_path):
    with open(csv_path, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            # Upload media if URL provided
            media_id = None
            if row['media_url']:
                media_response = requests.post(
                    f"{POSTIZ_URL}/upload-from-url",
                    headers=headers,
                    json={"url": row['media_url']}
                )
                media_id = media_response.json()['id']

            # Create post
            create_post(
                integration_id=row['integration_id'],
                content=row['content'],
                media_id=media_id,
                schedule_time=row['schedule_time']
            )
            print(f"Scheduled: {row['content'][:50]}...")

bulk_schedule_from_csv("content_calendar.csv")
```

### Node.js Code Example

**Basic Integration (Node.js):**
```javascript
const axios = require('axios');

const POSTIZ_API_KEY = 'your-api-key-here';
const POSTIZ_URL = 'https://postiz.your-domain.com/api/public/v1';

const headers = {
  'Authorization': POSTIZ_API_KEY,
  'Content-Type': 'application/json'
};

// Get all connected integrations
async function getIntegrations() {
  const response = await axios.get(`${POSTIZ_URL}/integrations`, { headers });
  return response.data;
}

// Create post
async function createPost(integrationId, content, mediaId = null) {
  const payload = {
    type: 'now',
    shortLink: false,
    tags: [],
    posts: [{
      integration: { id: integrationId },
      value: [{
        content: content,
        image: mediaId ? [{ id: mediaId }] : []
      }],
      settings: { __type: 'x' }
    }]
  };

  const response = await axios.post(`${POSTIZ_URL}/posts`, payload, { headers });
  return response.data;
}

// Example usage
(async () => {
  const integrations = await getIntegrations();
  console.log('Connected accounts:', integrations);

  const twitterIntegration = integrations.find(i => i.providerIdentifier === 'x');
  if (twitterIntegration) {
    const result = await createPost(
      twitterIntegration.id,
      'Hello from Node.js automation!'
    );
    console.log('Post created:', result);
  }
})();
```

### n8n Integration (Official Community Node)

**Installation:**
```bash
# Via npm (in n8n custom directory)
npm i n8n-nodes-postiz

# Or via n8n UI: Settings > Community Nodes > Install "n8n-nodes-postiz"
```

**Authentication in n8n:**
1. Go to Credentials section
2. Add Credential → Search "Postiz API"
3. Paste API key
4. For self-hosted: Set host to `http://yourdomain.com/api`

**Example Workflow: WordPress → Postiz**
> "Auto-Generate Platform-Optimized Social Media Posts from WordPress with Claude & Postiz"

Workflow steps:
1. Trigger: New WordPress post published (RSS/webhook)
2. Claude AI: Generate platform-specific content variations
3. Postiz Upload: Upload featured image
4. Postiz Create: Schedule posts to LinkedIn, X, Facebook, Instagram

**n8n Template Library:**
- [Multi-Platform Social Media Publisher](https://n8n.io/workflows/5943-multi-platform-social-media-publisher-with-airtable-google-drive-and-postiz/)
- [Auto-Generate Posts from WordPress](https://n8n.io/workflows/7046-auto-generate-platform-optimized-social-media-posts-from-wordpress-with-claude-and-postiz/)
- [Video Content Automation](https://n8n.io/workflows/6653-automate-video-content-posting-to-multiple-social-platforms-with-postiz/)

### Make.com Integration

**Official Make App:**
- [Postiz Make App Documentation](https://apps.make.com/postiz)

**Capabilities:**
- Create posts via Make scenarios
- Upload media
- Retrieve post analytics
- Connect to 1000+ Make-compatible apps

### Zapier Integration

**Integration Method:** HTTP requests via Zapier Webhooks (premium feature)

**Limitations:**
> "For more direct connections, Zapier offers two main tools, though both come with significant limitations: Webhooks are a premium feature, and you can have only one starting trigger per Zap."

**Recommendation:** Use n8n or Make for better Postiz integration (open-source, more flexible).

### MCP Server Implementation

**Current Status:** No official Postiz MCP (Model Context Protocol) server exists as of December 2025.

**Potential Implementation:**
Given Postiz's REST API structure, a custom MCP server could expose:
- `postiz://create-post` resource
- `postiz://upload-media` resource
- `postiz://list-integrations` tool

**Community Opportunity:** Open for community contribution at [github.com/gitroomhq/postiz-app](https://github.com/gitroomhq/postiz-app).

---

## 6. Self-Hosted Specific Considerations

### API Differences: Cloud vs Self-Hosted

| Feature | Cloud (postiz.com) | Self-Hosted |
|---------|-------------------|-------------|
| **Base URL** | `api.postiz.com/public/v1` | `{YOUR_DOMAIN}/api/public/v1` |
| **API Key Location** | Settings → Public API | Settings → Public API |
| **Rate Limiting** | 30 requests/hour | 30 requests/hour (same) |
| **Provider Setup** | Pre-configured | Manual .env configuration |
| **HTTPS Requirement** | Enforced | Optional (NOT_SECURED=true for dev) |
| **Storage** | Cloud (CDN) | Local/R2/S3 configurable |
| **Cost** | Usage-based pricing | Free (infrastructure costs only) |

**No API Feature Differences:**
> "The self-hosted version is perfect for automation (API) with platforms like N8N, Make.com, Zapier, etc."

All Public API features work identically on self-hosted installations.

### Database Direct Access vs API

**Database Schema:**
PostgreSQL stores:
- User accounts
- Scheduled posts
- Social media integrations
- Analytics data
- Application configuration

**When to Use Direct Database Access:**
- Advanced analytics not exposed via API
- Bulk data migrations
- Custom reporting dashboards
- Performance-critical queries

**When to Use API:**
- Creating/scheduling posts
- Upload media
- Production automation workflows
- External tool integrations

**Caution:**
> "The backend itself basically is the API, but it's not well documented for general API usage - it's only used by the frontend, and it's fairly tightly coupled with the frontend as well."

Avoid using internal/frontend API endpoints. Stick to documented Public API or direct database queries.

### Docker Deployment API Exposure

**Port Configuration:**

```yaml
services:
  backend:
    ports:
      - "3000:3000"  # API port - most users don't expose publicly
    environment:
      - DATABASE_URL=postgresql://...
      - REDIS_URL=redis://postiz-redis:6379
      - NEXT_PUBLIC_BACKEND_URL=https://postiz.domain.com/api

  postgres:
    ports:
      - "5432:5432"  # Database - DO NOT expose publicly

  redis:
    ports:
      - "6379:6379"  # Redis - DO NOT expose publicly
```

**Security Best Practice:**
> "Port 3000/tcp is used for the Backend service (the API). Most users do not need to expose this port publicly."

Use reverse proxy (Caddy/Nginx) to expose only HTTPS port 443.

### Network/Firewall Configuration for API Access

**Recommended Architecture:**
```
Internet
    ↓
Reverse Proxy (Caddy/Nginx) :443 [HTTPS]
    ↓
Docker Network (Internal)
    ├─ Backend :3000 (API)
    ├─ Frontend :5000 (UI)
    ├─ PostgreSQL :5432 (Database)
    └─ Redis :6379 (Queue)
```

**Caddy Configuration Example:**
```caddyfile
postiz.yourdomain.com {
    reverse_proxy frontend:5000

    handle /api/* {
        reverse_proxy backend:3000
    }
}
```

**Firewall Rules:**
- Allow inbound: 443/tcp (HTTPS only)
- Block inbound: 3000, 5000, 5432, 6379
- Allow internal Docker network communication

**OAuth Callback Requirements:**
> "Some features require internet exposure – For platforms like Twitter and Facebook, you'll need a public-facing URL to complete API authentication."

Social platforms need to reach your Postiz instance for OAuth callbacks. Self-hosted must be internet-accessible (not localhost).

---

## 7. Common Challenges & Solutions

### Known API Limitations

**1. Beta Status (Recently Resolved):**
> "The Postiz Public API is currently in Beta and in active development. It does not provide all of the Postiz features currently."

**Status:** API launched officially in January 2025 (issue #350 closed: "API has been implemented").

**2. Rate Limiting (30 requests/hour):**

**Workaround:** Batch operations in single requests:
```json
{
  "type": "schedule",
  "posts": [
    {"integration": {"id": "twitter-id"}, "value": [...]},
    {"integration": {"id": "linkedin-id"}, "value": [...]},
    {"integration": {"id": "facebook-id"}, "value": [...]}
  ]
}
```
One API request → 3 scheduled posts.

**3. Incomplete Platform Settings Documentation:**

Some of the 27 platforms lack detailed `settings` object documentation.

**Workaround:**
1. Create post manually via Postiz UI
2. Query `GET /posts` to inspect actual JSON structure
3. Replicate settings object in API calls

**4. No Multi-Tenant Token Management:**

**Current Limitation:**
> "Feature request for Postiz to accept tokens dynamically (via API or database) from an external system like an authorization portal, instead of requiring API keys/secrets in docker-compose.yml."

**Workaround:** Each tenant runs separate Postiz instance with isolated API key.

### Workarounds for Missing Features

**Missing: Analytics API Endpoints**

**Workaround:** Direct PostgreSQL queries for analytics data:
```sql
SELECT
  post_id,
  platform,
  engagement_metrics,
  publish_date
FROM analytics_table
WHERE created_at > NOW() - INTERVAL '30 days';
```

**Missing: Webhook Events for Post Status**

**Workaround:** Polling pattern with `GET /posts`:
```python
import time
import requests

def poll_post_status(post_id, max_attempts=10):
    for attempt in range(max_attempts):
        response = requests.get(
            f"{POSTIZ_URL}/posts/{post_id}",
            headers=headers
        )
        post = response.json()

        if post['state'] == 'PUBLISHED':
            return 'success'
        elif post['state'] == 'ERROR':
            return 'failed'

        time.sleep(60)  # Check every minute

    return 'timeout'
```

**Missing: Content Approval API**

**Workaround:** Draft-based approval workflow (see Section 4).

### Community Tools and Extensions

**1. n8n Community Node:**
- **GitHub:** [github.com/gitroomhq/postiz-n8n](https://github.com/gitroomhq/postiz-n8n)
- **npm:** `n8n-nodes-postiz`
- **Features:** Create posts, upload media, list integrations

**2. Postiz NodeJS SDK:**
> "You can use the Postiz NodeJS SDK to interact with the Postiz Public API."

**Status:** Mentioned in docs but not yet publicly released (as of December 2025).

**3. Browser Extension (Community Request):**
> "I loved Buffer's browser extension that allows collecting articles to share on multiple social networks."

**Status:** Requested but not yet available. Open for community contribution.

**4. Custom Connectors:**
- Make.com official app
- Pipedream integration available
- Zapier via HTTP requests

**5. dlt (Data Load Tool) Pipeline:**
> "You can set up a complete Postiz data pipeline from API credentials to your first data load in just 10 minutes using dlt's REST API connector with bearer token authentication."

Load Postiz data into data warehouses for advanced analytics.

### Troubleshooting API Issues

**Issue: 400 Bad Request on Create Post**

**Cause:** Missing required fields (`shortLink`, `tags`)

**Solution:**
```json
{
  "type": "now",
  "shortLink": false,  // Required
  "tags": [],          // Required (can be empty array)
  "posts": [...]
}
```

**Issue: Scheduled Posts Stuck in QUEUE**

**GitHub Issue:** [#818 - Scheduled posts created via Cursor remain in QUEUE](https://github.com/gitroomhq/postiz-app/issues/818)

**Causes:**
- Cron worker not running
- Redis connection failure
- Invalid integration credentials

**Solution:**
1. Check Docker logs: `docker compose logs backend worker`
2. Verify Redis connection: `docker compose ps redis`
3. Restart workers: `docker compose restart worker`

**Issue: 401 Unauthorized**

**Causes:**
- Invalid API key
- API key regenerated (old key still in use)
- Incorrect header format

**Solution:**
```bash
# Correct format
Authorization: your-actual-api-key-here

# Not "Bearer your-key" - just the key directly
```

**Issue: CORS Errors (Self-Hosted)**

**Cause:** Frontend/backend URL mismatch

**Solution:** Verify environment variables:
```bash
FRONTEND_URL=https://postiz.domain.com
NEXT_PUBLIC_BACKEND_URL=https://postiz.domain.com/api
BACKEND_INTERNAL_URL=http://backend:3000
```

**Issue: Rate Limit Exceeded**

**Error:** 429 Too Many Requests

**Solution:**
1. Implement exponential backoff
2. Batch operations (see Section 7)
3. Spread API calls across hour window

```python
import time
from functools import wraps

def rate_limit_retry(max_retries=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except requests.exceptions.HTTPError as e:
                    if e.response.status_code == 429:
                        wait_time = 2 ** attempt * 60  # Exponential backoff
                        time.sleep(wait_time)
                    else:
                        raise
            raise Exception("Max retries exceeded")
        return wrapper
    return decorator

@rate_limit_retry()
def create_post_with_retry(*args, **kwargs):
    return create_post(*args, **kwargs)
```

---

## 8. Complete Working Examples

### Example 1: RSS to Social Media Pipeline

**Scenario:** Automatically share new blog posts to X, LinkedIn, and Facebook.

**Python Implementation:**
```python
import feedparser
import requests
from datetime import datetime, timedelta

POSTIZ_URL = "https://postiz.your-domain.com/api/public/v1"
POSTIZ_API_KEY = "your-api-key"
RSS_FEED = "https://yourblog.com/feed"

headers = {
    "Authorization": POSTIZ_API_KEY,
    "Content-Type": "application/json"
}

def get_latest_post():
    feed = feedparser.parse(RSS_FEED)
    return feed.entries[0]  # Most recent post

def upload_featured_image(image_url):
    response = requests.post(
        f"{POSTIZ_URL}/upload-from-url",
        headers=headers,
        json={"url": image_url}
    )
    return response.json()['id']

def create_multi_platform_post(title, link, image_id, integrations):
    # Platform-specific content
    content_map = {
        'x': f"{title}\n\n{link}",  # X (280 char limit)
        'linkedin': f"New article: {title}\n\nRead more: {link}",  # LinkedIn (professional)
        'facebook': f"Check out our latest blog post!\n\n{title}\n\n{link}"  # Facebook
    }

    posts = []
    for integration in integrations:
        platform = integration['providerIdentifier']
        posts.append({
            "integration": {"id": integration['id']},
            "value": [{
                "content": content_map.get(platform, f"{title}\n{link}"),
                "image": [{"id": image_id}]
            }],
            "settings": {"__type": platform}
        })

    payload = {
        "type": "schedule",
        "date": (datetime.utcnow() + timedelta(hours=1)).isoformat() + "Z",
        "shortLink": True,  # Shorten links
        "tags": [{"value": "blog", "label": "Blog Post"}],
        "posts": posts
    }

    response = requests.post(f"{POSTIZ_URL}/posts", headers=headers, json=payload)
    return response.json()

# Main automation
def run_rss_automation():
    # Get integrations
    integrations_response = requests.get(f"{POSTIZ_URL}/integrations", headers=headers)
    integrations = integrations_response.json()

    # Get latest blog post
    latest_post = get_latest_post()

    # Upload featured image
    image_id = upload_featured_image(latest_post.enclosures[0].href)

    # Schedule to all platforms
    result = create_multi_platform_post(
        title=latest_post.title,
        link=latest_post.link,
        image_id=image_id,
        integrations=integrations
    )

    print(f"Scheduled {len(integrations)} posts for: {latest_post.title}")
    return result

# Run every hour via cron
if __name__ == "__main__":
    run_rss_automation()
```

### Example 2: Evergreen Content Recycler

**Scenario:** Automatically recycle top-performing posts every 30 days.

**Node.js Implementation:**
```javascript
const axios = require('axios');

const POSTIZ_URL = 'https://postiz.your-domain.com/api/public/v1';
const POSTIZ_API_KEY = 'your-api-key';
const headers = { 'Authorization': POSTIZ_API_KEY };

// Get posts from last 90 days
async function getTopPosts() {
  const endDate = new Date().toISOString();
  const startDate = new Date(Date.now() - 90 * 24 * 60 * 60 * 1000).toISOString();

  const response = await axios.get(
    `${POSTIZ_URL}/posts?startDate=${startDate}&endDate=${endDate}`,
    { headers }
  );

  // Filter for posts with high engagement (implement your logic)
  return response.data.filter(post => post.engagement > 100);
}

// Reschedule post for 7 days from now
async function recyclePost(post) {
  const scheduleDate = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000).toISOString();

  const payload = {
    type: 'schedule',
    date: scheduleDate,
    shortLink: false,
    tags: [{ value: 'evergreen', label: 'Evergreen' }],
    posts: [{
      integration: post.integration,
      value: post.value,
      settings: post.settings
    }]
  };

  const response = await axios.post(`${POSTIZ_URL}/posts`, payload, { headers });
  return response.data;
}

// Main recycling function
async function recycleEvergreenContent() {
  const topPosts = await getTopPosts();
  const recycledCount = topPosts.length;

  for (const post of topPosts) {
    await recyclePost(post);
    console.log(`Recycled: ${post.value[0].content.substring(0, 50)}...`);
  }

  console.log(`Total recycled: ${recycledCount} posts`);
}

// Run monthly via cron
recycleEvergreenContent();
```

### Example 3: Customer Success Story Automation

**Scenario:** When CRM marks deal as "Closed-Won," auto-post success story.

**n8n Workflow (Pseudocode):**
```
1. Webhook Trigger (HubSpot deal updated)
   ↓
2. Filter Node (deal.stage === "Closed-Won")
   ↓
3. HTTP Request (fetch company logo from CRM)
   ↓
4. Postiz Upload Node (upload logo)
   ↓
5. AI Node (ChatGPT: generate success story from deal notes)
   ↓
6. Postiz Create Node:
   - Platform: LinkedIn
   - Content: AI-generated story
   - Image: Customer logo
   - Schedule: Tomorrow 9am
   ↓
7. Slack Notification ("Success story scheduled!")
```

**JSON Payload (Step 6):**
```json
{
  "type": "schedule",
  "date": "2025-12-10T09:00:00.000Z",
  "shortLink": false,
  "tags": [
    {"value": "customer-success", "label": "Customer Success"},
    {"value": "case-study", "label": "Case Study"}
  ],
  "posts": [{
    "integration": {"id": "linkedin-integration-id"},
    "value": [{
      "content": "{{$json.ai_generated_story}}",
      "image": [{"id": "{{$json.uploaded_logo_id}}"}]
    }],
    "settings": {
      "__type": "linkedin",
      "post_type": "post"
    }
  }]
}
```

---

## 9. Best Practices for "Walking the Dog" Automation

### 1. Content Staging Strategy
```
Draft → Review → Approve → Queue → Publish
```

**Implementation:**
- Create all posts as `type: "draft"`
- Review via UI or custom dashboard
- API call to update `type: "schedule"` after approval

### 2. Platform-Specific Optimization
```python
platform_rules = {
    'x': {'max_chars': 280, 'hashtags': 2, 'media_limit': 4},
    'linkedin': {'max_chars': 3000, 'hashtags': 3, 'media_limit': 9},
    'instagram': {'max_chars': 2200, 'hashtags': 30, 'media_limit': 10},
    'facebook': {'max_chars': 63206, 'hashtags': 'unlimited', 'media_limit': 10}
}

def optimize_content(content, platform):
    rules = platform_rules[platform]
    # Truncate, adjust hashtags, etc.
    return optimized_content
```

### 3. Retry Logic with Dead Letter Queue
```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=4, max=60))
def post_with_retry(payload):
    try:
        return requests.post(f"{POSTIZ_URL}/posts", headers=headers, json=payload)
    except Exception as e:
        # Log to dead letter queue for manual review
        log_failed_post(payload, str(e))
        raise
```

### 4. Media Optimization Pipeline
```python
from PIL import Image
import io

def optimize_image_for_social(image_path, platform):
    img = Image.open(image_path)

    # Platform-specific sizes
    sizes = {
        'x': (1200, 675),        # 16:9
        'instagram': (1080, 1080), # 1:1
        'linkedin': (1200, 627)    # 1.91:1
    }

    img = img.resize(sizes[platform], Image.LANCZOS)

    # Compress
    buffer = io.BytesIO()
    img.save(buffer, format='JPEG', quality=85, optimize=True)
    buffer.seek(0)

    return buffer
```

### 5. Analytics-Driven Scheduling
```python
# Analyze past performance to find optimal posting times
def get_best_posting_time(integration_id, day_of_week):
    # Query analytics (direct DB or future API endpoint)
    best_times = analyze_engagement_by_hour(integration_id)
    return best_times[day_of_week]

# Schedule at optimal time
schedule_time = get_best_posting_time('twitter-id', 'monday')
payload['date'] = schedule_time.isoformat() + 'Z'
```

### 6. Monitoring and Alerting
```python
import logging

def monitor_posting_pipeline():
    # Check for posts stuck in QUEUE > 24 hours
    stuck_posts = get_posts_by_state('QUEUE', hours_ago=24)

    if len(stuck_posts) > 0:
        logging.error(f"{len(stuck_posts)} posts stuck in queue!")
        send_slack_alert(f"⚠️ {len(stuck_posts)} posts need attention")

    # Check error rate
    error_posts = get_posts_by_state('ERROR', hours_ago=1)
    if len(error_posts) > 5:
        logging.critical("High error rate detected!")
        send_pagerduty_alert("Postiz posting failure spike")
```

---

## 10. Quick Reference

### API Endpoints Cheat Sheet

```bash
# Authentication
Authorization: YOUR_API_KEY

# List integrations
GET /integrations

# Create post (immediate)
POST /posts
{
  "type": "now",
  "shortLink": false,
  "tags": [],
  "posts": [{"integration": {"id": "..."}, "value": [{"content": "..."}]}]
}

# Create post (scheduled)
POST /posts
{
  "type": "schedule",
  "date": "2025-12-10T15:00:00.000Z",
  ...
}

# Upload media (multipart)
POST /upload
Content-Type: multipart/form-data
file: <binary>

# Upload from URL
POST /upload-from-url
{"url": "https://example.com/image.jpg"}

# Get posts
GET /posts?startDate=2025-01-01T00:00:00Z&endDate=2025-12-31T23:59:59Z

# Delete post
DELETE /posts/{post_id}
```

### Common Payload Templates

**Minimal Post:**
```json
{
  "type": "now",
  "shortLink": false,
  "tags": [],
  "posts": [{
    "integration": {"id": "integration-id"},
    "value": [{"content": "Your post content"}],
    "settings": {"__type": "x"}
  }]
}
```

**Post with Media:**
```json
{
  "type": "now",
  "shortLink": false,
  "tags": [],
  "posts": [{
    "integration": {"id": "integration-id"},
    "value": [{
      "content": "Check out this image!",
      "image": [{"id": "uploaded-media-id", "path": "https://..."}]
    }],
    "settings": {"__type": "x"}
  }]
}
```

**Multi-Platform Post:**
```json
{
  "type": "schedule",
  "date": "2025-12-10T12:00:00.000Z",
  "shortLink": true,
  "tags": [{"value": "tag1", "label": "Tag 1"}],
  "posts": [
    {"integration": {"id": "twitter-id"}, "value": [...], "settings": {"__type": "x"}},
    {"integration": {"id": "linkedin-id"}, "value": [...], "settings": {"__type": "linkedin"}},
    {"integration": {"id": "facebook-id"}, "value": [...], "settings": {"__type": "facebook"}}
  ]
}
```

### Environment Variables Template

```bash
# Self-Hosted Postiz .env Template

# Database
DATABASE_URL=postgresql://user:password@postgres:5432/postiz

# Redis
REDIS_URL=redis://redis:6379

# URLs
MAIN_URL=https://postiz.your-domain.com
FRONTEND_URL=https://postiz.your-domain.com
NEXT_PUBLIC_BACKEND_URL=https://postiz.your-domain.com/api
BACKEND_INTERNAL_URL=http://backend:3000

# Security
JWT_SECRET=generate-long-random-string-here
NOT_SECURED=false  # Set to true only for local dev without HTTPS

# Storage (Local)
STORAGE_PROVIDER=local
UPLOAD_DIRECTORY=/uploads
NEXT_PUBLIC_UPLOAD_DIRECTORY=/uploads

# Social Platform APIs (configure as needed)
# X_CLIENT_ID=
# X_CLIENT_SECRET=
# LINKEDIN_CLIENT_ID=
# LINKEDIN_CLIENT_SECRET=
# etc...

# AI Features (optional)
# OPENAI_API_KEY=sk-...
```

### Troubleshooting Checklist

- [ ] API key copied correctly from Settings → Public API
- [ ] Base URL includes `/api/public/v1` for self-hosted
- [ ] `Authorization` header (not `Bearer`, just the key)
- [ ] `shortLink` and `tags` fields included in POST /posts
- [ ] `__type` matches platform identifier in settings
- [ ] Integration ID exists in `GET /integrations` response
- [ ] Media uploaded before referencing in post
- [ ] Date in ISO 8601 format with timezone (UTC recommended)
- [ ] Rate limit not exceeded (30 requests/hour)
- [ ] Docker containers running (`docker compose ps`)
- [ ] Redis connection healthy (`docker compose logs redis`)
- [ ] Backend logs checked (`docker compose logs backend`)

---

## 11. Additional Resources

### Official Documentation
- **Public API Docs**: [docs.postiz.com/public-api](https://docs.postiz.com/public-api)
- **GitHub Repository**: [github.com/gitroomhq/postiz-app](https://github.com/gitroomhq/postiz-app)
- **Docker Installation**: [docs.postiz.com/installation/docker](https://docs.postiz.com/installation/docker)
- **Development Guide**: [docs.postiz.com/installation/development](https://docs.postiz.com/installation/development)

### Community Resources
- **GitHub Issues**: [github.com/gitroomhq/postiz-app/issues](https://github.com/gitroomhq/postiz-app/issues)
- **Discord Server**: Community support and discussions
- **n8n Community Node**: [github.com/gitroomhq/postiz-n8n](https://github.com/gitroomhq/postiz-n8n)
- **Make.com Integration**: [apps.make.com/postiz](https://apps.make.com/postiz)

### Workflow Templates
- **n8n Template Library**: [n8n.io/integrations/postiz](https://n8n.io/integrations/postiz/)
- **Pipedream Postiz Workflows**: [pipedream.com/apps/postiz](https://pipedream.com/apps/postiz)

### Related Tools
- **dlt Postiz Connector**: [dlthub.com/workspace/source/postiz](https://dlthub.com/workspace/source/postiz)
- **Postiz Setup Guides**: [bababuilds.com/blog/postiz-setup-guide](https://bababuilds.com/blog/postiz-setup-guide/)

### Research Articles
- **Unlocking Automation with Postiz API**: [skywork.ai article](https://skywork.ai/skypage/en/Unlocking-Automation:-A-Deep-Dive-into-the-Postiz-API/1976120844408123392)
- **Mastering Social Media Automation**: [addrom.com n8n guide](https://addrom.com/mastering-social-media-automation-a-deep-dive-with-n8n-and-postiz/)

---

## 12. Conclusion

Postiz's self-hosted Public API provides a robust foundation for "walking the dog" social media automation workflows. With 27+ platform integrations, REST API endpoints for posts/media/integrations, and strong ecosystem support (n8n, Make, Zapier), you can build comprehensive automation pipelines.

### Key Takeaways

1. **API is Production-Ready**: Launched January 2025, all core features available
2. **Self-Hosted = Free**: No API costs, only infrastructure (Docker + PostgreSQL + Redis)
3. **Rate Limits are Manageable**: 30 requests/hour sufficient with batching strategies
4. **Strong n8n Integration**: Official community node, extensive workflow templates
5. **Platform Coverage**: 27 social platforms supported (X, LinkedIn, Instagram, TikTok, etc.)
6. **Extensible**: Open-source codebase allows custom contributions

### Next Steps

1. **Deploy Postiz**: Follow [Docker installation guide](https://docs.postiz.com/installation/docker)
2. **Generate API Key**: Settings → Public API
3. **Test Integration**: Start with `GET /integrations` to verify connectivity
4. **Build First Workflow**: Try n8n template or custom Python script
5. **Scale Up**: Implement monitoring, error handling, analytics

### Open Questions for Community

- MCP server implementation for LLM integration?
- Advanced analytics API endpoints?
- Webhook events for real-time post status?
- Multi-tenant token management?

Contribute at: [github.com/gitroomhq/postiz-app](https://github.com/gitroomhq/postiz-app)

---

## Sources

### Primary Sources
- [Public API - Postiz Docs](https://docs.postiz.com/public-api)
- [GitHub - gitroomhq/postiz-app](https://github.com/gitroomhq/postiz-app)
- [Postiz API Documentation (GitHub)](https://github.com/gitroomhq/postiz-docs/blob/main/pages/public-api.mdx)
- [Postiz API Issue #350](https://github.com/gitroomhq/postiz-app/issues/350)
- [Postiz Create/Update Post Issue #717](https://github.com/gitroomhq/postiz-app/issues/717)

### Integration Documentation
- [n8n Postiz Community Node](https://github.com/gitroomhq/postiz-n8n)
- [Postiz on Make.com](https://apps.make.com/postiz)
- [n8n Postiz Integrations](https://n8n.io/integrations/postiz/)
- [n8n WordPress + Postiz Workflow](https://n8n.io/workflows/7046-auto-generate-platform-optimized-social-media-posts-from-wordpress-with-claude-and-postiz/)

### Deployment & Setup
- [Postiz Docker Installation](https://docs.postiz.com/installation/docker)
- [Postiz Setup Guide - BabaBuilds](https://bababuilds.com/blog/postiz-setup-guide/)
- [Deploy Postiz on Railway](https://railway.com/deploy/postiz)
- [How to Set Up Postiz on Docker](https://modernizingtech.com/tips/other/how-to-set-up-postiz-on-docker/)

### Analysis & Guides
- [Unlocking Automation: A Deep Dive into the Postiz API](https://skywork.ai/skypage/en/Unlocking-Automation:-A-Deep-Dive-into-the-Postiz-API/1976120844408123392)
- [Mastering Social Media Automation with N8N and Postiz](https://addrom.com/mastering-social-media-automation-a-deep-dive-with-n8n-and-postiz/)
- [Schedule Posts with n8n - Postiz Blog](https://postiz.com/blog/use-postiz-with-n-8-n)
- [Postiz Review - BasicUtils](https://basicutils.com/learn/reviews/postiz)

### Additional Resources
- [Load Postiz Data with dltHub](https://dlthub.com/workspace/source/postiz)
- [Postiz on Pipedream](https://pipedream.com/apps/postiz/integrations/news-api)
- [n8n-nodes-postiz on npm](https://www.npmjs.com/package/n8n-nodes-postiz)

---

**Research Completed:** December 9, 2025
**Completeness:** 90% (Missing: MCP server implementation, some platform-specific settings)
**Performance:** 2.8 minutes (12 parallel searches + 4 targeted fetches)
