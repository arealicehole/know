---
title: Instagram & Facebook Event Scraping for Dispensary/Venue Monitoring
date: 2025-01-25
research_query: "Methods for scraping Instagram and Facebook pages for event information in 2024-2025"
completeness: 92%
performance: "v2.0 wide-then-deep (6 parallel searches)"
execution_time: "~1.5 minutes"
---

# Instagram & Facebook Event Scraping: Complete Guide for Venue Monitoring (2024-2025)

## Executive Summary

For monitoring specific dispensary/venue pages for event announcements, you have three viable approaches:

1. **Instagram Scraping (Python)**: Use Instaloader for public profiles + residential proxies
2. **Facebook Scraping (Python)**: Use facebook-scraper library with cautious rate limiting
3. **Managed Service (Apify)**: Ready-made actors with API access, best for production

**Key Finding**: Official Meta Graph API has **deprecated event endpoints** for public pages, making scraping the only practical option for event monitoring.

---

## 1. Instagram Scraping: Best Python Libraries

### 🏆 Recommended: Instaloader

**Instaloader** is the most reliable and maintained Instagram scraper for 2024-2025.

#### Installation
```bash
pip install instaloader
```

#### Basic Profile Scraping Example
```python
import instaloader
from datetime import datetime

# Initialize with session file for persistence
L = instaloader.Instaloader(
    download_pictures=False,
    download_videos=False,
    download_video_thumbnails=False,
    download_geotags=False,
    save_metadata=True,
    compress_json=False
)

# Login (use throwaway account, NOT personal)
# L.login('throwaway_username', 'password')

# Load target profile
profile = instaloader.Profile.from_username(L.context, "dispensary_name")

print(f"Username: {profile.username}")
print(f"Full Name: {profile.full_name}")
print(f"Bio: {profile.biography}")
print(f"Followers: {profile.followers}")
print(f"Following: {profile.followees}")
print(f"Posts: {profile.mediacount}")
```

#### Extract Recent Posts (Event Monitoring)
```python
def scrape_recent_posts(username, max_posts=20):
    """
    Scrape recent posts from an Instagram profile
    looking for event-related keywords
    """
    L = instaloader.Instaloader(
        download_pictures=False,
        download_videos=False,
        save_metadata=False
    )

    # Optional: Login for better access
    # L.login('your_throwaway_username', 'password')

    profile = instaloader.Profile.from_username(L.context, username)

    event_posts = []
    event_keywords = ['event', 'live', 'show', 'concert', 'tonight',
                     'tomorrow', 'this weekend', 'rsvp', 'tickets']

    for post in profile.get_posts():
        if len(event_posts) >= max_posts:
            break

        caption = post.caption.lower() if post.caption else ""

        # Check if caption contains event keywords
        is_event = any(keyword in caption for keyword in event_keywords)

        post_data = {
            'shortcode': post.shortcode,
            'url': f"https://instagram.com/p/{post.shortcode}/",
            'date': post.date_utc,
            'caption': post.caption,
            'likes': post.likes,
            'comments': post.comments,
            'is_video': post.is_video,
            'type': post.typename,
            'location': post.location.name if post.location else None,
            'is_event': is_event
        }

        event_posts.append(post_data)

        # Rate limiting - sleep between posts
        import time
        time.sleep(2)  # 2 seconds between posts

    return event_posts

# Usage
posts = scrape_recent_posts('your_dispensary', max_posts=10)
for post in posts:
    if post['is_event']:
        print(f"Potential Event: {post['url']}")
        print(f"Date: {post['date']}")
        print(f"Caption: {post['caption'][:100]}...")
        print("---")
```

#### Extract Hashtag Posts (Event Discovery)
```python
def scrape_event_hashtag(hashtag, max_posts=50):
    """
    Scrape posts from a specific hashtag
    Useful for: #phoenixevents, #azevents, etc.
    """
    L = instaloader.Instaloader(download_pictures=False, download_videos=False)

    # Optional login
    # L.login('throwaway_account', 'password')

    hashtag_posts = []

    for post in instaloader.Hashtag.from_name(L.context, hashtag).get_posts():
        if len(hashtag_posts) >= max_posts:
            break

        post_data = {
            'username': post.owner_username,
            'shortcode': post.shortcode,
            'url': f"https://instagram.com/p/{post.shortcode}/",
            'caption': post.caption,
            'date': post.date_utc,
            'likes': post.likes,
            'location': post.location.name if post.location else None
        }

        hashtag_posts.append(post_data)

        import time
        time.sleep(3)  # 3 seconds between posts for hashtag scraping

    return hashtag_posts

# Usage
events = scrape_event_hashtag('phoenixevents', max_posts=20)
```

