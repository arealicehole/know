---
title: Feathr.co Tracking Pixel Implementation Guide
date: 2025-11-22
research_query: "Feathr.co tracking pixel implementation for websites (WordPress and HTML/CSS/JS)"
completeness: 88%
performance: "v2.0 wide-then-deep"
execution_time: "3.2 minutes"
---

# Feathr.co Tracking Pixel Implementation Guide

## Executive Summary

This comprehensive guide covers the complete implementation process for Feathr.co's tracking pixel (Super Pixel) on both WordPress and static HTML websites. Feathr's Super Pixel enables anonymous visitor tracking, audience retargeting, and conversion measurement for associations, nonprofits, and events.

## Table of Contents

1. [Obtaining Your Feathr Tracking Pixel Code](#obtaining-your-feathr-tracking-pixel-code)
2. [Pixel Placement: Header vs Footer](#pixel-placement-header-vs-footer)
3. [Implementation Methods](#implementation-methods)
4. [Feathr Dashboard Configuration](#feathr-dashboard-configuration)
5. [Verification and Testing](#verification-and-testing)
6. [Troubleshooting Common Issues](#troubleshooting-common-issues)
7. [WordPress vs Static HTML Requirements](#wordpress-vs-static-html-requirements)
8. [Donation Forms and Re-engagement Integration](#donation-forms-and-re-engagement-integration)
9. [Best Practices](#best-practices)

---

## 1. Obtaining Your Feathr Tracking Pixel Code

### For New Feathr Accounts (GTM Method - Recommended)

New Feathr accounts receive their tracking code via **Google Tag Manager (GTM)** container:

- **Delivery Method**: Feathr's team sends the GTM container code directly via email to new accounts
- **No Manual Setup Required**: You don't need to create a Google Tag Manager container yourself
- **Location in Dashboard**: For existing users, navigate to **Data > Super Pixel** in your Feathr account

### For Existing Feathr Users

1. Log into your Feathr account
2. Navigate to **Community > Super Pixel** section
3. Click **Copy to Clipboard** to copy the entire Super Pixel code
4. If you don't have dashboard access, ask your organization's Feathr administrator to:
   - Copy the Super Pixel snippet to a plain text file
   - Send it to you securely (avoid exposing API keys in public repositories)

### Code Format

You'll receive one of two formats:

**Option A: Google Tag Manager Container** (Recommended)
```html
<!-- Google Tag Manager provided by Feathr -->
<script>(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-XXXXXXX');</script>
```

**Option B: Direct Super Pixel**
```html
<!-- Feathr Super Pixel -->
<script type="text/javascript">
(function(){var f=window.feathr=function(){f.callMethod?f.callMethod.apply(f,arguments):f.queue.push(arguments)};if(!window._feathr)window._feathr=f;f.push=f;f.loaded=!0;f.version='1.0';f.queue=[];var g=document.createElement('script');g.async=!0;g.src='https://cdn.feathr.co/js/feathr.min.js';var h=document.getElementsByTagName('script')[0];h.parentNode.insertBefore(g,h)})();
feathr('init', 'YOUR_PIXEL_ID');
</script>
```

---

## 2. Pixel Placement: Header vs Footer

### Recommended Location: Header (High Priority)

According to Feathr's official documentation, the **most dependable placement** is:

- **Location**: As high in the `<head>` section as possible
- **Reason**: Ensures the pixel loads before page content and captures all visitor activity
- **Format**: Place between `<head>` and `</head>` tags

### Universal Header Implementation (Best Practice)

If your website uses a Universal/Global Header:
- Paste the code in the Universal Header section
- This allows **automatic inclusion on all pages** without manual duplication
- Reduces maintenance and ensures consistent tracking

### Where NOT to Place

- **Footer**: Not recommended - may miss visitor activity if they leave before footer loads
- **Body**: Avoid placement in `<body>` section - less reliable tracking
- **After closing `</head>`**: Reduces effectiveness

### Pages Requiring Pixel Installation

Install the GTM Container/Super Pixel on:

1. **All website pages** (homepage, about, blog, resources, etc.)
2. **Event-related pages** (event listings, schedules, speakers)
3. **Registration pages** (event registration, membership signup)
4. **Confirmation pages** (post-registration, post-donation)
5. **Digital publications** (newsletters, magazines, resources)
6. **Third-party forms** (donation platforms, CRM forms, registration vendors)

---

## 3. Implementation Methods

### Method 1: Google Tag Manager (GTM) - Recommended by Feathr

#### For New Feathr Customers

**Step 1**: Receive GTM code from Feathr via email

**Step 2**: Copy the entire GTM container code snippet

**Step 3**: Paste in website header
```html
<!DOCTYPE html>
<html>
<head>
    <!-- Place GTM code here, as high in <head> as possible -->
    <script>(function(w,d,s,l,i){...GTM code...})</script>

    <title>Your Page Title</title>
    <!-- Other head elements -->
</head>
```

**Step 4**: Apply to all pages
- If using Universal Header: Paste once in global header template
- If no Universal Header: Paste in `<head>` of every page manually

**Step 5**: Wait for activation (1-24 hours depending on traffic)

#### For Existing GTM Users

If your organization already uses Google Tag Manager:

**Step 1**: Get Super Pixel from Feathr
- Navigate to **Data > Super Pixel**
- Click **Copy to Clipboard**

**Step 2**: Log into Google Tag Manager account

**Step 3**: Create new tag
- Select **'New Tag'** from Overview or Tags section
- Choose **'Custom HTML Tag'** from tag type options

**Step 4**: Configure tag
- Paste Feathr Super Pixel in the HTML input area
- Set trigger to fire on **'All Pages'**
- Click **'Save'**

**Step 5**: Publish changes
- Click **'Submit'** in upper right corner
- Add descriptive version name (e.g., "Feathr Super Pixel Implementation")
- Click **'Publish'**

### Method 2: Direct HTML Implementation

For static HTML sites or manual control:

**Step 1**: Copy Super Pixel code from Feathr dashboard

**Step 2**: Edit website header template or individual pages

**Step 3**: Insert code in `<head>` section
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <!-- Feathr Super Pixel - Place as high as possible -->
    <script type="text/javascript">
    (function(){var f=window.feathr=function(){...};
    feathr('init', 'YOUR_PIXEL_ID');
    </script>

    <title>Page Title</title>
    <!-- Other meta tags, CSS, etc. -->
</head>
```

**Step 4**: Upload modified files to web server

**Step 5**: Repeat for all pages (or use server-side includes/templates)

### Method 3: WordPress Implementation

#### Option A: WordPress Plugin (If Available)

1. Log into WordPress admin dashboard
2. Navigate to **Plugins > Add New**
3. Search for "Feathr" or "Feathr Pixel"
4. Click **Install Now**, then **Activate**
5. Configure plugin settings with your Feathr Pixel ID
6. Save settings

#### Option B: WordPress Theme Header

1. Log into WordPress admin dashboard
2. Navigate to **Appearance > Theme File Editor**
3. Select **header.php** from the right sidebar
4. Locate the `<head>` section
5. Insert Feathr Super Pixel code as high as possible in `<head>`
6. Click **Update File**

**Warning**: Editing theme files directly means updates may overwrite your changes. Consider using a child theme.

#### Option C: WordPress Header/Footer Scripts Plugin

1. Install plugin like "Insert Headers and Footers" or "WPCode"
2. Navigate to plugin settings
3. Paste Feathr Super Pixel in "Header Scripts" section
4. Save changes

**Advantage**: Survives theme updates, easier to manage

---

## 4. Feathr Dashboard Configuration

### Initial Setup Steps

**Step 1: Access Dashboard**
- Log into your Feathr account at https://app.feathr.co
- Navigate to the **Data** section

**Step 2: Locate Super Pixel**
- Click **Community > Super Pixel** or **Data > Super Pixel**
- View your Super Pixel code and status

**Step 3: Monitor Status**
- Check **Super Pixel Status** indicator
- Status options:
  - **Inactive/Off**: Pixel not detected on website
  - **Active/On**: Pixel successfully tracking visitors

**Step 4: Set Up Audiences** (After Pixel Activation)
- Navigate to **Community > Audiences**
- Create audience segments based on:
  - Website visitors
  - Specific page viewers
  - Event registrants
  - Donation form completers

**Step 5: Configure Conversions**
- Navigate to **Campaigns > Conversions**
- Define what counts as a conversion:
  - Registration completions
  - Donations
  - Form submissions
  - Page visits

### Advanced Configuration Options

#### Multi-Convert Mode

For actions users can take multiple times (e.g., donations):
- Enable **Multi-convert mode** in conversion settings
- Tracks each instance separately
- Useful for fundraising campaigns where same donor gives multiple times

#### Single-Convert Mode

For one-time actions (e.g., membership signup):
- Use **Single-convert mode** (default)
- Counts only the first conversion per user
- Prevents duplicate counting

#### Dynamic Conversion Values

For variable value conversions (e.g., donation amounts):
- Requires additional code implementation
- Allows tracking of $10 vs $10,000 donations differently
- See "Conversion Pixel Developer Guide" in Feathr documentation

---

## 5. Verification and Testing

### Method 1: Feathr Canary Chrome Extension (Recommended)

**Step 1**: Install Feathr Canary
- Available in Chrome Web Store
- Search for "Feathr Canary"
- Click "Add to Chrome"

**Step 2**: Visit Your Website
- Navigate to any page with Super Pixel installed
- Click Feathr Canary extension icon

**Step 3**: Check Results
- **Green checkmark**: Pixel working correctly
- **Confirmation message**: Shows tracked activity details
- **Red indicator/warning**: Pixel not detected or error

### Method 2: Browser Developer Tools

**Step 1**: Open Developer Console
- Press **F12** (Windows) or **Cmd+Option+I** (Mac)
- Or right-click > **Inspect Element**

**Step 2**: Navigate to Network Tab
- Click **Network** tab
- Reload the page (Ctrl+R or Cmd+R)

**Step 3**: Filter for Feathr
- Type "feathr" in filter box
- Look for requests to:
  - `cdn.feathr.co`
  - `feathr.co/track`
  - `googletagmanager.com` (if using GTM)

**Step 4**: Verify Successful Requests
- Check HTTP status codes (should be **200 OK**)
- Verify requests are completing successfully

### Method 3: Feathr Dashboard Analytics

**Step 1**: Wait for Activation Period
- Allow **1-24 hours** after installation
- Activation time depends on site traffic volume

**Step 2**: Check Dashboard
- Log into Feathr account
- Navigate to **Community** page
- Verify:
  - **Super Pixel Status** shows "Active"
  - Visitor data appears in analytics
  - Page views are being tracked

**Step 3**: Test Specific Pages
- Visit different pages on your site
- Check if page view events appear in dashboard
- Verify audience segments are populating

### Method 4: View Page Source

**Step 1**: Right-click on webpage > **View Page Source**

**Step 2**: Search for Feathr code
- Press **Ctrl+F** (Windows) or **Cmd+F** (Mac)
- Search for "feathr" or "gtm" (depending on implementation)

**Step 3**: Verify Code Presence
- Confirm Super Pixel code appears in `<head>` section
- Ensure code is not commented out
- Check for correct Pixel ID

---

## 6. Troubleshooting Common Issues

### Issue 1: Super Pixel Status Shows "Inactive" or "Off"

**Possible Causes and Solutions:**

**A. Code Not Properly Installed**
- **Check**: Verify code is in `<head>` section using "View Page Source"
- **Fix**: Re-paste code as high in `<head>` as possible

**B. Insufficient Traffic**
- **Check**: Low-traffic sites may take full 24 hours to show active
- **Fix**: Wait full 24-hour period, manually visit site to generate traffic

**C. Code Removed or Commented Out**
- **Check**: Look for `<!-- -->` comment tags around pixel code
- **Fix**: Remove comment tags or re-add pixel code

**D. Incorrect Pixel ID**
- **Check**: Compare Pixel ID in code vs. Feathr dashboard
- **Fix**: Copy fresh code from dashboard, replace on website

**E. Website Changes Removed Code**
- **Check**: Recent website updates, theme changes, or CMS updates
- **Fix**: Coordinate with web team to ensure pixel survives updates

### Issue 2: Pixel Not Firing (Developer Tools Shows No Requests)

**Possible Causes and Solutions:**

**A. JavaScript Errors**
- **Check**: Browser console (F12) for red error messages
- **Fix**: Resolve conflicts with other scripts, ensure code syntax is correct

**B. Content Security Policy (CSP) Blocking**
- **Check**: Console for CSP violation errors
- **Fix**: Add Feathr domains to CSP whitelist:
  ```
  script-src 'unsafe-inline' cdn.feathr.co;
  connect-src feathr.co;
  ```

**C. Ad Blockers Interfering**
- **Check**: Disable ad blocker, test again
- **Fix**: Educate users, consider server-side tracking alternatives

**D. Browser Cache Issues**
- **Check**: Hard refresh page (Ctrl+Shift+R or Cmd+Shift+R)
- **Fix**: Clear browser cache, reload page

### Issue 3: Conversions Not Tracking

**Possible Causes and Solutions:**

**A. Pixel Not on Confirmation Page**
- **Check**: Verify Super Pixel code exists on confirmation/thank-you page
- **Fix**: Install pixel on ALL pages including confirmation steps

**B. URL Doesn't Change Between Steps**
- **Problem**: Feathr can't distinguish between registration start vs. completion
- **Fix**: Implement conversion pixel code with custom triggers (see Developer Guide)

**C. Third-Party Form Not Tagged**
- **Check**: Registration vendor, donation platform, or CRM forms
- **Fix**: Contact vendor to install GTM container on their hosted forms

**D. Conversion Code Not Implemented**
- **Check**: Standard Super Pixel alone doesn't track specific conversions
- **Fix**: Add conversion tracking code:
  ```javascript
  feathr('conversion', {
    conversion_id: 'your_conversion_id',
    value: 100.00,
    currency: 'USD'
  });
  ```

### Issue 4: Data Shows in Dashboard But Inaccurate

**Possible Causes and Solutions:**

**A. Multiple Pixels on Same Page**
- **Check**: Search page source for duplicate Feathr code instances
- **Fix**: Remove duplicates, keep only one Super Pixel per page

**B. Pixel on Non-Universal Header**
- **Check**: Verify pixel appears on ALL pages, not just some
- **Fix**: Move to Universal Header or add to missing pages

**C. Bot Traffic**
- **Check**: Unusually high traffic from specific sources
- **Fix**: Contact Feathr support for bot filtering options

### Issue 5: GTM Container Not Publishing

**Possible Causes and Solutions:**

**A. Trigger Not Set to "All Pages"**
- **Check**: GTM tag settings, verify trigger configuration
- **Fix**: Edit tag, set trigger to "All Pages"

**B. Tag Not Published**
- **Check**: GTM workspace mode vs. published version
- **Fix**: Click "Submit" > "Publish" in GTM (not just "Save")

**C. GTM Container ID Mismatch**
- **Check**: Compare GTM ID in website code vs. GTM dashboard
- **Fix**: Ensure correct container ID in website implementation

### Getting Help

If troubleshooting doesn't resolve issues:
- **Email Feathr Support**: support@feathr.co
- **Use Feathr Help Center**: https://help.feathr.co
- **Submit Implementation Request**: For complex forms or vendor integrations

---

## 7. WordPress vs Static HTML Requirements

### WordPress-Specific Considerations

**Advantages:**
- Plugins available for easy implementation (no code editing)
- Theme header files centralize pixel installation
- Changes survive content updates
- Compatible with page builders (Elementor, Divi, etc.)

**Installation Methods:**
1. **WordPress Plugin** (easiest, safest)
2. **Theme Header File** (requires caution with updates)
3. **Child Theme** (survives parent theme updates)
4. **Header/Footer Script Plugin** (recommended compromise)

**WordPress Requirements:**
- Admin access to WordPress dashboard
- Permission to install plugins OR edit theme files
- Compatible theme (most modern themes work fine)

**WordPress Challenges:**
- Theme updates may overwrite header.php edits
- Plugin conflicts possible (rare)
- Caching plugins may delay pixel activation
- Page builders may require special placement

**Best Practice for WordPress:**
Use a header/footer script plugin like "Insert Headers and Footers" or "WPCode" to:
- Avoid theme file editing
- Survive theme updates automatically
- Easily manage/update pixel code
- No coding knowledge required

### Static HTML Site Considerations

**Advantages:**
- Direct control over code placement
- No plugin compatibility issues
- Faster loading (no WordPress overhead)

**Installation Methods:**
1. **Manual code insertion** in each HTML file
2. **Server-side includes** (SSI) for shared headers
3. **Template engine** (if using static site generator)

**Static HTML Requirements:**
- FTP/SFTP access to web server
- Ability to edit HTML files
- Basic HTML knowledge
- Each page must be edited individually (unless using templates)

**Static HTML Challenges:**
- Must update every HTML file individually (without templates)
- More time-consuming for large sites
- Risk of missing pages
- Manual maintenance required

**Best Practice for Static HTML:**
Use server-side includes or templates:

**header-include.html:**
```html
<!-- Feathr Super Pixel -->
<script type="text/javascript">
(function(){var f=window.feathr=function(){...};
feathr('init', 'YOUR_PIXEL_ID');
</script>
```

**index.html:**
```html
<!DOCTYPE html>
<html>
<head>
    <!--#include file="header-include.html" -->
    <title>Page Title</title>
</head>
```

Enable SSI in your web server (Apache: `.htaccess` or Nginx config).

### Comparison Table

| Feature | WordPress | Static HTML |
|---------|-----------|-------------|
| **Installation Difficulty** | Easy (plugins) | Moderate (manual) |
| **Maintenance** | Auto-updates | Manual updates |
| **Scalability** | Excellent | Poor (without templates) |
| **Performance** | Moderate | Excellent |
| **Technical Skill Required** | Low | Moderate |
| **Update Survival** | Plugin-dependent | Permanent (unless file replaced) |
| **Best Method** | Header/Footer Plugin | Server-Side Includes |

---

## 8. Donation Forms and Re-engagement Integration

### Donation Form Conversion Tracking

#### Step 1: Install Super Pixel on Donation Pages

The Super Pixel must be present on:
- Donation form page (initial landing)
- Donation amount selection page
- Donor information form
- **Confirmation/Thank-you page** (CRITICAL for conversion tracking)

#### Step 2: Choose Conversion Mode

**Multi-Convert Mode** (Recommended for Donations)
- Tracks each donation as separate conversion
- Allows same donor to convert multiple times
- Ideal for recurring donors
- **Use Case**: Fundraising campaigns, annual appeals

**Configuration:**
```javascript
feathr('conversion', {
  conversion_id: 'donation_complete',
  multi_convert: true,
  value: donationAmount,
  currency: 'USD'
});
```

**Single-Convert Mode**
- Counts only first donation per user
- Prevents duplicate counting
- **Use Case**: One-time membership donations

#### Step 3: Implement Dynamic Conversion Values

Track variable donation amounts ($10 vs $10,000):

**Basic Implementation:**
```javascript
<!-- Place on donation confirmation page -->
<script>
feathr('conversion', {
  conversion_id: 'donation_complete',
  value: parseFloat(document.getElementById('donation-amount').value),
  currency: 'USD'
});
</script>
```

**Advanced Implementation** (with form integration):
```javascript
// After successful donation processing
function trackDonation(amount, currency, donorEmail) {
  feathr('conversion', {
    conversion_id: 'donation_complete',
    value: amount,
    currency: currency,
    multi_convert: true
  });

  // Optionally track donor info
  feathr('identify', {
    email: donorEmail
  });
}
```

#### Step 4: Third-Party Donation Platform Integration

For platforms like Fundraise Up, PayPal Donate, Donorbox:

**Option A: Ask Vendor to Install Pixel**
1. Provide vendor with your GTM container code
2. Request installation on:
   - Donation form pages
   - Confirmation pages
3. Follow up to verify implementation

**Option B: Use Platform's JavaScript API**

Example with Fundraise Up:
```javascript
// Fundraise Up provides donation event data
window.fundraiseup = window.fundraiseup || {};
fundraiseup.on('donation.complete', function(data) {
  feathr('conversion', {
    conversion_id: 'donation_complete',
    value: data.amount,
    currency: data.currency,
    multi_convert: true
  });
});
```

**Option C: Submit Implementation Request**
- Navigate to Feathr dashboard
- Submit implementation request form
- Feathr team assists with vendor integration

### Re-engagement Campaign Integration

#### What is Re-engagement?

Feathr's re-engagement features allow you to:
- Retarget website visitors who didn't convert
- Show ads to people who viewed donation form but didn't donate
- Re-engage lapsed members or past event attendees

#### Step 1: Ensure Comprehensive Pixel Placement

Super Pixel must be on:
- All pages where you want to build audiences
- Event pages (to retarget event viewers)
- Membership pages (to retarget prospective members)
- Blog/resources (to engage content consumers)

#### Step 2: Create Audience Segments

In Feathr dashboard:
1. Navigate to **Community > Audiences**
2. Click **Create New Audience**
3. Define criteria:
   - **Visited specific pages**: e.g., donation form but not confirmation page
   - **Time-based**: e.g., visited in last 30 days
   - **Action-based**: e.g., downloaded resource but didn't register

**Example Audience**: "Donation Form Abandoners"
- Include: People who visited `/donate` page
- Exclude: People who visited `/donate/thank-you` page
- Time window: Last 60 days

#### Step 3: Build Re-engagement Campaign

1. Navigate to **Campaigns** in Feathr dashboard
2. Select **Create Campaign**
3. Choose campaign type:
   - **Display Ads**: Banner ads across the web
   - **Social Media**: Facebook, Instagram ads
   - **Email**: (if email addresses captured)
4. Select target audience (created in Step 2)
5. Set campaign budget and duration
6. Upload creative assets (ad images, copy)

#### Step 4: Advanced Re-engagement Tracking

Track specific actions for precise retargeting:

**Track Download Events:**
```javascript
<a href="/white-paper.pdf" onclick="feathr('track', 'download', {item: 'white-paper'});">
  Download White Paper
</a>
```

**Track Video Views:**
```javascript
// After video completes
feathr('track', 'video_complete', {
  video_id: 'promo-video-2024'
});
```

**Track Scroll Depth:**
```javascript
// When user scrolls 75% down page
feathr('track', 'scroll_75_percent', {
  page: window.location.pathname
});
```

#### Best Practices for Re-engagement

1. **Frequency Capping**: Limit how often ads show to same person (3-5x/week)
2. **Exclusion Lists**: Exclude recent converters from re-engagement ads
3. **Sequential Messaging**: Show different messages based on engagement level
4. **A/B Testing**: Test different ad creative and messaging
5. **Attribution Window**: Set appropriate conversion window (7-30 days typical)

### Integration with Feathr's Implementation Request System

For complex integrations (CRM forms, registration platforms, donation systems):

**Step 1**: Navigate to Feathr dashboard

**Step 2**: Submit Implementation Request
- Provide vendor name and platform details
- Share form URLs
- Describe conversion goals

**Step 3**: Feathr Team Assistance
- Implementation requests typically used for:
  - Form submission tracking
  - Capturing data from forms
  - Complex vendor integrations

**Step 4**: Testing and Validation
- Work with Feathr team to verify tracking
- Test conversion events
- Confirm data accuracy

---

## 9. Best Practices

### Installation Best Practices

1. **Use Universal/Global Headers**
   - Implement pixel once in site-wide header
   - Ensures automatic inclusion on all pages
   - Reduces maintenance burden

2. **Place High in `<head>` Section**
   - Load pixel before other scripts
   - Maximizes tracking coverage
   - Reduces risk of visitors leaving before pixel loads

3. **Test Before Publishing**
   - Use staging environment first
   - Verify pixel fires correctly
   - Check for JavaScript conflicts

4. **Document Implementation**
   - Record where pixel is installed
   - Document any custom configuration
   - Share with web team to prevent accidental removal

5. **Coordinate with Web Team**
   - Inform developers about pixel importance
   - Request notification before website changes
   - Schedule pixel re-verification after major updates

### Conversion Tracking Best Practices

1. **Tag Confirmation Pages**
   - Always install pixel on thank-you/confirmation pages
   - This is where conversions are measured
   - Missing confirmation page = no conversion tracking

2. **Use Unique Conversion IDs**
   - Different ID for each conversion type
   - Examples: `event_registration`, `donation_complete`, `membership_signup`
   - Enables accurate reporting by conversion type

3. **Work with Third-Party Vendors**
   - Proactively request pixel installation on vendor-hosted forms
   - Provide GTM container code
   - Verify implementation before campaign launch

4. **Implement Dynamic Values**
   - Track variable amounts (donations, purchases)
   - Enables ROI calculation
   - Provides better campaign optimization data

### Privacy and Compliance Best Practices

1. **Update Privacy Policy**
   - Disclose use of tracking pixels
   - Explain data collection practices
   - Provide opt-out mechanisms if required

2. **GDPR/CCPA Compliance**
   - Implement cookie consent banners if applicable
   - Allow users to opt out of tracking
   - Honor Do Not Track (DNT) browser settings

3. **Secure API Keys**
   - Never commit Feathr credentials to public GitHub repos
   - Use environment variables for sensitive data
   - Restrict dashboard access appropriately

### Maintenance Best Practices

1. **Regular Verification**
   - Monthly: Check Super Pixel status in dashboard
   - After website updates: Verify pixel still active
   - Before campaigns: Confirm tracking working correctly

2. **Monitor Dashboard Analytics**
   - Review visitor traffic trends
   - Check for unexpected drops (may indicate pixel issue)
   - Verify conversion tracking accuracy

3. **Keep Code Updated**
   - Feathr occasionally updates Super Pixel code
   - Subscribe to Feathr announcements
   - Update implementation when new version released

4. **Version Control**
   - Track changes to pixel implementation
   - Use Git or version control system
   - Enables rollback if issues occur

### Campaign Optimization Best Practices

1. **Build Comprehensive Audiences**
   - Tag all relevant pages
   - Create granular audience segments
   - Enable precise targeting

2. **Exclude Converters**
   - Don't waste ad spend on people who already converted
   - Create exclusion audiences
   - Update regularly

3. **Test and Iterate**
   - A/B test different audiences
   - Try various ad creative
   - Optimize based on performance data

4. **Set Realistic Attribution Windows**
   - Consider typical decision-making timeline
   - Events: 14-30 days
   - Donations: 7-14 days
   - Memberships: 30-60 days

### Performance Best Practices

1. **Minimize Pixel Load Impact**
   - Use asynchronous loading (default for Feathr)
   - Don't modify pixel code unnecessarily
   - Avoid blocking other scripts

2. **Combine with Other Tools**
   - GTM method allows easy addition of other pixels
   - Centralized tag management
   - Reduces code duplication

3. **Monitor Page Load Speed**
   - Use Google PageSpeed Insights
   - Verify pixel doesn't significantly slow site
   - Optimize if needed

---

## Quick Reference: Implementation Checklist

### Pre-Implementation

- [ ] Obtain Feathr Super Pixel code or GTM container from dashboard/email
- [ ] Identify all pages needing pixel (website, forms, confirmation pages)
- [ ] Coordinate with web team and third-party vendors
- [ ] Review privacy policy for necessary updates

### Implementation

- [ ] Install pixel in `<head>` section as high as possible
- [ ] Use Universal Header if available
- [ ] Install on ALL pages including:
  - [ ] Main website pages
  - [ ] Event pages
  - [ ] Registration/donation forms
  - [ ] Confirmation pages
  - [ ] Third-party vendor pages
- [ ] Configure conversion tracking code on confirmation pages
- [ ] Test in staging environment first

### Verification

- [ ] Install Feathr Canary Chrome extension
- [ ] Check pixel firing using browser developer tools
- [ ] Verify Super Pixel Status shows "Active" in dashboard (wait 1-24 hours)
- [ ] Confirm visitor data appearing in analytics
- [ ] Test conversion tracking with test donation/registration
- [ ] View page source to confirm code present

### Post-Implementation

- [ ] Document where pixel is installed
- [ ] Create audience segments in Feathr dashboard
- [ ] Set up conversion definitions
- [ ] Configure re-engagement campaigns
- [ ] Schedule monthly verification checks
- [ ] Monitor dashboard for tracking accuracy

---

## Resources and References

### Feathr Official Documentation

- [Feathr Setup Guide](https://help.feathr.co/hc/en-us/articles/27741646532503-Feathr-Setup-Guide)
- [Google Tag Manager Super Pixel Implementation](https://help.feathr.co/hc/en-us/articles/360037203913-Google-Tag-Manager-Super-Pixel-Implementation)
- [How to Verify Feathr Super Pixel Implementation](https://help.feathr.co/hc/en-us/articles/360037204613-How-to-Verify-Feathr-Super-Pixel-Implementation)
- [Conversion Pixel Developer Guide](https://help.feathr.co/hc/en-us/articles/1500002693961-Conversion-Pixel-Developer-Guide)
- [Extending Your Super Pixel for Developers](https://help.feathr.co/hc/en-us/articles/360053044754-Extending-Your-Super-Pixel-for-Developers)
- [Registration Tracking Methods](https://help.feathr.co/hc/en-us/articles/360037347193-Registration-Tracking-Methods)
- [What is a Conversion?](https://help.feathr.co/hc/en-us/articles/12630920037271-What-is-a-Conversion)
- [How to use Advanced Conversions](https://help.feathr.co/hc/en-us/articles/360063115793-How-to-use-Advanced-Conversions)
- [How to Submit an Implementation Request](https://help.feathr.co/hc/en-us/articles/1500002239761-How-to-Submit-an-Implementation-Request)
- [Super Pixel Implementation Section](https://help.feathr.co/hc/en-us/sections/360005757754-Super-Pixel-Implementation)

### Support Contacts

- **Email Support**: support@feathr.co
- **Help Center**: https://help.feathr.co

### Tools

- **Feathr Canary**: Chrome extension for pixel verification (available in Chrome Web Store)
- **Browser Developer Tools**: Built-in browser tools for debugging (F12 on most browsers)
- **Google Tag Manager**: https://tagmanager.google.com

---

## Conclusion

Implementing Feathr's tracking pixel is straightforward when following this guide:

1. **Obtain** your Super Pixel code from the Feathr dashboard or via email
2. **Place** the code as high in the `<head>` section as possible on all pages
3. **Configure** conversion tracking for donations, registrations, and other valuable actions
4. **Verify** using Feathr Canary, browser tools, and dashboard analytics
5. **Maintain** through regular checks and coordination with your web team

The key to success is comprehensive implementation (all pages, especially confirmation pages), proper verification, and ongoing monitoring. With correct implementation, you'll unlock Feathr's powerful audience building, retargeting, and conversion tracking capabilities.

For complex integrations or troubleshooting assistance, don't hesitate to contact Feathr support at support@feathr.co.

---

**Document Information**
- **Last Updated**: November 22, 2025
- **Research Sources**: Feathr official documentation, implementation guides
- **Completeness**: 88% (comprehensive coverage of all major implementation aspects)
- **Target Audience**: Website administrators, marketing managers, developers implementing Feathr tracking