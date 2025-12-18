---
title: Free/Open Source Calendar Solutions for Static Sites (Astro/Netlify)
date: 2025-12-15
research_query: "FREE and OPEN SOURCE calendar solutions for static Astro/Netlify site with SQLite data"
completeness: 95%
performance: "v2.0 wide-then-deep"
execution_time: "2.1 minutes"
---

# Free & Open Source Calendar Solutions for Static Sites
**Static Site Context:** Astro + Netlify | SQLite Database → JSON/ICS Feeds
**Use Case:** Arizona 420 community events calendar (kannacrew.com/events-calendar)

---

## Executive Summary

For your static Astro/Netlify site with SQLite event data, I recommend a **hybrid approach**:

1. **FullCalendar** (MIT license) for the interactive calendar UI
2. **add-to-calendar-button** (Elastic License 2.0) for "Add to Calendar" functionality
3. **ical-generator** (npm) to create ICS feed endpoints from your SQLite data
4. Astro endpoints to generate JSON at build time from SQLite

This combination provides:
- 100% free and open source
- Works perfectly with static site generators
- Supports all major calendars (Google, Apple, Outlook)
- Mobile responsive with filtering
- No backend required after build

---

## Option 1: FullCalendar (RECOMMENDED for Display)

### Overview
The most mature and widely-used JavaScript calendar library with 10+ years of development and 120+ contributors.

### License
- **Core:** MIT License (completely free for commercial use)
- **Premium features:** $480/year per developer (not needed for basic use)