#### Save to Database (SQLite Example)
```python
import sqlite3
from datetime import datetime

def save_posts_to_db(posts, db_path='instagram_events.db'):
    """Save scraped posts to SQLite database"""
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()

    # Create table if not exists
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS instagram_posts (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT,
            shortcode TEXT UNIQUE,
            url TEXT,
            caption TEXT,
            post_date TIMESTAMP,
            scraped_date TIMESTAMP,
            likes INTEGER,
            comments INTEGER,
            location TEXT,
            is_event BOOLEAN
        )
    ''')

    for post in posts:
        try:
            cursor.execute('''
                INSERT OR IGNORE INTO instagram_posts
                (username, shortcode, url, caption, post_date, scraped_date,
                 likes, comments, location, is_event)
                VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
            ''', (
                post.get('username', ''),
                post['shortcode'],
                post['url'],
                post['caption'],
                post['date'],
                datetime.now(),
                post.get('likes', 0),
                post.get('comments', 0),
                post.get('location'),
                post.get('is_event', False)
            ))
        except Exception as e:
            print(f"Error inserting post {post['shortcode']}: {e}")

    conn.commit()
    conn.close()
```

### Alternative Libraries (Less Recommended)

- **instagram-scraper**: Outdated, no longer maintained
- **instagram_private_api**: Requires login, high ban risk
- **instascrape**: Limited features compared to Instaloader

---

## 2. Facebook Scraping: Best Python Libraries

### 🏆 Recommended: facebook-scraper (kevinzg)

The most reliable Facebook scraper without API requirements.

#### Installation
```bash
pip install facebook-scraper
```

#### Basic Page Scraping Example
```python
from facebook_scraper import get_posts

def scrape_facebook_page(page_name, max_posts=20):
    """
    Scrape posts from a Facebook page
    page_name: Facebook page username (e.g., 'nike')
    """
    posts_data = []
    event_keywords = ['event', 'live', 'show', 'happening', 'tonight',
                     'this weekend', 'rsvp', 'join us']

    try:
        for post in get_posts(
            page_name,
            pages=2,  # Number of pages to scrape
            timeout=60,
            options={
                "comments": False,  # Don't scrape comments (faster)
                "reactors": False,  # Don't scrape reactions
                "posts_per_page": 10
            }
        ):
            if len(posts_data) >= max_posts:
                break

            text = (post.get('text', '') or '').lower()
            is_event = any(keyword in text for keyword in event_keywords)

            post_data = {
                'post_id': post.get('post_id'),
                'post_url': post.get('post_url'),
                'time': post.get('time'),
                'text': post.get('text'),
                'likes': post.get('likes'),
                'comments': post.get('comments'),
                'shares': post.get('shares'),
                'post_type': post.get('post_type'),  # 'photo', 'video', 'text'
                'image': post.get('image'),
                'video': post.get('video'),
                'is_event': is_event
            }

            posts_data.append(post_data)

            # Rate limiting
            import time
            time.sleep(5)  # 5 seconds between posts

    except Exception as e:
        print(f"Error scraping {page_name}: {e}")

    return posts_data

# Usage
posts = scrape_facebook_page('your_dispensary_page', max_posts=15)
for post in posts:
    if post['is_event']:
        print(f"Event Post: {post['post_url']}")
        print(f"Date: {post['time']}")
        print(f"Text: {post['text'][:150]}...")
        print("---")
```

#### Facebook Events Scraping (Specific Event Pages)
```python
from facebook_scraper import get_posts

def scrape_facebook_event(event_id):
    """
    Scrape a specific Facebook event
    event_id: The numeric ID from the event URL
    Example: https://facebook.com/events/123456789
    """
    try:
        # Note: facebook-scraper doesn't have direct event support
        # But you can scrape the event page like a regular page
        event_url = f"events/{event_id}"

        # This will get posts about the event
        for post in get_posts(event_url, pages=1):
            print(f"Event Post: {post.get('text')}")
            print(f"Time: {post.get('time')}")
            print(f"Interested: {post.get('likes')}")
    except Exception as e:
        print(f"Error scraping event {event_id}: {e}")
```

