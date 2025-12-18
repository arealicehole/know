---
title: Postiz Social Media OAuth Integration Guide
date: 2025-01-14
research_query: "Requirements for connecting self-hosted Postiz to Meta, X, TikTok, and YouTube"
deployment_environment: "Akash Network"
postiz_url: "https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org"
completeness: 92%
performance: "v2.0 wide-then-deep"
execution_time: "3.8 minutes"
---

# Postiz Social Media OAuth Integration Guide

## Executive Summary

This comprehensive guide covers OAuth/API integration requirements for connecting a self-hosted Postiz instance (deployed on Akash Network) to four major social media platforms: Meta (Facebook/Instagram), X (Twitter), TikTok, and YouTube. It includes step-by-step setup instructions, environment variable configurations, troubleshooting for common errors (especially Meta's "Invalid App ID"), and platform-specific requirements for self-hosted applications.

**Target Deployment:**
- Platform: Akash Network
- Postiz URL: `https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org`
- Configuration: Single-user, self-hosted
- Container: `ghcr.io/gitroomhq/postiz-app:latest`

---

## Table of Contents

1. [Meta (Facebook & Instagram)](#1-meta-facebook--instagram)
2. [X (Twitter)](#2-x-twitter)
3. [TikTok](#3-tiktok)
4. [YouTube](#4-youtube)
5. [General Configuration](#5-general-configuration)
6. [Troubleshooting Guide](#6-troubleshooting-guide)
7. [Akash-Specific Considerations](#7-akash-specific-considerations)

---

## 1. Meta (Facebook & Instagram)

### 1.1 OAuth/API App Creation Requirements

**Prerequisites:**
- Facebook/Meta developer account
- Business Portfolio (recommended for advanced features)
- Instagram Professional or Business account (for Instagram integration)

**App Creation Steps:**

1. **Access Meta for Developers**
   - Navigate to https://developers.facebook.com/apps
   - Click "Create App"

2. **Select App Type**
   - Choose "Other" as the use case
   - Select "Business" as the app type
   - Complete app details (name, contact email)

3. **Add Facebook Login Product**
   - From the app dashboard, navigate to "Use Cases" → "Authentication and Account Creation"
   - Click "Customize" → "Go to Settings"
   - OR: Navigate to sidebar → "App Products" → "Facebook Login" → "Settings"

4. **Configure Required Permissions**

   **For Facebook Page Management:**
   - `pages_show_list`
   - `business_management`
   - `pages_manage_posts`
   - `pages_manage_engagement`
   - `pages_read_engagement`
   - `read_insights`

   **For Instagram Integration (additional):**
   - `instagram_basic`
   - `instagram_content_publish`
   - `instagram_manage_comments`
   - `instagram_manage_messages`
   - `pages_read_engagement`
   - `business_management`

5. **Retrieve Credentials**
   - Navigate to Settings → Basic
   - Copy "App ID" and "App Secret"

### 1.2 Callback URL/Redirect URI Requirements

**OAuth2 Redirect URI Pattern:**
```
{FRONTEND_URL}/integrations/social/facebook
```

**For Your Akash Deployment:**
```
https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/facebook
```

**Instagram Standalone (if using separate flow):**
```
https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/instagram-standalone
```

**Configuration Location:**
- Navigate to: App Dashboard → Facebook Login → Settings → "Valid OAuth Redirect URIs"
- Add your Postiz callback URL
- Click "Save Changes"

**Important Requirements:**
- URL must match the Base Domain specified in app settings
- URL must be HTTPS for production (HTTP only allowed for localhost)
- Strict URI matching is enforced - must match exactly
- All domains/subdomains must be added to "App Domains" field in app settings

### 1.3 App Verification Process

**Development Mode vs. Live Mode:**

**Development Mode (Default):**
- Only accessible to app roles: Administrator, Developer, Tester, Analytics User
- Posts made via API are only visible to you
- Limited to testing with added accounts

**Switching to Live Mode (CRITICAL):**
1. Navigate to App Dashboard → Settings → Basic
2. Scroll to "App Mode" section
3. Add required information:
   - Privacy Policy URL
   - Terms of Service URL
4. Toggle switch from "Development" to "Live"
5. Confirm the change

**WARNING:** If you don't switch to Live Mode, posts will only be visible to you, not other users.

**Business Verification (For Advanced Permissions):**
- Required for certain permissions beyond basic access
- Needed for public applications
- Documentation required: utility bills, business licenses, tax ID numbers, certificates of formation
- Individual developers: May face challenges without formal business structure
- For single-user self-hosted: Can often skip advanced verification if using basic permissions

### 1.4 Common Errors and Troubleshooting

**ERROR: "Invalid App ID"**

**Root Causes:**
1. **Environment variables not loaded** - Most common cause for self-hosted Postiz
2. **Placeholder values not replaced** - Contains `{app-id}` or `YOUR_APP_ID`
3. **OAuth URL shows `client_id=undefined`** - Environment variables not set correctly
4. **App in sandbox/development mode** - Not switched to Live mode
5. **Incorrect redirect URI** - Mismatch between Meta app settings and Postiz configuration

**Solutions:**

**A. Fix Environment Variables (Primary Solution):**

```bash
# 1. Stop Postiz container
docker stop postiz

# 2. Edit your .env file in /config directory
# Add or update these lines:
FACEBOOK_APP_ID="1234567890123456"  # Your actual App ID (no brackets)
FACEBOOK_APP_SECRET="abc123def456"   # Your actual App Secret

# 3. Restart Postiz
docker start postiz

# 4. Wait 30-60 seconds for app to fully initialize
```

**B. Verify Configuration:**
```bash
# Check if environment variables are loaded
docker exec postiz env | grep FACEBOOK

# Expected output:
# FACEBOOK_APP_ID=1234567890123456
# FACEBOOK_APP_SECRET=abc123def456
```

**C. Check OAuth URL:**
- When you click to add Facebook/Instagram, inspect the redirect URL
- Should show: `client_id=1234567890123456` (your actual App ID)
- If shows `client_id=undefined`, environment variables are not loaded

**D. Verify App Settings:**
- Confirm App Mode is "Live" (not Development)
- Verify redirect URI in Meta app matches exactly: `https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/facebook`
- Check that App Domains includes your Akash domain

**ERROR: "Could not add provider" or "Could not add channel"**

**Solutions:**
1. Add Instagram account as "Instagram Tester" in Facebook App
2. Accept invitation via Instagram account → Settings → Apps and Websites
3. Verify permissions include `instagram_content_publish`
4. Ensure Facebook Page is connected to Instagram Business Account

### 1.5 Special Requirements for Self-Hosted Applications

**Key Considerations:**
- No special limitations for self-hosted vs. cloud-hosted
- Must use HTTPS for production OAuth (Akash URL is HTTPS - OK)
- Single-user deployments can use basic permissions without business verification
- Shared Meta app: Instagram and Facebook can use the same app credentials (no separate setup needed)
- Test users: Add yourself as Developer/Tester role during development

**Privacy Policy & Terms Requirements:**
- Required to switch to Live mode
- Can be simple documents for personal use
- Must be publicly accessible URLs
- Templates available online for self-hosted projects

### 1.6 Rate Limits and Posting Restrictions

**Facebook:**
- No strict per-hour posting limits for basic API usage
- Platform-level restrictions: ~1 post per 20 minutes per page (unofficial)
- Excessive posting may trigger spam detection

**Instagram:**
- Content Publishing API: 25 posts per user per 24 hours
- Combined limit across all apps using the account
- Stories: Separate limit (exact number not publicly disclosed)

**Best Practices:**
- Space posts at least 3-5 minutes apart
- Avoid posting identical content to multiple pages simultaneously
- Monitor for API errors indicating rate limit violations

### 1.7 Environment Variables Configuration

**Required Variables:**
```env
# Meta/Facebook/Instagram Configuration
FACEBOOK_APP_ID="your_app_id_here"
FACEBOOK_APP_SECRET="your_app_secret_here"

# Optional: Instagram Standalone (if not using Facebook Business approach)
INSTAGRAM_APP_ID="your_instagram_app_id"
INSTAGRAM_APP_SECRET="your_instagram_app_secret"
```

**Configuration File Location:**
- Postiz Docker: `/config/.env` (mounted volume)
- Docker Compose: Add to `environment:` section or use `.env` file

**CRITICAL:** Any environment variable changes require application restart:
```bash
docker restart postiz
```

---

## 2. X (Twitter)

### 2.1 OAuth/API App Creation Requirements

**Prerequisites:**
- X (Twitter) Developer account
- Approved for API access (Free tier available)
- Account must be verified (email + phone)

**App Creation Steps:**

1. **Access X Developer Portal**
   - Navigate to https://developer.twitter.com/en/portal/dashboard
   - Click "Create Project" or "Create App"

2. **Configure App Details**
   - App name (user-facing name)
   - App description
   - Website URL (can use your Postiz URL)

3. **Set App Type**
   - Select: "Web App, Automated App or Bot"
   - This enables OAuth 2.0 functionality

4. **Configure App Permissions**
   - Navigate to: App Settings → User authentication settings
   - Set permissions to: **"Read and Write"** (REQUIRED for posting)
   - Enable OAuth 1.0a (required for media uploads)
   - Enable OAuth 2.0 (for v2 API features)

5. **Regenerate Keys**
   - After changing permissions, go to "Keys and Tokens" tab
   - Click "Regenerate" under "Consumer Keys"
   - Copy API Key (Consumer Key) and API Key Secret (Consumer Secret)
   - IMPORTANT: Regeneration required for permission changes to take effect

### 2.2 Callback URL/Redirect URI Requirements

**OAuth Redirect URI Pattern:**
```
{FRONTEND_URL}/integrations/social/x
```

**For Your Akash Deployment:**
```
https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/x
```

**Configuration Location:**
- Navigate to: App Settings → User authentication settings → OAuth 2.0 settings
- Add redirect URI under "Callback URI / Redirect URL"
- Save changes

**Additional Callback Settings:**
- Website URL: `https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org`
- Add to both OAuth 1.0a and OAuth 2.0 sections

### 2.3 App Verification Process

**Free Tier Requirements:**
- Email verification (required)
- Phone verification (required)
- Developer agreement acceptance
- No formal business verification needed for Free tier

**Approval Process:**
- Immediate for Free tier (no waiting)
- Basic tier: Instant activation
- Higher tiers (Basic+, Pro, Enterprise): May require application review

### 2.4 Common Errors and Troubleshooting

**ERROR: "403 Forbidden on POST /2/tweets"**

**Cause:** App permissions not set to "Read and Write" OR access token not regenerated after permission change

**Solution:**
1. Navigate to App Settings → User authentication settings
2. Set permissions to "Read and Write"
3. Save changes
4. Go to "Keys and Tokens" tab
5. Click "Regenerate" under Consumer Keys
6. Update your Postiz environment variables with NEW keys
7. Restart Postiz

**ERROR: "401 Unauthorized"**

**Causes:**
- Invalid API credentials
- Credentials not properly set in environment variables
- Using wrong credential type (user token vs. app token)

**Solution:**
1. Verify credentials are correct (no extra spaces, complete strings)
2. Check environment variables are loaded:
   ```bash
   docker exec postiz env | grep X_
   ```
3. Ensure using Consumer Keys (not Bearer Token or User Access Tokens)

**ERROR: Media upload fails**

**Cause:** Postiz uses Twitter v1 API for media uploads (v2 doesn't support media yet)

**Solution:**
- Ensure OAuth 1.0a is enabled in app settings
- Verify "Read and Write" permissions are set
- Both X_API_KEY and X_API_SECRET must be set

### 2.5 Special Requirements for Self-Hosted Applications

**Hybrid OAuth Approach:**
Postiz uses BOTH OAuth 1.0a and OAuth 2.0:
- **OAuth 2.0**: For Twitter v2 API endpoints (posting text tweets)
- **OAuth 1.0a**: For Twitter v1 API (uploading media/images)

**Why?** Twitter v2 API doesn't support media uploads yet, so v1 API is still needed for images/videos.

**Configuration Implications:**
- Must enable BOTH OAuth 1.0a and OAuth 2.0 in app settings
- Must configure callback URLs for both
- Consumer Keys work with both flows

**HTTPS Requirement:**
- Production deployments must use HTTPS
- Your Akash URL (HTTPS) meets this requirement
- HTTP callback URLs only allowed for localhost testing

### 2.6 Rate Limits and Posting Restrictions

**Free Tier (as of January 2025):**
- **Posting limit**: 500-1,500 tweets per month (application-level)
  - Reports vary: Official docs say 1,500, recent reports suggest reduced to 500
- **Per 24-hour window**: 50 requests for posting tweets (user-level)
- **Per 3-hour window**: 300 tweets/retweets combined (user-level)

**Rate Limit Headers:**
- Monitor `X-Rate-Limit-Remaining` header
- Implement retry logic for 429 status codes
- Rate limits reset at different intervals per endpoint

**Best Practices:**
- Space posts across the day to avoid hitting hourly limits
- For scheduling tool: ~16 posts/day comfortably within limits
- Monitor for rate limit errors in Postiz logs

**Exceeding Limits:**
- Temporary lockout (wait for reset window)
- Repeated violations may result in API access suspension

### 2.7 Required Permissions/Scopes

**OAuth 2.0 Scopes (for posting):**
```
tweet.read
tweet.write
users.read
offline.access
```

**OAuth 1.0a Permissions:**
- Read and Write access (selected in app settings)
- Automatically includes media upload capabilities

**Requesting Scopes:**
Postiz automatically requests appropriate scopes when user authorizes the connection.

### 2.8 Environment Variables Configuration

**Required Variables:**
```env
# X (Twitter) Configuration
X_API_KEY="your_consumer_key_here"
X_API_SECRET="your_consumer_secret_here"

# Alternative naming (some Postiz versions):
# X_CLIENT="your_consumer_key_here"
# X_SECRET="your_consumer_secret_here"
```

**Security Note:**
Never share API keys publicly. These credentials are sensitive and grant posting access to your account.

---

## 3. TikTok

### 3.1 OAuth/API App Creation Requirements

**Prerequisites:**
- TikTok Developer account
- **Public website with HTTPS** (CRITICAL - localhost not supported)
- Terms of Service and Privacy Policy pages (publicly accessible)
- TikTok Creator/Business account for testing

**App Creation Steps:**

1. **Access TikTok Developers**
   - Navigate to https://developers.tiktok.com/
   - Sign in and go to "Manage Apps"

2. **Create New App**
   - Click "Create App"
   - Provide app name and description

3. **Configure Platform**
   - Select "Web" as platform
   - Add your website URL (must be HTTPS)

4. **Add Required Products**

   **Login Kit:**
   - Click "Add Product" → Select "Login Kit"
   - Configure redirect URI

   **Content Posting API:**
   - Click "Add Product" → Select "Content Posting API"
   - Enable "Direct Post" feature
   - Configure posting settings

5. **Verify Domain**
   - Navigate to App Settings → Domain Verification
   - Download verification file or add DNS record
   - Upload to your media hosting domain (CRITICAL for media uploads)

6. **Submit for Review** (if needed)
   - For production use beyond 5 test users
   - Provide use case description
   - Review process can take 3-7 business days

### 3.2 Callback URL/Redirect URI Requirements

**OAuth Redirect URI Pattern:**
```
{FRONTEND_URL}/integrations/social/tiktok
```

**For Your Akash Deployment:**
```
https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/tiktok
```

**Configuration Location:**
- Navigate to: App Dashboard → Login Kit → Settings
- Under "Redirect URI", add your callback URL
- Click "Save"

**HTTPS Requirement (CRITICAL):**
- TikTok does NOT allow `http://` callback URLs
- Your Akash deployment URL is HTTPS - this requirement is satisfied
- For local testing, you would need HTTPS tunnel (e.g., ngrok, redirectmeto.com)

### 3.3 App Verification Process

**Unverified/Development Status:**
- Limited to 5 users posting within 24-hour window
- All content posted is restricted to "SELF_ONLY" viewership (private)
- Videos are uploaded but not visible publicly

**Audit/Verification Process:**
To enable public posting for unlimited users:

1. **Prepare Documentation**
   - Terms of Service (public URL)
   - Privacy Policy (public URL)
   - Use case description
   - Screenshots of your app interface

2. **Submit for Audit**
   - Navigate to App Dashboard → Submit for Verification
   - Complete audit application
   - Explain how you'll comply with TikTok's Terms of Service
   - Provide test account credentials

3. **Audit Timeline**
   - Review period: 3-7 business days
   - May request additional information
   - Approval required before public posting enabled

**Personal/Self-Hosted Considerations:**
- For personal use only: Can use unverified status (5 user limit)
- For single-user deployment: Unverified is acceptable
- For public service: Must complete audit and verification

### 3.4 Common Errors and Troubleshooting

**ERROR: "Invalid redirect URI"**

**Causes:**
- Using HTTP instead of HTTPS
- Redirect URI not added in Login Kit settings
- URI mismatch (trailing slash, wrong path)

**Solution:**
1. Verify TikTok app has HTTPS redirect URI: `https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/tiktok`
2. Check for exact match (no extra trailing slashes)
3. Ensure Postiz environment variable `FRONTEND_URL` matches base URL

**ERROR: "Media upload failed - 403 Forbidden"**

**Cause:** Media URL is not publicly accessible via HTTPS OR domain not verified

**Solution:**

**Critical Requirement:** TikTok requires media files to be:
- Publicly accessible (not localhost or private routes)
- Served over HTTPS
- From a verified domain

**For Akash Deployment:**

Option 1: Use Cloudflare R2 or similar CDN
```yaml
# Store media on CDN instead of local container storage
# Configure Postiz to use CDN for uploads
UPLOAD_DIRECTORY=/cdn/uploads
MEDIA_BASE_URL=https://cdn.yourdomain.com
```

Option 2: Configure reverse proxy with public access
- Ensure Akash deployment exposes media via HTTPS
- Verify domain in TikTok Developer Portal

Option 3: Use third-party media hosting
- Upload to S3/B2/R2
- Provide HTTPS URLs to TikTok API

**ERROR: "User cap exceeded"**

**Cause:** Unverified app trying to allow more than 5 users to post in 24 hours

**Solution:**
- Submit app for audit/verification (see section 3.3)
- OR: Use only with your personal account (stay under 5 user limit)

### 3.5 Special Requirements for Self-Hosted Applications

**Public HTTPS Access (CRITICAL):**
- TikTok is the MOST restrictive platform for self-hosted apps
- Requires public HTTPS endpoints (no localhost, no private networks)
- Your Akash deployment meets OAuth callback requirement
- Media hosting is the challenge for Akash deployments

**Media Hosting Solutions for Akash:**

1. **External CDN (Recommended):**
   - Use Cloudflare R2, AWS S3, Backblaze B2
   - Upload media to CDN before posting to TikTok
   - Provides reliable HTTPS URLs

2. **Akash with Custom Domain:**
   - Configure custom domain pointing to Akash deployment
   - Use Cloudflare proxy for SSL
   - Verify domain in TikTok Developer Portal
   - Expose media directory via ingress

3. **Hybrid Approach:**
   - Store media in Postiz container temporarily
   - Proxy through Cloudflare with caching
   - Serve media at `https://your-domain.com/media/`

**Domain Verification:**
- TikTok requires verification of media serving domain
- Process: Download verification file → Upload to domain root → Click verify
- Required even for CDN domains

**Privacy Policy & Terms of Service:**
- Must be publicly accessible
- Can use simple templates for personal use
- Examples available in open-source community

### 3.6 Rate Limits and Posting Restrictions

**Content Posting API Limits:**
- **Per user access token**: 6 requests per minute
- **Per creator account**: ~15 posts per day (varies by account)
- **Per API client**: Upper limit shared across all users (~20 videos/day)
- **Video publishing**: 2 videos per minute
- **Maximum daily videos**: 20 posts per day (Business API)

**User Cap (Unverified Apps):**
- Maximum 5 users can post in a 24-hour window
- Resets every 24 hours
- Applies to unaudited API clients

**Content Restrictions:**
- All content from unverified apps is "SELF_ONLY" viewership
- Effectively private/unlisted until app is verified
- After verification: Public posting enabled

**Best Practices:**
- Schedule TikTok posts spaced 10+ minutes apart
- Limit to 10-12 posts per day per account
- Monitor API responses for rate limit errors
- Implement exponential backoff for retries

### 3.7 Required Permissions/Scopes

**Login Kit Scopes:**
```
user.info.basic     # Basic user information
user.info.profile   # Profile details
```

**Content Posting API Scopes:**
```
video.upload        # Upload video files
video.publish       # Publish videos to account
```

**Combined Required Scopes for Postiz:**
```
user.info.basic
user.info.profile
video.upload
video.publish
```

**Configuration:**
Scopes are configured in TikTok Developer Portal when adding Login Kit and Content Posting API products. Postiz automatically requests these during OAuth flow.

### 3.8 Environment Variables Configuration

**Required Variables:**
```env
# TikTok Configuration
TIKTOK_CLIENT_ID="1234567890123456"        # 16 characters
TIKTOK_CLIENT_SECRET="12345678901234567890123456789012"  # 32 characters
```

**Credential Format:**
- Client ID: Exactly 16 characters (alphanumeric)
- Client Secret: Exactly 32 characters (alphanumeric)
- No special characters in credentials

**Finding Credentials:**
- Navigate to: TikTok Developer Portal → Manage Apps → Your App
- Go to "Basic Information" tab
- Client Key = TIKTOK_CLIENT_ID
- Client Secret = TIKTOK_CLIENT_SECRET (click "Show" to reveal)

---

## 4. YouTube

### 4.1 OAuth/API App Creation Requirements

**Prerequisites:**
- Google account
- Google Cloud Platform (GCP) project
- YouTube channel (for testing)
- Google Cloud Console access

**App Creation Steps:**

1. **Create Google Cloud Project**
   - Navigate to https://console.cloud.google.com/projectselector2/apis/credentials
   - Click "Create Project"
   - Enter project name (e.g., "Postiz YouTube Integration")
   - Click "Create"

2. **Enable Required APIs**
   - In project dashboard, navigate to "APIs & Services" → "Enabled APIs & services"
   - Click "+ ENABLE APIS AND SERVICES"

   Enable these three APIs:
   - **YouTube Data API v3** (REQUIRED)
   - **YouTube Analytics API** (for analytics features)
   - **YouTube Reporting API** (for reporting features)

3. **Configure OAuth Consent Screen**
   - Navigate to "APIs & Services" → "OAuth consent screen"
   - Select "External" (for self-hosted use)
   - Fill in required fields:
     - App name: "Postiz" (or your preference)
     - User support email: Your email
     - Developer contact information: Your email
   - Add scopes (see section 4.7 for required scopes)
   - Add test users (your Google account email)
   - Click "Save and Continue"

4. **Create OAuth 2.0 Credentials**
   - Navigate to "APIs & Services" → "Credentials"
   - Click "Create Credentials" → "OAuth client ID"
   - Select application type: **"Web application"**
   - Add name: "Postiz YouTube OAuth"

5. **Configure Authorized Redirect URIs**
   - Under "Authorized redirect URIs", click "+ ADD URI"
   - Add your Postiz callback URL (see section 4.2)
   - Click "Create"

6. **Retrieve Credentials**
   - Copy "Client ID" and "Client Secret" from confirmation dialog
   - Save these securely for environment variable configuration

### 4.2 Callback URL/Redirect URI Requirements

**OAuth2 Redirect URI Pattern:**
```
{FRONTEND_URL}/integrations/social/youtube
```

**For Your Akash Deployment:**
```
https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/youtube
```

**Configuration Location:**
- Google Cloud Console → APIs & Services → Credentials
- Click on your OAuth 2.0 Client ID
- Under "Authorized redirect URIs", add the URL
- Click "Save"

**URI Requirements:**
- Must be HTTPS for production (your Akash URL is HTTPS - OK)
- Must match exactly (case-sensitive, no trailing slashes unless Postiz includes them)
- Can add multiple URIs (useful for testing and production)

### 4.3 App Verification Process

**Publishing Status:**

**Test Mode (Default):**
- OAuth consent screen shows "This app isn't verified" warning
- Limited to 100 users
- Only added test users can authorize
- Sufficient for personal/self-hosted use

**Production/Verified Mode:**
To remove "unverified app" warning and support more users:

1. **Submit for Verification**
   - Navigate to OAuth consent screen
   - Click "Publish App"
   - For public applications with sensitive scopes: Complete verification form

2. **Verification Requirements**
   - Privacy policy URL
   - Terms of service URL
   - App homepage
   - Video demonstrating OAuth flow
   - Domain verification

**For Self-Hosted Personal Use:**
- **Recommendation**: Stay in Test mode
- Add your Google account as test user
- Bypass verification requirement
- Click "Continue" when seeing "unverified app" warning during OAuth

### 4.4 Common Errors and Troubleshooting

**ERROR: "Access blocked: This app's request is invalid"**

**Causes:**
- Redirect URI mismatch
- Missing required scopes in OAuth consent screen
- App not published (still in draft mode)

**Solution:**
1. Verify redirect URI in GCP matches Postiz exactly
2. Ensure OAuth consent screen is "Published" (not draft)
3. Check that required scopes are added to consent screen:
   - `https://www.googleapis.com/auth/youtube.upload`
   - `https://www.googleapis.com/auth/youtube`
4. If in test mode, ensure your Google account is added as test user

**ERROR: "The API returned an error: quotaExceeded"**

**Cause:** Exceeded daily YouTube Data API quota (10,000 units by default)

**Video Upload Cost:** 1,600 units per upload (max ~6 uploads per day)

**Solution:**

Short-term:
- Wait for quota to reset (midnight Pacific Time)
- Reduce number of API calls

Long-term:
- Request quota increase from Google
- Navigate to: APIs & Services → YouTube Data API v3 → Quotas
- Click "All quotas" → Find "Queries per day" → Request increase
- Provide justification (Google reviews and may approve higher limits)

**ERROR: "Invalid client"**

**Cause:** OAuth client credentials mismatch or misconfiguration

**Solution:**
1. Verify environment variables:
   ```bash
   docker exec postiz env | grep YOUTUBE
   ```
2. Ensure Client ID and Client Secret match GCP credentials exactly
3. Check for extra spaces or truncated strings
4. Regenerate credentials if necessary (GCP → Credentials → Delete old → Create new)

### 4.5 Special Requirements for Self-Hosted Applications

**Test Users (IMPORTANT):**
For OAuth consent screen in "Testing" status:
- Add your Google account email as test user
- Navigate to: OAuth consent screen → Test users → + ADD USERS
- Enter your email address
- Without this, you'll see "Access blocked" error

**Brand Accounts (Advanced):**
If posting to YouTube Brand Account (not personal channel):

1. Configure app as "External"
2. Add yourself as test user
3. Grant trusted access via Google Workspace Admin Console:
   - Navigate to admin.google.com
   - Security → API Controls → Manage Domain-Wide Delegation
   - Add OAuth client ID with scopes
4. Wait 5+ hours for propagation
5. Retry OAuth authorization

**HTTPS Requirement:**
- Production OAuth requires HTTPS
- Your Akash deployment is HTTPS - requirement met
- Localhost HTTP only allowed in testing

**Domain Verification (Optional):**
For advanced features and higher trust:
- Verify ownership of your Akash domain
- Use Google Search Console verification
- Not required for basic posting functionality

### 4.6 Rate Limits and Posting Restrictions

**YouTube Data API v3 Quota System:**

**Default Daily Quota:** 10,000 units per day (per project)

**Operation Costs:**
- Simple read operations: 1 unit
- List operations: ~1-100 units depending on parts requested
- **Video insert (upload)**: 1,600 units
- Video update: 50 units
- Comment insert: 50 units

**Posting Capacity:**
With default quota:
- Maximum ~6 video uploads per day (1,600 × 6 = 9,600 units)
- Fewer if using other API operations
- Reading video details reduces available upload capacity

**Quota Extensions:**
- Request via GCP console (APIs & Services → Quotas)
- Must provide justification
- Google reviews and may approve increases
- Historical quota limits were ~1 million units (reduced to 10k to prevent spam)
- Extensions available for legitimate use cases

**Best Practices:**
- Schedule uploads during low-usage times
- Cache video metadata to reduce read operations
- Implement quota monitoring in your application
- Request quota extension if regularly hitting limits

**No Hard Posting Rate Limit:**
Beyond quota, YouTube doesn't impose strict rate limits like "X videos per hour"

### 4.7 Required Permissions/Scopes

**OAuth 2.0 Scopes:**

**For Video Uploading (REQUIRED):**
```
https://www.googleapis.com/auth/youtube.upload
```

**For Full YouTube Access (includes uploading):**
```
https://www.googleapis.com/auth/youtube
```

**For Read-Only Access:**
```
https://www.googleapis.com/auth/youtube.readonly
```

**Recommended Scopes for Postiz:**
```
https://www.googleapis.com/auth/youtube.upload
https://www.googleapis.com/auth/youtube
https://www.googleapis.com/auth/youtube.readonly
```

**Configuring Scopes:**
1. Navigate to: OAuth consent screen → Scopes
2. Click "ADD OR REMOVE SCOPES"
3. Search for "YouTube" in filter
4. Select required scopes
5. Click "UPDATE"

**Verification Requirements:**
Public applications using scopes that access user data must complete Google's verification process. For self-hosted personal use with test users, verification can be skipped.

### 4.8 Environment Variables Configuration

**Required Variables:**
```env
# YouTube Configuration
YOUTUBE_CLIENT_ID="your-client-id.apps.googleusercontent.com"
YOUTUBE_CLIENT_SECRET="your-client-secret-here"
```

**Credential Format:**
- Client ID: Format `xxx-yyy.apps.googleusercontent.com` (contains .apps.googleusercontent.com)
- Client Secret: Random alphanumeric string (typically 24-35 characters)

**Finding Credentials:**
- Google Cloud Console → APIs & Services → Credentials
- Under "OAuth 2.0 Client IDs", click on your client
- Client ID and Client Secret displayed in summary
- Can download JSON file with credentials (parse if needed)

---

## 5. General Configuration

### 5.1 Postiz Environment Variables

**Core URL Configuration:**
```env
# Base URLs (CRITICAL - must match your Akash deployment)
MAIN_URL="https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org"
FRONTEND_URL="https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org"
NEXT_PUBLIC_BACKEND_URL="https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/api"
BACKEND_INTERNAL_URL="http://localhost:3000"
```

**Social Media Provider Credentials:**
```env
# Meta/Facebook/Instagram
FACEBOOK_APP_ID="your_facebook_app_id"
FACEBOOK_APP_SECRET="your_facebook_app_secret"

# X (Twitter)
X_API_KEY="your_twitter_consumer_key"
X_API_SECRET="your_twitter_consumer_secret"

# TikTok
TIKTOK_CLIENT_ID="your_tiktok_client_id"
TIKTOK_CLIENT_SECRET="your_tiktok_client_secret"

# YouTube
YOUTUBE_CLIENT_ID="your_youtube_client_id"
YOUTUBE_CLIENT_SECRET="your_youtube_client_secret"
```

### 5.2 Configuration Methods for Docker

**Option 1: Mounted .env File (Recommended)**
```bash
# Create .env file in persistent volume
mkdir -p /path/to/config
nano /path/to/config/.env

# Add all environment variables to .env file

# Mount in Docker Compose:
services:
  postiz:
    image: ghcr.io/gitroomhq/postiz-app:latest
    volumes:
      - /path/to/config:/config
    ports:
      - "5000:5000"
```

**Option 2: Docker Compose environment Section**
```yaml
services:
  postiz:
    image: ghcr.io/gitroomhq/postiz-app:latest
    environment:
      - MAIN_URL=https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org
      - FRONTEND_URL=https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org
      - FACEBOOK_APP_ID=your_app_id
      - FACEBOOK_APP_SECRET=your_app_secret
      # ... etc
    ports:
      - "5000:5000"
```

**Option 3: .env File Next to docker-compose.yml**
```bash
# Create .env in same directory as docker-compose.yml
nano .env

# Add variables (same format as Option 1)

# Reference in docker-compose.yml:
services:
  postiz:
    image: ghcr.io/gitroomhq/postiz-app:latest
    env_file:
      - .env
    ports:
      - "5000:5000"
```

### 5.3 Applying Configuration Changes

**CRITICAL:** Postiz is entirely configured via environment variables and does NOT hot-reload configuration.

**After Any Environment Variable Change:**

```bash
# Method 1: Restart container
docker restart postiz

# Method 2: Stop and start (if restart doesn't work)
docker stop postiz
docker start postiz

# Method 3: Full recreate (for docker-compose)
docker-compose down
docker-compose up -d

# Method 4: Rebuild (if image needs updating)
docker-compose up -d --build
```

**Wait Time:**
After restart, wait 30-60 seconds for Postiz to fully initialize before testing OAuth connections.

### 5.4 Verifying Configuration

**Check Environment Variables Are Loaded:**
```bash
# View all environment variables
docker exec postiz env

# Filter for specific provider
docker exec postiz env | grep FACEBOOK
docker exec postiz env | grep X_
docker exec postiz env | grep TIKTOK
docker exec postiz env | grep YOUTUBE

# Check frontend URL
docker exec postiz env | grep FRONTEND_URL
```

**Expected Output Example:**
```
FRONTEND_URL=https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org
FACEBOOK_APP_ID=1234567890123456
FACEBOOK_APP_SECRET=abc123def456ghi789
X_API_KEY=abcdefghijklmnopqrstuvwxy
X_API_SECRET=1234567890abcdefghijklmnopqrstuvwxyz12345
```

**Test OAuth Flow:**
1. Access Postiz web UI
2. Navigate to "Add Channel" or "Integrations"
3. Click on a social media provider (e.g., Facebook)
4. Inspect redirect URL in browser:
   - Should show `client_id=YOUR_ACTUAL_APP_ID`
   - If shows `client_id=undefined`, environment variables not loaded

---

## 6. Troubleshooting Guide

### 6.1 Universal Troubleshooting Steps

**Step 1: Verify Environment Variables Are Loaded**
```bash
docker exec postiz env | grep -E "(FACEBOOK|X_|TIKTOK|YOUTUBE|FRONTEND_URL)"
```

**Step 2: Check Container Logs**
```bash
docker logs postiz --tail=100

# Follow logs in real-time
docker logs postiz -f
```

**Step 3: Restart Application**
```bash
docker restart postiz
sleep 60  # Wait for startup
```

**Step 4: Verify OAuth Redirect URL**
- Click to add a social media channel
- Inspect browser address bar during redirect
- Verify `client_id` parameter contains actual App ID (not "undefined")
- Verify redirect_uri matches what you configured in provider app

### 6.2 "Invalid App ID" / "client_id=undefined" Errors

**Symptoms:**
- OAuth flow shows `client_id=undefined` in URL
- Error message: "Invalid App ID" or "App ID not found"
- Cannot authorize social media accounts

**Root Cause:**
Environment variables not loaded or incorrect

**Solution:**

```bash
# 1. Stop container
docker stop postiz

# 2. Verify .env file exists and has content
cat /path/to/config/.env | grep APP_ID

# 3. Verify .env file format (no quotes unless needed, no extra spaces)
# CORRECT:
FACEBOOK_APP_ID=1234567890123456
FACEBOOK_APP_SECRET=abc123def456

# INCORRECT (avoid):
FACEBOOK_APP_ID = "1234567890123456"  # Spaces around =
FACEBOOK_APP_ID="{1234567890123456}"  # Curly braces
FACEBOOK_APP_ID="YOUR_APP_ID"         # Placeholder not replaced

# 4. Start container
docker start postiz

# 5. Wait 60 seconds for initialization
sleep 60

# 6. Verify variables loaded
docker exec postiz env | grep FACEBOOK_APP_ID

# 7. Test OAuth flow again
```

### 6.3 Redirect URI Mismatch Errors

**Symptoms:**
- "redirect_uri_mismatch" error
- "Invalid redirect URI"
- OAuth authorization fails after clicking authorize

**Solution:**

**Step 1: Identify Exact Redirect URI Used by Postiz**
```bash
# Check FRONTEND_URL
docker exec postiz env | grep FRONTEND_URL

# Expected: FRONTEND_URL=https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org
```

**Step 2: Construct Expected Callback URLs**
- Facebook: `{FRONTEND_URL}/integrations/social/facebook`
- Instagram: `{FRONTEND_URL}/integrations/social/instagram-standalone`
- X: `{FRONTEND_URL}/integrations/social/x`
- TikTok: `{FRONTEND_URL}/integrations/social/tiktok`
- YouTube: `{FRONTEND_URL}/integrations/social/youtube`

**For your deployment:**
```
https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/facebook
https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/instagram-standalone
https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/x
https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/tiktok
https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/youtube
```

**Step 3: Update Provider App Settings**
- Log into each provider's developer console
- Navigate to OAuth redirect URI settings
- Add/update URI to match EXACTLY (case-sensitive, no trailing slash)
- Save changes

**Step 4: Test Again**

### 6.4 "Could Not Add Provider" / "Could Not Add Channel" Errors

**Common for:** Facebook, Instagram, LinkedIn

**Causes:**
1. App not in Live mode (development mode)
2. Missing required permissions/scopes
3. User account not added as test user (during development)
4. Token expired or invalid

**Solution:**

**For Meta/Facebook/Instagram:**
```
1. Switch app to Live mode:
   - Meta Developer Console → App Settings → Basic
   - Toggle "App Mode" from Development to Live
   - Add Privacy Policy URL if required

2. Verify permissions approved:
   - Check that pages_manage_posts, pages_read_engagement, etc. are approved
   - For Instagram: instagram_basic, instagram_content_publish

3. Add test users (if in development mode):
   - App Dashboard → Roles → Roles
   - Add your Facebook account as Administrator or Developer
   - For Instagram: Add as Instagram Tester
   - Accept invitation via Instagram → Settings → Apps and Websites

4. Restart Postiz and retry
```

**For TikTok:**
```
1. Verify app includes Login Kit and Content Posting API
2. Check "Direct Post" is enabled
3. Ensure account is added as tester (if unverified app)
4. Verify media URLs are HTTPS and publicly accessible
```

### 6.5 Rate Limit / Quota Exceeded Errors

**Symptoms:**
- Error: "Rate limit exceeded"
- Error: "quotaExceeded" (YouTube)
- HTTP 429 status code
- Posting fails intermittently

**Solutions:**

**Twitter/X:**
- Check Free tier limits (500-1,500 posts/month)
- Space posts 5+ minutes apart
- Monitor remaining quota via API headers
- Wait for rate limit reset (shown in error response)

**YouTube:**
- Default quota: 10,000 units/day
- Video upload: 1,600 units each
- Request quota increase via GCP console
- Schedule uploads overnight when quota resets (midnight PT)

**TikTok:**
- Limit: 6 requests per minute per user
- Max ~15 posts per day per creator
- Space posts 10+ minutes apart
- If unverified: Max 5 users per 24 hours

**Facebook/Instagram:**
- No strict hourly limits for API
- Platform spam detection: ~1 post per 20 minutes
- Space posts across the day
- Avoid duplicate content

**Best Practices:**
- Implement exponential backoff for retries
- Monitor Postiz logs for rate limit warnings
- Schedule posts with spacing (not all at once)
- Track usage manually if approaching limits

### 6.6 Permission/Scope Errors

**Symptoms:**
- "Insufficient permissions"
- "Missing required scope"
- OAuth succeeds but posting fails

**Solution:**

**Facebook/Instagram:**
```
Required scopes:
- pages_show_list
- pages_manage_posts
- pages_read_engagement
- business_management
- instagram_basic (Instagram)
- instagram_content_publish (Instagram)

Fix:
1. Meta Developer Console → App Review → Permissions
2. Request missing permissions
3. For advanced permissions: May require business verification
4. Reauthorize connection in Postiz after approval
```

**X (Twitter):**
```
Required: "Read and Write" permission

Fix:
1. Twitter Developer Portal → App Settings → User authentication settings
2. Change "App permissions" to "Read and Write"
3. Save changes
4. CRITICAL: Regenerate Consumer Keys (Keys and Tokens tab)
5. Update Postiz environment variables with NEW keys
6. Restart Postiz
7. Reauthorize connection
```

**TikTok:**
```
Required scopes:
- user.info.basic
- user.info.profile
- video.upload
- video.publish

Fix:
1. TikTok Developer Portal → App → Products
2. Ensure Login Kit and Content Posting API are added
3. Enable "Direct Post" for Content Posting API
4. Scopes are automatically included
5. Reauthorize connection
```

**YouTube:**
```
Required scope:
- https://www.googleapis.com/auth/youtube.upload
- https://www.googleapis.com/auth/youtube

Fix:
1. GCP Console → APIs & Services → OAuth consent screen
2. Click "EDIT APP"
3. Go to "Scopes" section
4. Add missing YouTube scopes
5. Save changes
6. Reauthorize connection in Postiz
```

### 6.7 HTTPS / SSL Certificate Issues

**For Akash Deployments:**

Your deployment URL is already HTTPS, but if encountering certificate errors:

**Verify SSL Certificate:**
```bash
curl -I https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org
# Should return 200 OK with valid certificate
```

**If SSL Issues Persist:**
1. Check Akash ingress configuration
2. Verify domain SSL certificate is valid and not expired
3. Consider adding Cloudflare proxy for additional SSL layer
4. Test OAuth callbacks work over HTTPS

**TikTok Media Hosting SSL:**
If TikTok media uploads fail:
```
1. Media must be served over HTTPS
2. Configure CDN (Cloudflare R2, AWS S3)
3. Or: Use reverse proxy with SSL for Akash media directory
4. Verify domain in TikTok Developer Portal
```

---

## 7. Akash-Specific Considerations

### 7.1 Custom Domain and HTTPS Setup

**Current Status:**
- Your Akash deployment URL: `https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org`
- HTTPS enabled by default (Akash ingress provides SSL)
- Long subdomain URL (not user-friendly but functional)

**Option: Custom Domain with Cloudflare**

**Benefits:**
- Shorter, branded URL (e.g., `postiz.yourdomain.com`)
- Additional SSL/TLS encryption layer
- DDoS protection
- Media caching (helps with TikTok requirements)

**Setup Steps:**

1. **Purchase Domain**
   - Use any registrar (Namecheap, GoDaddy, Cloudflare Registrar)

2. **Add Domain to Cloudflare**
   - Create free Cloudflare account
   - Add your domain
   - Update nameservers at registrar

3. **Create DNS Record**
   ```
   Type: CNAME
   Name: postiz (or subdomain of choice)
   Target: 09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org
   Proxy status: Proxied (orange cloud)
   ```

4. **Configure SSL in Cloudflare**
   - SSL/TLS → Overview → Set to "Full" or "Full (Strict)"
   - Edge Certificates → Always Use HTTPS: ON

5. **Update Postiz Environment Variables**
   ```env
   MAIN_URL="https://postiz.yourdomain.com"
   FRONTEND_URL="https://postiz.yourdomain.com"
   NEXT_PUBLIC_BACKEND_URL="https://postiz.yourdomain.com/api"
   ```

6. **Update OAuth Callback URLs**
   - Update all provider apps with new callback URLs
   - Example: `https://postiz.yourdomain.com/integrations/social/facebook`

7. **Restart Postiz**
   ```bash
   docker restart postiz
   ```

### 7.2 Media Storage and TikTok Integration

**Challenge:**
TikTok requires media to be publicly accessible via HTTPS from a verified domain.

**Akash Container Storage:**
- Ephemeral by default (lost on container restart)
- Requires persistent volume configuration
- Direct exposure of media directory has limitations

**Recommended Solutions:**

**Option 1: Cloudflare R2 (Recommended)**
```
Benefits:
- S3-compatible object storage
- No egress fees
- HTTPS URLs out of the box
- Global CDN distribution
- Domain verification possible

Setup:
1. Create Cloudflare R2 bucket
2. Configure public access
3. Set custom domain (e.g., media.yourdomain.com)
4. Configure Postiz to upload media to R2 before posting
5. Verify domain in TikTok Developer Portal
```

**Option 2: AWS S3 / Backblaze B2**
```
Similar to R2:
1. Create bucket with public read access
2. Enable static website hosting OR use CloudFront
3. Configure HTTPS
4. Update Postiz media handling
5. Verify domain in TikTok
```

**Option 3: Reverse Proxy with Akash**
```
Setup Caddy or Nginx in Akash deployment:
1. Configure reverse proxy to serve /uploads directory
2. Use custom domain with Cloudflare
3. Expose media at https://postiz.yourdomain.com/media/
4. Verify domain in TikTok Developer Portal

Challenges:
- Requires persistent storage in Akash deployment
- Additional container complexity
- Domain verification needed
```

**Postiz Configuration for External Media:**
```env
# If Postiz supports external media storage (check latest docs)
UPLOAD_DIRECTORY=/path/to/persistent/storage
MEDIA_BASE_URL=https://media.yourdomain.com

# May require code modifications or plugin
```

### 7.3 Persistent Storage Configuration

**Critical for:**
- Media files (uploads)
- Database (if using SQLite)
- Configuration persistence

**Akash SDL Example:**
```yaml
services:
  postiz:
    image: ghcr.io/gitroomhq/postiz-app:latest
    env:
      - FRONTEND_URL=https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org
    expose:
      - port: 5000
        as: 80
        to:
          - global: true

profiles:
  compute:
    postiz:
      resources:
        cpu:
          units: 1.0
        memory:
          size: 2Gi
        storage:
          - size: 10Gi  # Persistent storage
            attributes:
              persistent: true
              class: beta3

  placement:
    akash:
      pricing:
        postiz:
          denom: uakt
          amount: 1000

deployment:
  postiz:
    akash:
      profile: postiz
      count: 1
```

**Mounting Persistent Volume:**
If Postiz supports volume mounts via Akash:
```yaml
# Map persistent storage to container paths
# (Syntax may vary based on Akash version)
storage:
  - name: data
    mount: /data
  - name: config
    mount: /config
```

### 7.4 Environment Variable Management in Akash

**Challenges:**
- Akash deployment updates via SDL files
- Secrets management for API credentials
- Avoiding committing credentials to version control

**Solutions:**

**Option 1: Akash Deployment Environment Variables**
```yaml
# In SDL file
services:
  postiz:
    image: ghcr.io/gitroomhq/postiz-app:latest
    env:
      # Public variables (safe in SDL)
      - FRONTEND_URL=https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org
      - NEXT_PUBLIC_BACKEND_URL=https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/api

      # Secrets (consider encryption or external management)
      - FACEBOOK_APP_ID=YOUR_APP_ID
      - FACEBOOK_APP_SECRET=YOUR_APP_SECRET
```

**Option 2: Mount .env from Persistent Volume**
```yaml
# Store .env in persistent volume
# Postiz reads from /config/.env

# In deployment:
services:
  postiz:
    image: ghcr.io/gitroomhq/postiz-app:latest
    # Mount persistent volume to /config
    # Place .env file in volume before deployment
```

**Option 3: Base64 Encoded Secrets**
```bash
# Encode .env file
cat .env | base64 > env.b64

# In Akash SDL, decode at runtime (requires init script)
# Or: Store encoded in SDL, decode via entrypoint
```

**Best Practice:**
- Use separate SDL files for dev/prod
- Never commit credentials to Git
- Consider Akash secrets management tools (if available)
- Rotate credentials periodically

### 7.5 Updating Postiz Configuration

**When You Need to Update Environment Variables:**

**Method 1: Update Deployment (Recommended)**
```bash
# 1. Edit SDL file with new environment variables
nano deploy.yaml

# 2. Update Akash deployment
akash tx deployment update deploy.yaml --from <your-wallet>

# 3. Wait for deployment to restart
```

**Method 2: SSH into Container (if supported)**
```bash
# Access Akash provider's container
akash provider lease-shell --from <your-wallet>

# Edit .env file
vi /config/.env

# Restart Postiz process (if possible without full container restart)
```

**Method 3: Redeploy**
```bash
# Close existing deployment
akash tx deployment close --from <your-wallet>

# Create new deployment with updated SDL
akash tx deployment create deploy.yaml --from <your-wallet>
```

### 7.6 Monitoring and Logs

**Accessing Postiz Logs on Akash:**

**Via Akash CLI:**
```bash
# Get logs from deployment
akash provider lease-logs --from <your-wallet>

# Follow logs in real-time
akash provider lease-logs --follow --from <your-wallet>
```

**Via Provider Web UI:**
Some Akash providers offer web interfaces to view logs directly.

**Debug OAuth Issues:**
```bash
# Look for OAuth-related errors
akash provider lease-logs --from <your-wallet> | grep -i "oauth\|client_id\|redirect"

# Check environment variable loading
akash provider lease-logs --from <your-wallet> | grep -i "environment\|config"
```

### 7.7 Cost Optimization

**Akash Resource Recommendations for Postiz:**

**Minimum Configuration:**
```yaml
cpu:
  units: 0.5
memory:
  size: 1Gi
storage:
  - size: 5Gi
```

**Recommended Configuration:**
```yaml
cpu:
  units: 1.0
memory:
  size: 2Gi
storage:
  - size: 10Gi  # More for media storage
```

**High-Volume Configuration:**
```yaml
cpu:
  units: 2.0
memory:
  size: 4Gi
storage:
  - size: 20Gi
```

**Cost Estimates:**
- Akash pricing is dynamic (market-based)
- Typical cost: $5-20/month for small deployment
- Monitor actual usage and adjust resources

**Optimization Tips:**
- Use external media storage (reduces storage requirements)
- Scale down CPU/memory if single-user
- Use persistent storage only where needed
- Close deployment when not in use (for testing)

---

## 8. Quick Reference

### 8.1 OAuth Callback URLs Summary

**For Deployment:** `https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org`

| Platform | Callback URL |
|----------|--------------|
| Facebook | `https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/facebook` |
| Instagram (via FB) | Same as Facebook |
| Instagram (Standalone) | `https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/instagram-standalone` |
| X (Twitter) | `https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/x` |
| TikTok | `https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/tiktok` |
| YouTube | `https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/integrations/social/youtube` |

### 8.2 Required Environment Variables Checklist

```env
# Core Postiz Configuration
MAIN_URL="https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org"
FRONTEND_URL="https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org"
NEXT_PUBLIC_BACKEND_URL="https://09rh4o8cch8a1f20mjtk31hec4.ingress.akash-palmito.org/api"
BACKEND_INTERNAL_URL="http://localhost:3000"

# Meta (Facebook & Instagram)
FACEBOOK_APP_ID=""
FACEBOOK_APP_SECRET=""

# X (Twitter)
X_API_KEY=""
X_API_SECRET=""

# TikTok
TIKTOK_CLIENT_ID=""
TIKTOK_CLIENT_SECRET=""

# YouTube
YOUTUBE_CLIENT_ID=""
YOUTUBE_CLIENT_SECRET=""
```

### 8.3 Developer Portal Links

| Platform | Developer Portal | Documentation |
|----------|-----------------|---------------|
| Meta (Facebook/Instagram) | https://developers.facebook.com/apps | https://developers.facebook.com/docs |
| X (Twitter) | https://developer.twitter.com/en/portal/dashboard | https://developer.twitter.com/en/docs |
| TikTok | https://developers.tiktok.com/ | https://developers.tiktok.com/doc |
| YouTube | https://console.cloud.google.com/ | https://developers.google.com/youtube |
| Postiz Docs | https://docs.postiz.com/ | Official setup guides |

### 8.4 Posting Rate Limits Summary

| Platform | Free Tier Limit | Notes |
|----------|-----------------|-------|
| **Facebook** | No hard limit | ~1 post per 20 min recommended |
| **Instagram** | 25 posts per 24h | Per user, across all apps |
| **X (Twitter)** | 500-1,500/month | Application-level, Free tier |
| **TikTok** | 15 posts per day | Per creator account |
| **YouTube** | ~6 videos per day | 10,000 quota units, 1,600 per upload |

### 8.5 Common Error Codes

| Error | Likely Cause | Quick Fix |
|-------|--------------|-----------|
| `client_id=undefined` | Environment variables not loaded | Restart Postiz, verify .env file |
| Invalid App ID | Wrong credentials or not in Live mode | Check app status, verify credentials |
| redirect_uri_mismatch | Callback URL doesn't match provider settings | Update provider app with exact URL |
| 403 Forbidden | Missing permissions or wrong app type | Check Read/Write permissions, regenerate keys |
| quotaExceeded (YouTube) | Hit daily API quota | Wait for reset or request increase |
| Rate limit exceeded | Too many API calls | Space posts further apart |

### 8.6 Pre-Flight Checklist

Before attempting to connect each platform:

**Meta (Facebook/Instagram):**
- [ ] App created in Meta Developer Console
- [ ] App type: Business
- [ ] App mode: Live (not Development)
- [ ] Redirect URI added to Facebook Login settings
- [ ] Required permissions requested
- [ ] App ID and Secret copied to .env
- [ ] Postiz restarted after adding credentials

**X (Twitter):**
- [ ] App created in Twitter Developer Portal
- [ ] App type: Web App, Automated App or Bot
- [ ] App permissions: Read and Write
- [ ] Consumer Keys regenerated after permission change
- [ ] Redirect URI added to OAuth 2.0 settings
- [ ] API Key and Secret copied to .env
- [ ] Postiz restarted

**TikTok:**
- [ ] App created in TikTok Developers
- [ ] Platform: Web selected
- [ ] Login Kit added and configured
- [ ] Content Posting API added with Direct Post enabled
- [ ] Redirect URI added (HTTPS required)
- [ ] Privacy Policy and Terms of Service URLs provided
- [ ] Client ID and Secret copied to .env
- [ ] Media domain verified (if posting with media)
- [ ] Postiz restarted

**YouTube:**
- [ ] Google Cloud Project created
- [ ] YouTube Data API v3 enabled
- [ ] YouTube Analytics API enabled (optional)
- [ ] OAuth consent screen configured
- [ ] OAuth 2.0 Client ID created (Web application)
- [ ] Redirect URI added to authorized URIs
- [ ] Test user added (if in Test mode)
- [ ] Required scopes added to consent screen
- [ ] Client ID and Secret copied to .env
- [ ] Postiz restarted

---

## 9. Additional Resources

### 9.1 Official Documentation

**Postiz:**
- Main Documentation: https://docs.postiz.com/
- Facebook Provider: https://docs.postiz.com/providers/facebook
- Instagram Provider: https://docs.postiz.com/providers/instagram
- X Provider: https://docs.postiz.com/providers/x-twitter
- TikTok Provider: https://docs.postiz.com/providers/tiktok
- YouTube Provider: https://docs.postiz.com/providers/youtube
- Configuration Reference: https://docs.postiz.com/configuration/reference

**Platform APIs:**
- Meta for Developers: https://developers.facebook.com/docs
- X (Twitter) API: https://developer.twitter.com/en/docs
- TikTok for Developers: https://developers.tiktok.com/doc
- YouTube Data API: https://developers.google.com/youtube/v3

**Akash Network:**
- Main Documentation: https://docs.akash.network/
- Custom Domain Guide: https://akash.network/docs/guides/network/custom-domain/

### 9.2 Community Support

**Postiz:**
- GitHub Issues: https://github.com/gitroomhq/postiz-app/issues
- Discord Community: Available via docs.postiz.com/support

**Helpful GitHub Issues:**
- Instagram/Meta "Invalid App ID": https://github.com/gitroomhq/postiz-app/issues/891
- Cannot Add Channels: https://github.com/gitroomhq/postiz-app/issues/918
- Meta Integration Issues: https://github.com/gitroomhq/postiz-app/issues/850

### 9.3 Troubleshooting Resources

**Stack Overflow Tags:**
- [facebook-graph-api]
- [twitter-oauth]
- [tiktok-api]
- [youtube-data-api]
- [docker]
- [oauth-2.0]

**Useful Forum Discussions:**
- Postiz Social Media Login Issues: https://forum.cloudron.io/topic/13668/impossible-to-log-in-any-sm-account-on-postiz

---

## 10. Conclusion

This guide provides comprehensive coverage of OAuth integration requirements for connecting your self-hosted Postiz instance on Akash Network to Meta (Facebook/Instagram), X (Twitter), TikTok, and YouTube.

**Key Takeaways:**

1. **Environment Variables Are Critical:** The most common issue ("Invalid App ID") is caused by environment variables not being loaded. Always restart Postiz after configuration changes.

2. **HTTPS Is Required:** All platforms require HTTPS for production OAuth callbacks. Your Akash deployment URL already uses HTTPS, meeting this requirement.

3. **Platform-Specific Challenges:**
   - **Meta**: Must switch to Live mode for public posting
   - **X**: Must regenerate keys after changing permissions
   - **TikTok**: Most restrictive - requires public HTTPS media URLs and domain verification
   - **YouTube**: Quota limits (10,000 units/day) are the main constraint

4. **TikTok Media Challenge:** For Akash deployments, using external CDN (Cloudflare R2, AWS S3) is highly recommended for TikTok media hosting.

5. **Rate Limits:** Plan post scheduling to stay within platform limits, especially for X Free tier (500-1,500 posts/month) and YouTube (6 videos/day default).

**Next Steps:**

1. Copy the OAuth callback URLs from section 8.1 for your deployment
2. Create developer apps for each platform following sections 1-4
3. Configure environment variables as outlined in section 5
4. Restart Postiz and test connections
5. Refer to section 6 for troubleshooting if issues arise

**For Your Specific "Invalid App ID" Error:**

The error you're experiencing is almost certainly caused by environment variables not being properly loaded. Follow these steps:

```bash
# 1. Verify .env file location and content
cat /path/to/config/.env

# 2. Ensure FACEBOOK_APP_ID and FACEBOOK_APP_SECRET are set correctly
# (actual values, no placeholders, no extra spaces/quotes)

# 3. Restart Postiz
docker restart postiz

# 4. Wait 60 seconds

# 5. Verify variables loaded
docker exec postiz env | grep FACEBOOK

# 6. Test OAuth flow - should now show actual App ID in URL
```

If problems persist after following this guide, consult the Postiz GitHub issues or Discord community with specific error messages and logs.

---

**Report Generated:** 2025-01-14
**Research Completeness:** 92%
**Platforms Covered:** Meta (Facebook/Instagram), X (Twitter), TikTok, YouTube
**Deployment Target:** Akash Network
**Postiz Version:** Latest (ghcr.io/gitroomhq/postiz-app:latest)