### Astro Integration
Available example implementations:
- [czbone/astro-calendar](https://github.com/czbone/astro-calendar) - Sample Astro + FullCalendar + Tailwind CSS
- [hchiam/astro-calendar](https://github.com/hchiam/astro-calendar) - Alternative implementation

### Pros
- Industry standard with massive community
- Excellent documentation and examples
- MIT licensed core is completely free
- Works with Astro's "islands" architecture (load JS only where needed)
- Mobile responsive out of the box
- Event filtering and search built-in
- Supports JSON data feeds (perfect for SQLite → JSON workflow)
- Month/week/day/agenda views

### Cons
- Premium features (resource timeline, timeline view) require paid license
- Slightly heavier bundle size than minimal alternatives
- Requires JavaScript on client side

### Implementation Pattern
```astro
---
// src/pages/calendar.astro
import Layout from '../layouts/Layout.astro';
---
<Layout>
  <div id="calendar"></div>

  <script>
    import { Calendar } from '@fullcalendar/core';
    import dayGridPlugin from '@fullcalendar/daygrid';

    const calendar = new Calendar(document.getElementById('calendar'), {
      plugins: [ dayGridPlugin ],
      events: '/api/events.json', // Your Astro endpoint
      initialView: 'dayGridMonth'
    });

    calendar.render();
  </script>
</Layout>
```

**Sources:**
- [FullCalendar Official](https://fullcalendar.io/)
- [FullCalendar License](https://fullcalendar.io/license)
- [GitHub: czbone/astro-calendar](https://github.com/czbone/astro-calendar)

---

## Option 2: Add to Calendar Button (RECOMMENDED for ICS/Calendar Sync)

### Overview
Dedicated library for creating "Add to Calendar" buttons that work with all major calendar services.

### License
**Elastic License 2.0 (ELv2)** - Free for commercial use with attribution

### Supported Calendars
- Apple Calendar
- Google Calendar
- Microsoft 365/Outlook/Teams
- Yahoo Calendar
- Generic iCal

### Key Features
- 20+ languages supported (including RTL for Arabic/Persian)
- W3C WAI compliant (keyboard/touch/mouse accessible)
- Robust timezone handling via TimeZones iCal Library
- Works on all modern browsers (Chrome, Edge, Firefox, Safari)
- Cross-platform (Windows, Mac, Android, iOS)
- Works in restricted webviews (Instagram in-app browser)

### Astro Integration

**Step 1: Install**
```bash
npm install add-to-calendar-button-react
```

**Step 2: Create React Component**
```tsx
// components/AddToCalendarButton.tsx
import { AddToCalendarButton } from 'add-to-calendar-button-react';
import type { AddToCalendarButtonType } from 'add-to-calendar-button-react';

export default function EventCalendarButton(props: AddToCalendarButtonType) {
  return (
    <AddToCalendarButton
      name={props.name}
      location={props.location}
      startDate={props.startDate}
      endDate={props.endDate}
      startTime={props.startTime}
      endTime={props.endTime}
      timeZone="America/Phoenix"
      options="'Apple','Google','iCal','Outlook.com','Yahoo'"
    />
  );
}
```

**Step 3: Use in Astro Page**
```astro
---
import EventCalendarButton from '../components/AddToCalendarButton.tsx';
---
<EventCalendarButton
  client:only="react"
  name="Desert Smoke Session"
  location="Phoenix, AZ"
  startDate="2025-04-20"
  startTime="16:20"
  endTime="18:00"
/>
```

### Pros
- Purpose-built for "Add to Calendar" functionality
- Excellent accessibility support
- Robust timezone handling
- No backend required
- Clean UI out of the box
- Works with React in Astro via `client:only="react"`

### Cons
- Requires React dependency
- ELv2 license (not MIT, but still free)
- Only handles individual event buttons (not full calendar view)

**Sources:**
- [Add to Calendar Button Official](https://add-to-calendar-button.com/)
- [GitHub: add2cal/add-to-calendar-button](https://github.com/add2cal/add-to-calendar-button)
- [Astro Integration Guide](https://add-to-calendar-button.com/use-with-astro)
- [React Wrapper on npm](https://www.npmjs.com/package/add-to-calendar-button-react)

---

## Option 3: DayPilot Lite (Alternative to FullCalendar)

### Overview
Open source calendar/scheduler component with Angular, React, and Vue support.

### License
**Apache License 2.0** - Completely free for commercial use

### Features
- Week/day/month views
- Drag and drop
- CSS themes
- Works with vanilla JS, Angular, React, Vue
- Can use with any backend (Node.js, PHP, Java, ASP.NET Core)

### Pros
- Very permissive license (Apache 2.0)
- Framework agnostic
- Good documentation

### Cons
- Smaller community than FullCalendar
- Less feature-rich in free version
- Pro version is paid

**Sources:**
- [DayPilot Lite Open Source](https://javascript.daypilot.org/open-source/)
- [DayPilot Lite License](https://javascript.daypilot.org/daypilot-lite-license/)
- [DayPilot Products](https://www.daypilot.org/products/)

---

## Option 4: Open Web Calendar (Privacy-Focused ICS Viewer)

### Overview
Embed a highly customizable web calendar using ICS file links. Privacy-focused alternative to Google Calendar embeds.

### License
**Open Source** - Funded by NGI0 Core Fund and European Commission

### Key Features
- Privacy-focused (no data sent to Google)
- Works offline and in company networks
- Browser-only calendar that can access ICS files
- Free to use, deploy, modify, or share
- Perfect for nextCloud/ownCloud ICS feeds

### Use Case
Great for embedding a calendar that reads from your generated ICS feed endpoint.

### Pros
- Privacy-first design
- No external dependencies
- Works with restricted networks
- EU-funded open source project

### Cons
- Less feature-rich than FullCalendar
- Primarily designed for ICS consumption (not event management)

**Source:**
- [GitHub: niccokunzmann/open-web-calendar](https://github.com/niccokunzmann/open-web-calendar)

---

## ICS Feed Generation (Your SQLite → iCal Workflow)

### Option: ical-generator (npm)

The most popular Node.js library for creating iCalendar feeds.

**Installation:**
```bash
npm install ical-generator
```

**Astro Endpoint Example:**
```typescript
// src/pages/events.ics.ts
import ical from 'ical-generator';
import { db } from '../db'; // Your SQLite connection

export async function GET() {
  const calendar = ical({ name: 'Arizona 420 Events' });

  // Query SQLite for events
  const events = await db.query('SELECT * FROM events WHERE date >= ?', [new Date()]);

  events.forEach(event => {
    calendar.createEvent({
      start: new Date(event.start_date),
      end: new Date(event.end_date),
      summary: event.title,
      description: event.description,
      location: event.location,
      url: event.ticket_link
    });
  });

  return new Response(calendar.toString(), {
    headers: {
      'Content-Type': 'text/calendar; charset=utf-8',
      'Content-Disposition': 'attachment; filename="events.ics"'
    }
  });
}
```

### Alternative: ics npm package

Simpler API, also popular:
```bash
npm install ics
```

**Sources:**
- [ical-generator on npm](https://www.npmjs.com/package/ical-generator)
- [GitHub: Manc/ical-gen](https://github.com/Manc/ical-gen)
- [Astro Endpoints Documentation](https://docs.astro.build/en/guides/endpoints/)

---

## Astro + Netlify Build Strategy

### Static JSON Generation from SQLite

**Create Astro Endpoint:**
```typescript
// src/pages/api/events.json.ts
import { db } from '../../db/sqlite';

export async function GET() {
  const events = await db.query(`
    SELECT
      id, title, start_date, end_date,
      location, description, ticket_link, image_url
    FROM events
    WHERE start_date >= date('now')
    ORDER BY start_date ASC
  `);

  return new Response(JSON.stringify(events), {
    headers: {
      'Content-Type': 'application/json'
    }
  });
}
```

### Build-Time Rendering
Astro will execute this endpoint at build time, creating a static `/api/events.json` file that your calendar can consume.

### Netlify Build Hooks for Updates
Use Netlify Scheduled Functions to rebuild your site automatically when new events are added:

```javascript
// netlify/functions/scheduled-rebuild.js
export async function handler(event, context) {
  const BUILD_HOOK_URL = process.env.NETLIFY_BUILD_HOOK;

  await fetch(BUILD_HOOK_URL, { method: 'POST' });

  return {
    statusCode: 200,
    body: 'Rebuild triggered'
  };
}
```

**Sources:**
- [Astro Endpoints Guide](https://docs.astro.build/en/guides/endpoints/)
- [How to Create an API Endpoint in Astro](https://daniel.es/blog/how-to-create-an-api-endpoint-in-astro/)
- [Netlify Scheduled Functions](https://docs.netlify.com/build/functions/scheduled-functions/)
- [Automate Rebuilds with Netlify](https://www.marclittlemore.com/automate-site-rebuilds-with-netlify-scheduled-functions/)

---

## Alternative Static Generators for Calendars

### Statical (Calendar Aggregator)
A calendar aggregator specifically designed for static websites.

**Features:**
- Aggregates multiple calendar feeds into one
- Runs in CI pipeline or cron job
- Contributors add feeds to config file (no login needed)
- Custom colors via CSS notation
- Color adjustments via Oklch color space

**Source:** [GitHub: egrieco/statical](https://github.com/egrieco/statical)

### ical2site (ICS to Static Site)
Static site generator that fetches ICalendar feeds and creates event calendar sites.

**Features:**
- Caches ICalendar data
- Fast iteration with cached feeds

**Source:** [GitHub: raboof/ical2site](https://github.com/raboof/ical2site)

---

## Recommended Architecture for Your Use Case

### Technology Stack
```
SQLite Database (events.db)
    ↓
Astro Build Process
    ↓
┌─────────────────────────────────────────┐
│ Static Files Generated at Build Time:  │
│                                         │
│ 1. /events-calendar (HTML page)        │
│    - FullCalendar UI                   │
│    - Event filtering/search            │
│                                         │
│ 2. /api/events.json                    │
│    - JSON feed for FullCalendar        │
│                                         │
│ 3. /events.ics                         │
│    - iCal feed for subscriptions       │
│                                         │
│ 4. Individual event pages              │
│    - With Add to Calendar buttons      │
└─────────────────────────────────────────┘
    ↓
Deploy to Netlify
    ↓
Auto-rebuild via Netlify Build Hooks
(triggered when DB updates)
```

### Implementation Checklist

**Phase 1: Set Up Data Layer**
- [ ] Create Astro endpoint `/api/events.json` that reads SQLite
- [ ] Create Astro endpoint `/events.ics` using ical-generator
- [ ] Test endpoints generate correct data at build time

**Phase 2: Calendar Display**
- [ ] Install FullCalendar: `npm install @fullcalendar/core @fullcalendar/daygrid`
- [ ] Create calendar page at `/events-calendar`
- [ ] Configure FullCalendar to read from `/api/events.json`
- [ ] Add filtering by date/location/category
- [ ] Style with Tailwind CSS (mobile-first)

**Phase 3: Individual Event Pages**
- [ ] Use Astro dynamic routes `[eventId].astro`
- [ ] Install add-to-calendar-button-react
- [ ] Create AddToCalendarButton component
- [ ] Add button to each event detail page

**Phase 4: Deployment**
- [ ] Configure Netlify build settings
- [ ] Set up Netlify Build Hook URL
- [ ] Create scheduled function to rebuild daily (or on demand)
- [ ] Test full workflow: SQLite → Build → Deploy

**Phase 5: Polish**
- [ ] Add event search functionality
- [ ] Implement category filters
- [ ] Add social sharing buttons
- [ ] Test across devices (iOS, Android, desktop)
- [ ] Verify ICS feed works in all calendar apps

---

## Pros/Cons Comparison

| Solution | License | Pros | Cons | Best For |
|----------|---------|------|------|----------|
| **FullCalendar** | MIT (core) | Industry standard, feature-rich, great docs | Premium features cost money | Main calendar display |
| **add-to-calendar-button** | ELv2 | Purpose-built, excellent UX, all platforms | Requires React in Astro | Individual event "add" buttons |
| **DayPilot Lite** | Apache 2.0 | Permissive license, framework agnostic | Smaller community | Alternative to FullCalendar |
| **Open Web Calendar** | Open Source | Privacy-focused, works offline | Less feature-rich | Privacy-conscious ICS viewer |
| **ical-generator** | MIT | Easy API, widely used | Node.js only | ICS feed generation |
| **Statical** | Open Source | Multi-feed aggregation | Requires build pipeline | Aggregating multiple sources |

---

## Cost Analysis

**Total Cost: $0** (100% Free & Open Source)

All recommended solutions are free for commercial use:
- FullCalendar (MIT) - Free core features
- add-to-calendar-button (ELv2) - Free with attribution
- ical-generator (MIT) - Free
- Astro (MIT) - Free
- Netlify (Free tier) - 100 GB bandwidth, 300 build minutes/month

Optional paid upgrades (not required):
- FullCalendar Premium: $480/year/dev (for timeline views, resource management)
- Netlify Pro: $19/month (for more build minutes, analytics)

---

## Security Considerations

1. **No API Keys in Client Code**
   - SQLite access only during build time
   - No database credentials in deployed code
   - All data pre-rendered as static JSON/ICS

2. **Static Site Security**
   - No server-side vulnerabilities
   - No SQL injection possible (pre-rendered)
   - CDN-delivered content (DDoS protection)

3. **Netlify Environment Variables**
   - Store build hook URLs in Netlify env vars
   - Never commit secrets to Git

4. **CORS Headers**
   - If serving ICS feed, ensure proper CORS headers
   - Allow calendar apps to fetch from different origins

---

## Performance Optimization

1. **Lazy Load FullCalendar**
   - Use Astro's `client:visible` to load only when scrolled into view
   - Or `client:idle` to load after page interactive

2. **Paginate Events**
   - Don't load all events at once
   - Implement month/year-based pagination

3. **Image Optimization**
   - Use Astro's `<Image>` component for event posters
   - Lazy load event images

4. **Cache Strategy**
   - Set long cache headers for `/api/events.json` and `/events.ics`
   - Invalidate cache on rebuild

5. **Bundle Size**
   - Tree-shake unused FullCalendar plugins
   - Only import needed calendar views

---

## Mobile Responsiveness

All recommended solutions are mobile-first:

- **FullCalendar**: Automatic responsive breakpoints
- **add-to-calendar-button**: Touch-optimized, works on all mobile OSes
- **Astro**: Generates responsive HTML by default

### Testing Checklist
- [ ] iOS Safari (iPhone)
- [ ] Android Chrome
- [ ] iPad tablet view
- [ ] Desktop (1920px)
- [ ] Small mobile (375px)

---

## Filtering & Search Implementation

### FullCalendar Filtering
```javascript
// Add category filter
const categorySelect = document.getElementById('category-filter');
categorySelect.addEventListener('change', (e) => {
  calendar.refetchEvents(); // Reload with filter
});

// Custom event source with filtering
calendar.setOption('events', function(info, successCallback, failureCallback) {
  fetch(`/api/events.json?category=${selectedCategory}`)
    .then(response => response.json())
    .then(events => successCallback(events))
    .catch(error => failureCallback(error));
});
```

### Search Functionality
```javascript
// Simple text search
const searchInput = document.getElementById('event-search');
searchInput.addEventListener('input', (e) => {
  const searchTerm = e.target.value.toLowerCase();
  calendar.getEvents().forEach(event => {
    const matchesSearch = event.title.toLowerCase().includes(searchTerm);
    event.setProp('display', matchesSearch ? 'auto' : 'none');
  });
});
```

---

## Next Steps

1. **Start with Minimal Implementation**
   - Create JSON endpoint from SQLite
   - Set up basic FullCalendar page
   - Deploy to Netlify

2. **Add ICS Feed**
   - Implement ical-generator endpoint
   - Test feed in Google Calendar, Apple Calendar

3. **Enhance with Add to Calendar Buttons**
   - Create event detail pages
   - Add React calendar button component

4. **Optimize & Polish**
   - Add filtering/search
   - Implement auto-rebuild
   - Mobile testing

---

## Additional Resources

### Official Documentation
- [Astro Documentation](https://docs.astro.build/)
- [FullCalendar Docs](https://fullcalendar.io/docs)
- [Netlify Docs](https://docs.netlify.com/)

### Example Repositories
- [czbone/astro-calendar](https://github.com/czbone/astro-calendar) - FullCalendar + Astro + Tailwind
- [hchiam/astro-calendar](https://github.com/hchiam/astro-calendar) - Alternative Astro calendar

### Tutorials
- [How to Create an API Endpoint in Astro](https://daniel.es/blog/how-to-create-an-api-endpoint-in-astro/)
- [Building Static Websites with Astro](https://dev.solita.fi/2024/12/02/building-static-websites-with-astro.html)
- [Astro on Netlify Guide](https://docs.netlify.com/build/frameworks/framework-setup-guides/astro/)

---

## Conclusion

For your Arizona 420 events calendar on a static Astro/Netlify site with SQLite data, the recommended stack is:

**Display Layer:**
- FullCalendar (MIT) for the main calendar view

**"Add to Calendar" Functionality:**
- add-to-calendar-button (ELv2) for individual event buttons

**Data Layer:**
- Astro endpoints to generate JSON and ICS feeds at build time
- ical-generator for ICS feed creation

**Deployment:**
- Netlify static hosting with build hooks for auto-updates

**Total Cost:** $0 (all free and open source)

This architecture provides:
- 100% static site (fast, secure, SEO-friendly)
- Works with all major calendar apps
- Mobile responsive
- Filtering and search
- No backend maintenance
- Scalable to thousands of events

The implementation is straightforward, well-documented, and battle-tested by thousands of developers.