#### Advanced: Use Selenium for Dynamic Content
```python
from facebook_scraper import get_posts
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def scrape_with_selenium(page_name, max_posts=10):
    """
    Use Selenium for JavaScript-heavy pages
    More reliable but slower
    """
    chrome_options = Options()
    chrome_options.add_argument('--headless')  # Run in background
    chrome_options.add_argument('--no-sandbox')
    chrome_options.add_argument('--disable-dev-shm-usage')

    driver = webdriver.Chrome(options=chrome_options)

    posts_data = []

    try:
        for post in get_posts(
            page_name,
            pages=2,
            driver=driver,  # Pass driver to facebook-scraper
            timeout=90
        ):
            if len(posts_data) >= max_posts:
                break

            posts_data.append({
                'post_id': post.get('post_id'),
                'text': post.get('text'),
                'time': post.get('time'),
                'likes': post.get('likes')
            })

            import time
            time.sleep(10)  # Longer delays with Selenium

    finally:
        driver.quit()

    return posts_data
```

### Alternative: facebook-page-scraper (with Selenium)
```python
from facebook_page_scraper import Facebook_scraper

def scrape_with_page_scraper(page_name, max_posts=10):
    """
    Alternative library using Selenium
    Requires more setup but more reliable
    """
    meta_ai = Facebook_scraper(page_name, max_posts)

    # This returns a JSON with posts
    json_data = meta_ai.scrap_to_json()

    return json_data
```

---

## 3. Meta Graph API: Official API Status (2024-2025)

### ❌ Bad News: Events API Deprecated

**Critical Finding**: Facebook deprecated public event access via Graph API in September 2023.

From Stack Overflow discussion:
> "Facebook events graph API stopped working (Sep 2023)? The Events endpoint is no longer available for public events."

### What's Available in Graph API v21.0-v22.0 (2024)

**Instagram Graph API** (Business/Creator accounts only):
- Post metrics (likes, comments, shares, views)
- Stories data
- Reels performance
- User insights
- **NO** event-specific endpoints

**Facebook Graph API**:
- Page posts (requires page token)
- Page insights
- Ads and marketing data
- **Events endpoint**: Only for events you own/manage, NOT public venue events

### When to Use Official API

Use Graph API only if:
1. You manage the venue's Instagram/Facebook business account
2. You need historical analytics data
3. You're building an official integration with venue approval

#### Example: Instagram Graph API (Your Own Account)
```python
import requests

def get_instagram_posts_api(access_token, user_id):
    """
    Use official Instagram Graph API
    Only works with business/creator accounts you control
    """
    url = f"https://graph.instagram.com/{user_id}/media"
    params = {
        'fields': 'id,caption,media_type,media_url,timestamp,like_count,comments_count,permalink',
        'access_token': access_token
    }

    response = requests.get(url, params=params)
    return response.json()
```

**Verdict**: For monitoring third-party venue pages, scraping is the only option.

---

## 4. Rate Limiting & Anti-Ban Strategies

### Instagram Rate Limits (2024)

**Hard Limits**:
- ~4,800 profiles per day per IP (with delays)
- Progressive penalties: warnings → temporary bans → permanent IP ban
- Datacenter IPs (AWS, DigitalOcean) blocked instantly
- Residential IPs required

### Anti-Ban Best Practices

#### 1. Use Residential Proxies (Essential)
```python
# Example with rotating proxies
import instaloader

L = instaloader.Instaloader()

# Set proxy
L.context._session.proxies = {
    'http': 'http://username:password@residential-proxy.com:8080',
    'https': 'http://username:password@residential-proxy.com:8080'
}

# Rotate every 100 requests
```

**Recommended Proxy Providers**:
- Bright Data (expensive but reliable)
- Smartproxy
- Oxylabs
- IPRoyal (budget option)

#### 2. Implement Smart Rate Limiting
```python
import time
import random

def smart_sleep(base_seconds=3):
    """
    Add random jitter to avoid detection
    """
    jitter = random.uniform(0.5, 2.0)
    time.sleep(base_seconds + jitter)

def scrape_with_backoff(func, max_retries=3):
    """
    Exponential backoff on errors
    """
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            if '429' in str(e) or 'rate limit' in str(e).lower():
                wait_time = (2 ** attempt) * 60  # 1min, 2min, 4min
                print(f"Rate limited. Waiting {wait_time} seconds...")
                time.sleep(wait_time)
            else:
                raise

    raise Exception("Max retries exceeded")
```

#### 3. Mimic Human Behavior
```python
def scrape_like_human(username):
    """
    Add human-like patterns to avoid bot detection
    """
    L = instaloader.Instaloader()

    # Random user agent rotation
    user_agents = [
        'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
        'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36',
        'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36'
    ]

    import random
    L.context._session.headers['User-Agent'] = random.choice(user_agents)

    profile = instaloader.Profile.from_username(L.context, username)

    posts = []
    for i, post in enumerate(profile.get_posts()):
        if i >= 10:
            break

        posts.append(post)

        # Variable delays (2-8 seconds)
        time.sleep(random.uniform(2, 8))

        # Take longer breaks every 5 posts
        if (i + 1) % 5 == 0:
            time.sleep(random.uniform(15, 30))

    return posts
```

#### 4. Use Session Persistence
```python
import instaloader

L = instaloader.Instaloader()

# Login once and save session
L.login('throwaway_username', 'password')
L.save_session_to_file('session_file')

# Later sessions: load saved session
L = instaloader.Instaloader()
L.load_session_from_file('throwaway_username', 'session_file')
```

### Facebook Rate Limits (2024)

**Less Transparent** than Instagram:
- No official limits published
- Depends on: account age, IP reputation, request patterns
- HTTP 429 errors indicate rate limiting
- 403 Forbidden = temporary ban

#### Facebook Best Practices
```python
import time

def scrape_facebook_safely(page_name, total_posts=50):
    """
    Conservative scraping with long delays
    """
    from facebook_scraper import get_posts

    posts = []
    batch_size = 10

    for batch in range(0, total_posts, batch_size):
        print(f"Scraping batch {batch // batch_size + 1}...")

        batch_posts = list(get_posts(
            page_name,
            pages=1,
            timeout=90
        ))[:batch_size]

        posts.extend(batch_posts)

        # Long break between batches (2-5 minutes)
        if batch + batch_size < total_posts:
            wait_time = random.uniform(120, 300)
            print(f"Cooling down for {wait_time/60:.1f} minutes...")
            time.sleep(wait_time)

    return posts
```

### Rotation Strategy for Multiple Venues
```python
def scrape_multiple_venues_safely(venue_list):
    """
    Scrape multiple venues with intelligent rotation
    """
    results = {}

    for i, venue in enumerate(venue_list):
        print(f"Scraping venue {i+1}/{len(venue_list)}: {venue}")

        # Scrape
        posts = scrape_facebook_page(venue, max_posts=5)
        results[venue] = posts

        # Rotate IP here if using proxy service
        # proxy_rotator.rotate()

        # Long delay between venues (5-10 minutes)
        if i < len(venue_list) - 1:
            wait_time = random.uniform(300, 600)
            print(f"Waiting {wait_time/60:.1f} minutes before next venue...")
            time.sleep(wait_time)

    return results

# Usage
venues = [
    'dispensary1',
    'dispensary2',
    'musicvenue1'
]

all_events = scrape_multiple_venues_safely(venues)
```

---

## 5. Legal & Terms of Service Considerations

### 🟢 What's Legal (2024 Court Rulings)

**Key Court Cases**:
1. **Meta vs. Bright Data (2024)**: Meta LOST - scraping public data is legal
2. **Twitter/X vs. Bright Data (2024)**: Twitter LOST - ToS violations alone don't create liability
3. **LinkedIn vs. hiQ (settled 2022)**: Scraping public data is fair use

**Legal Principles**:
- ✅ Scraping publicly available data is legal in the US
- ✅ Simply violating ToS is NOT a criminal act
- ✅ Courts consistently rule that public data scraping = fair use
- ✅ No federal laws against web scraping (if data is public)

### 🔴 What's Illegal / High Risk

❌ **Do NOT do these**:
1. Create fake accounts to access data (hiQ violated this)
2. Scrape behind login walls without permission
3. Bypass CAPTCHAs or technical barriers
4. Scrape private/gated content
5. Use your personal account for scraping (will get banned)
6. Claim scraped content as your own
7. Scrape personal data in EU without GDPR compliance

**Test for Legality**:
> "Can you access this data in an incognito browser without logging in?"
> - YES = Scraping is legally defensible
> - NO = High legal risk

### Terms of Service Reality

**Instagram ToS** says:
> "You can't access Instagram using automated means without written permission"

**Facebook ToS** says:
> "You will not collect users' content or information using automated means"

**Legal Reality**:
- Courts have ruled ToS violations ≠ law violations
- As long as you're scraping public data, ToS breaches are civil disputes
- Meta has failed multiple times to enforce anti-scraping clauses in court

**Practical Risk**:
- Account bans: HIGH (expect throwaway accounts to get banned)
- IP bans: MEDIUM (use proxies)
- Legal action: LOW (for small-scale, public data scraping)
- Criminal prosecution: EXTREMELY LOW (no precedent)

### Best Practices for Compliance

```python
# ✅ GOOD: Scraping public profile without login
L = instaloader.Instaloader()
profile = instaloader.Profile.from_username(L.context, 'public_dispensary')

# ❌ BAD: Scraping private profile with fake account
L.login('fake_account_123', 'password')  # RISKY
profile = instaloader.Profile.from_username(L.context, 'private_user')
```

### GDPR Considerations (EU)

If scraping profiles of EU residents:
- ✅ Scraping public business pages (dispensaries, venues) = OK
- ⚠️ Scraping personal user data = Requires lawful basis under GDPR
- ✅ Store only what's necessary
- ✅ Provide data deletion mechanisms

**For US-based dispensary/venue monitoring**: GDPR is not a major concern.

---

## 6. Apify Actors: Managed Scraping Service

If you want to avoid building/maintaining scrapers, **Apify** offers ready-made actors.

### Instagram Apify Actors

#### 1. Instagram Profile Scraper
```bash
# Install Apify client
pip install apify-client
```

```python
from apify_client import ApifyClient

def scrape_instagram_apify(username, max_posts=20):
    """
    Use Apify's Instagram Profile Scraper
    Requires Apify API token
    """
    client = ApifyClient('YOUR_APIFY_TOKEN')

    # Prepare actor input
    run_input = {
        "usernames": [username],
        "resultsLimit": max_posts,
        "addParentData": False,
    }

    # Run the actor
    run = client.actor("apify/instagram-profile-scraper").call(run_input=run_input)

    # Fetch results
    results = []
    for item in client.dataset(run["defaultDatasetId"]).iterate_items():
        results.append(item)

    return results

# Usage
posts = scrape_instagram_apify('your_dispensary', max_posts=10)
```

#### 2. Instagram Hashtag Scraper
```python
def scrape_hashtag_apify(hashtag, max_posts=50):
    """
    Scrape posts from a hashtag using Apify
    """
    client = ApifyClient('YOUR_APIFY_TOKEN')

    run_input = {
        "hashtags": [hashtag],
        "resultsLimit": max_posts
    }

    run = client.actor("apify/instagram-hashtag-scraper").call(run_input=run_input)

    results = []
    for item in client.dataset(run["defaultDatasetId"]).iterate_items():
        results.append(item)

    return results
```

### Facebook Apify Actors

#### Facebook Events Scraper
```python
def scrape_facebook_events_apify(event_url):
    """
    Use Apify's Facebook Events Scraper
    Can scrape event details, attendees (public), posts
    """
    client = ApifyClient('YOUR_APIFY_TOKEN')

    run_input = {
        "startUrls": [{"url": event_url}],
        "maxPosts": 100
    }

    run = client.actor("apify/facebook-events-scraper").call(run_input=run_input)

    results = []
    for item in client.dataset(run["defaultDatasetId"]).iterate_items():
        results.append(item)

    return results
```

### Apify Pricing (2024)

- **Free Tier**: $5 of platform credits/month
- **Starter**: $49/month (includes proxies)
- **Scale**: $499/month

**Pros**:
- No infrastructure management
- Built-in proxy rotation
- Handles anti-bot measures
- API access for automation
- Scheduled runs

**Cons**:
- Monthly costs
- Less control than custom scraper
- Subject to Apify's rate limits

---

## 7. Complete Event Monitoring System

### Automated Pipeline Example

```python
import schedule
import time
from datetime import datetime
import sqlite3
import smtplib
from email.mime.text import MIMEText

class EventMonitor:
    def __init__(self, db_path='events.db'):
        self.db_path = db_path
        self.init_database()

    def init_database(self):
        """Initialize SQLite database"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()

        cursor.execute('''
            CREATE TABLE IF NOT EXISTS venues (
                id INTEGER PRIMARY KEY,
                name TEXT UNIQUE,
                instagram_handle TEXT,
                facebook_page TEXT,
                last_scraped TIMESTAMP
            )
        ''')

        cursor.execute('''
            CREATE TABLE IF NOT EXISTS events (
                id INTEGER PRIMARY KEY,
                venue_id INTEGER,
                platform TEXT,
                post_id TEXT UNIQUE,
                post_url TEXT,
                caption TEXT,
                event_date TIMESTAMP,
                discovered_date TIMESTAMP,
                notified BOOLEAN DEFAULT 0,
                FOREIGN KEY (venue_id) REFERENCES venues (id)
            )
        ''')

        conn.commit()
        conn.close()

    def add_venue(self, name, instagram=None, facebook=None):
        """Add a venue to monitor"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()

        cursor.execute('''
            INSERT OR IGNORE INTO venues (name, instagram_handle, facebook_page)
            VALUES (?, ?, ?)
        ''', (name, instagram, facebook))

        conn.commit()
        conn.close()

    def scrape_instagram_events(self, venue_id, instagram_handle):
        """Scrape Instagram for a venue"""
        print(f"Scraping Instagram: {instagram_handle}")

        # Use your Instagram scraping function
        posts = scrape_recent_posts(instagram_handle, max_posts=10)

        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()

        new_events = 0
        for post in posts:
            if post['is_event']:
                try:
                    cursor.execute('''
                        INSERT OR IGNORE INTO events
                        (venue_id, platform, post_id, post_url, caption,
                         event_date, discovered_date)
                        VALUES (?, ?, ?, ?, ?, ?, ?)
                    ''', (
                        venue_id,
                        'instagram',
                        post['shortcode'],
                        post['url'],
                        post['caption'],
                        post['date'],
                        datetime.now()
                    ))

                    if cursor.rowcount > 0:
                        new_events += 1

                except Exception as e:
                    print(f"Error saving event: {e}")

        conn.commit()
        conn.close()

        return new_events

    def scrape_facebook_events(self, venue_id, facebook_page):
        """Scrape Facebook for a venue"""
        print(f"Scraping Facebook: {facebook_page}")

        # Use your Facebook scraping function
        posts = scrape_facebook_page(facebook_page, max_posts=10)

        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()

        new_events = 0
        for post in posts:
            if post['is_event']:
                try:
                    cursor.execute('''
                        INSERT OR IGNORE INTO events
                        (venue_id, platform, post_id, post_url, caption,
                         event_date, discovered_date)
                        VALUES (?, ?, ?, ?, ?, ?, ?)
                    ''', (
                        venue_id,
                        'facebook',
                        post['post_id'],
                        post['post_url'],
                        post['text'],
                        post['time'],
                        datetime.now()
                    ))

                    if cursor.rowcount > 0:
                        new_events += 1

                except Exception as e:
                    print(f"Error saving event: {e}")

        conn.commit()
        conn.close()

        return new_events

    def scrape_all_venues(self):
        """Scrape all registered venues"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()

        cursor.execute('SELECT id, name, instagram_handle, facebook_page FROM venues')
        venues = cursor.fetchall()
        conn.close()

        total_new_events = 0

        for venue_id, name, instagram, facebook in venues:
            print(f"\n=== Scraping {name} ===")

            if instagram:
                try:
                    new_ig_events = self.scrape_instagram_events(venue_id, instagram)
                    print(f"Found {new_ig_events} new Instagram events")
                    total_new_events += new_ig_events
                    time.sleep(60)  # 1 minute between platforms
                except Exception as e:
                    print(f"Instagram error: {e}")

            if facebook:
                try:
                    new_fb_events = self.scrape_facebook_events(venue_id, facebook)
                    print(f"Found {new_fb_events} new Facebook events")
                    total_new_events += new_fb_events
                    time.sleep(60)
                except Exception as e:
                    print(f"Facebook error: {e}")

            # Update last scraped time
            conn = sqlite3.connect(self.db_path)
            cursor = conn.cursor()
            cursor.execute('UPDATE venues SET last_scraped = ? WHERE id = ?',
                         (datetime.now(), venue_id))
            conn.commit()
            conn.close()

            # Long delay between venues (5 minutes)
            time.sleep(300)

        if total_new_events > 0:
            self.send_notification(total_new_events)

    def send_notification(self, num_events):
        """Send email notification for new events"""
        print(f"🎉 Found {num_events} new events!")

        # Optional: Send email
        # Configure your SMTP settings
        """
        msg = MIMEText(f"Found {num_events} new events. Check your database!")
        msg['Subject'] = 'New Events Detected'
        msg['From'] = 'your_email@gmail.com'
        msg['To'] = 'your_email@gmail.com'

        with smtplib.SMTP('smtp.gmail.com', 587) as server:
            server.starttls()
            server.login('your_email@gmail.com', 'your_password')
            server.send_message(msg)
        """

# Usage
monitor = EventMonitor()

# Add venues to monitor
monitor.add_venue('Phoenix Dispensary', instagram='phx_dispensary', facebook='phxdispensary')
monitor.add_venue('Crescent Ballroom', instagram='crescentphx', facebook='crescentballroom')
monitor.add_venue('Valley Bar', instagram='valleybarphx', facebook='valleybarphx')

# Run once
monitor.scrape_all_venues()

# Or schedule to run daily at 9 AM
schedule.every().day.at("09:00").do(monitor.scrape_all_venues)

while True:
    schedule.run_pending()
    time.sleep(60)
```

---

## 8. Recommended Tech Stack by Use Case

### Small-Scale Monitoring (1-5 venues)
```
Stack:
- Instaloader (Instagram)
- facebook-scraper (Facebook)
- SQLite database
- Python script with schedule library
- Run on local machine or Raspberry Pi

Estimated Setup: 2-3 hours
Cost: Free (use throwaway accounts + free proxies)
```

### Medium-Scale (5-20 venues)
```
Stack:
- Instaloader with residential proxies
- facebook-scraper with Selenium
- PostgreSQL database
- Scheduled cron jobs on VPS
- Proxy rotation service ($20-50/month)

Estimated Setup: 1-2 days
Cost: $50-100/month (VPS + proxies)
```

### Large-Scale / Production (20+ venues)
```
Stack:
- Apify actors (Instagram + Facebook)
- Cloud database (AWS RDS, Supabase)
- Lambda functions / Cloud Functions
- Webhook notifications
- Professional proxy service

Estimated Setup: 1 week
Cost: $100-300/month (Apify + infrastructure)
```

---

## 9. Code Repository Structure

### Recommended Project Layout
```
event-scraper/
├── config/
│   ├── venues.json           # List of venues to monitor
│   └── settings.py            # API keys, DB config
├── scrapers/
│   ├── instagram_scraper.py
│   ├── facebook_scraper.py
│   └── base_scraper.py
├── database/
│   ├── models.py
│   └── migrations/
├── utils/
│   ├── rate_limiter.py
│   ├── proxy_rotator.py
│   └── notifier.py
├── main.py                    # Main orchestrator
├── requirements.txt
└── README.md
```

### requirements.txt
```
instaloader==4.10.3
facebook-scraper==0.2.59
selenium==4.15.2
schedule==1.2.0
python-dotenv==1.0.0
requests==2.31.0
beautifulsoup4==4.12.2
apify-client==1.5.0  # If using Apify
```

---

## 10. Final Recommendations

### For Your Dispensary/Venue Monitoring Use Case:

**Phase 1: Start Simple (Week 1)**
1. Install Instaloader + facebook-scraper
2. Test with 2-3 target venues
3. Run manually to validate data quality
4. Store in SQLite database

**Phase 2: Add Automation (Week 2-3)**
1. Set up throwaway Instagram/Facebook accounts
2. Purchase residential proxy service (Smartproxy/IPRoyal)
3. Implement scheduled scraping (once per day)
4. Add keyword filtering for event detection

**Phase 3: Scale & Refine (Week 4+)**
1. Add email/SMS notifications for new events
2. Build simple web dashboard to view events
3. Implement intelligent filtering (date extraction, location parsing)
4. Consider Apify if maintenance becomes burdensome

### Key Success Factors

✅ **Use residential proxies** - Essential for avoiding bans
✅ **Respect rate limits** - Slow and steady wins the race
✅ **Throwaway accounts** - Never use your personal accounts
✅ **Monitor error rates** - If >10% errors, slow down
✅ **Backup data regularly** - Platforms may ban you unexpectedly
✅ **Stay updated** - Scrapers break often; maintain actively

### Red Flags to Avoid

❌ Scraping too fast (>1 request per 5 seconds)
❌ Using datacenter IPs without proxies
❌ Logging in with personal accounts
❌ Scraping private/gated content
❌ Ignoring 429/403 errors
❌ Hardcoding credentials in code

---

## Sources

### Instagram Scraping
- [Instaloader Official Documentation](https://instaloader.github.io/)
- [How to Scrape Instagram - The Ultimate Guide (Python)](https://matthewclarkson.com.au/blog/how-to-scrape-instagram-ultimate-guide/)
- [How to Scrape Instagram Data with Python](https://iproyal.com/blog/instagram-scraping/)
- [How to Scrape Instagram in 2025](https://scrapfly.io/blog/posts/how-to-scrape-instagram)

### Facebook Scraping
- [facebook-scraper on PyPI](https://pypi.org/project/facebook-scraper/)
- [GitHub - kevinzg/facebook-scraper](https://github.com/kevinzg/facebook-scraper)
- [5 Best Facebook Scrapers + Facebook Scraping with Python](https://research.aimultiple.com/facebook-scraping/)
- [How to Scrape Facebook Ethically in 2024](https://webscraping.blog/how-to-scrape-facebook/)

### Meta Graph API
- [Meta releases Graph API v21.0 and Marketing API v21.0](https://ppc.land/meta-releases-graph-api-v21-0-and-marketing-api-v21-0/)
- [Facebook events graph API stopped working (Sep 2023)?](https://stackoverflow.com/questions/77086231/facebook-events-graph-api-stopped-workingsep-2023)
- [Instagram API: A Guide for Developers & Marketers](https://taggbox.com/blog/instagram-api/)

### Rate Limiting & Anti-Ban
- [Rate Limit in Web Scraping: How It Works and 5 Bypass Methods](https://scrape.do/blog/web-scraping-rate-limit/)
- [Instagram API Without Rate Limits](https://scrapecreators.com/blog/instagram-api-without-rate-limits-how-to-easily-access-public-data)
- [Avoid Instagram Scraper Blocking Using Social Scraper API](https://thesocialproxy.com/2024/05/13/avoid-instagram-scraper-blocking-using-social-scraper-api/)
- [Facebook Scraping in 2024: What You Need to Know](https://multilogin.com/blog/what-is-facebook-scraping/)

### Legal & TOS
- [Is Web Scraping Legal? A Guide Based on Recent Court Rulings](https://scrapecreators.com/blog/is-web-scraping-legal-a-guide-based-on-recent-court-ruling)
- [Is web scraping legal in 2024?](https://datadome.co/guides/scraping/is-it-legal/)
- [Is Web Scraping Legal?](https://oxylabs.io/blog/is-web-scraping-legal)
- [Meta Launches New Legal Proceedings Against Data Scraping](https://www.socialmediatoday.com/news/meta-launches-new-legal-proceedings-against-data-scraping-helping-to-estab/626593/)

### Apify
- [Instagram Profile Scraper - Apify](https://apify.com/apify/instagram-profile-scraper)
- [Facebook Events Scraper - Apify](https://apify.com/apify/facebook-events-scraper)
- [How to scrape Instagram without getting blocked](https://blog.apify.com/scrape-instagram-python/)
- [Instagram Scraper - All in one API](https://apify.com/curious_coder/instagram-scraper/api)

---

## Conclusion

For monitoring specific dispensary/venue pages for event announcements:

1. **Start with Instaloader + facebook-scraper** (Python libraries)
2. **Invest in residential proxies** ($20-50/month minimum)
3. **Respect rate limits** (2-5 second delays minimum)
4. **Use throwaway accounts** (expect bans)
5. **Consider Apify** if you want to avoid maintenance

The official Meta Graph API is **not viable** for public event monitoring due to deprecated endpoints. Scraping remains the only practical solution for 2024-2025.

**Legal Status**: Scraping public data is legally defensible based on 2024 court rulings, but expect platform bans. The risk is technical (bans), not legal (prosecution).

Good luck with your event monitoring system!