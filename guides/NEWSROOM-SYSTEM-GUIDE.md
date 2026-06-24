# Multi-Platform Content Newsroom System

**Version:** 1.0
**Date:** 2025-11-09
**Purpose:** Complete guide to the multi-platform content creation newsroom powered by specialized subagents

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Agent Roster](#agent-roster)
3. [How to Use](#how-to-use)
4. [Workflows](#workflows)
5. [Platform Strategies](#platform-strategies)
6. [Best Practices](#best-practices)
7. [Examples](#examples)
8. [Troubleshooting](#troubleshooting)

---

## System Overview

The Newsroom System is a multi-agent content creation platform that coordinates specialized platform experts to produce high-quality, optimized content across:

- **YouTube** - Long-form videos and Shorts
- **Instagram** - Posts, Reels, Carousels, Stories
- **TikTok** - Viral short-form videos
- **X (Twitter)** - Tweets, threads, and conversations
- **Website/Blog** - Articles, landing pages, case studies

### Architecture

```
User Request
    ↓
Newsroom Editor (Orchestrator)
    ↓
[YouTube, Instagram, TikTok, X, Website] Creators (Parallel)
    ↓
Editor Review & Quality Control
    ↓
Final Multi-Platform Content Package
```

### Key Features

✅ **Parallel Production** - Create content for all platforms simultaneously
✅ **Platform Expertise** - Each agent is specialized in platform best practices
✅ **Quality Control** - Editor reviews and pushes for excellence
✅ **Adaptation Mode** - Convert existing content to any platform
✅ **Consultant Mode** - Get expert advice without content creation
✅ **Brand Consistency** - Maintain voice across all platforms

---

## Agent Roster

### 1. Newsroom Editor (Orchestrator)
**File:** `.claude/agents/newsroom-editor.md`
**Model:** Sonnet
**Role:** Chief content coordinator

**Responsibilities:**
- Assigns content creation tasks to platform specialists
- Coordinates parallel production workflows
- Reviews all creator outputs for quality
- Ensures brand consistency across platforms
- Provides strategic feedback and revision requests
- Delivers final content packages
- Consults platform experts for strategy

**When to use directly:**
```
Use the newsroom-editor agent to create multi-platform content about [topic]
```

---

### 2. YouTube Creator
**File:** `.claude/agents/youtube-creator.md`
**Model:** Sonnet
**Tools:** WebSearch
**Role:** YouTube video content specialist

**Expertise:**
- Video scripts (long-form and Shorts)
- Titles and thumbnails
- Descriptions and tags
- SEO optimization
- Watch time and retention
- YouTube algorithm

**Output includes:**
- Complete video script with hook, sections, CTA
- 3 title options
- Full description with timestamps
- 15-20 optimized tags
- Thumbnail concept
- Publishing strategy

---

### 3. Instagram Creator
**File:** `.claude/agents/instagram-creator.md`
**Model:** Haiku
**Tools:** WebSearch
**Role:** Instagram visual content specialist

**Expertise:**
- Feed posts and carousels
- Reels (short-form video)
- Stories and Highlights
- Visual aesthetics
- Hashtag strategy
- Instagram algorithm

**Output includes:**
- Format-specific content (post/reel/carousel/story)
- Captions with hooks and CTAs
- Hashtag strategy (5-15 tags)
- Visual concepts and briefs
- Alt text for accessibility
- Posting strategy

---

### 4. TikTok Creator
**File:** `.claude/agents/tiktok-creator.md`
**Model:** Haiku
**Tools:** WebSearch
**Role:** TikTok viral content specialist

**Expertise:**
- Short-form video scripts (15-60s)
- Viral hooks and trends
- Trending sounds
- For You Page (FYP) optimization
- Watch time and completion
- TikTok culture

**Output includes:**
- Scene-by-scene video script
- Hook strategy (first 0.5-1s)
- Trending sound recommendations
- Caption with engagement prompt
- Hashtag strategy
- Viral potential analysis

---

### 5. X (Twitter) Creator
**File:** `.claude/agents/x-creator.md`
**Model:** Haiku
**Tools:** WebSearch
**Role:** X engagement specialist

**Expertise:**
- Tweets and threads
- Viral hooks
- Conversation starters
- Quote tweets and replies
- X algorithm
- Community building

**Output includes:**
- Single tweets or threads
- Hook-optimized text
- Engagement prompts
- Hashtag strategy (0-2 tags)
- Reply and QT strategy
- Posting timing

---

### 6. Website/Blog Creator
**File:** `.claude/agents/website-creator.md`
**Model:** Sonnet
**Tools:** WebSearch
**Role:** Long-form content and conversion specialist

**Expertise:**
- Blog posts and articles (1500-3000+ words)
- Landing pages
- Case studies
- SEO optimization
- Conversion optimization
- Readability and scanning

**Output includes:**
- Complete structured content
- SEO optimization (keywords, meta, headings)
- Internal/external links
- CTAs and conversion elements
- Visual recommendations
- Readability analysis

---

## How to Use

### Mode 1: Multi-Platform Content Creation

**Create content for all platforms simultaneously:**

```
Use the newsroom-editor agent to create content about [topic] for YouTube, Instagram, TikTok, X, and Website.

Goal: [engagement/education/promotion/etc.]
Tone: [professional/casual/witty/etc.]
Target audience: [description]
```

**Example:**
```
Use the newsroom-editor agent to create content about "5 productivity hacks for remote workers" for all platforms.

Goal: Engagement and education
Tone: Friendly and practical
Target audience: Millennial/Gen Z remote workers
```

The editor will:
1. Plan the strategy for each platform
2. Spawn all 5 creators in parallel
3. Review each output for quality
4. Request revisions if needed
5. Deliver complete package

---

### Mode 2: Single Platform Content

**Create content for one specific platform:**

```
Use the newsroom-editor agent to create a YouTube video about [topic]
```

or invoke the specialist directly:

```
Use the youtube-creator agent to create a video script about [topic]
```

---

### Mode 3: Content Adaptation

**Adapt existing content to multiple platforms:**

```
Use the newsroom-editor agent to adapt this blog post to Instagram, TikTok, and X:

[paste content or URL]
```

The editor will:
1. Analyze source material
2. Extract core message and key points
3. Brief each platform creator on adaptation
4. Ensure platform-specific optimization
5. Maintain core message across all versions

---

### Mode 4: Platform Consultation

**Get expert advice without content creation:**

```
Use the newsroom-editor agent to consult the TikTok expert: What's the best hook strategy for educational content?
```

or:

```
Use the instagram-creator agent as a consultant: Should I post this as a Reel or carousel?
```

---

### Mode 5: Review and Feedback

**Get feedback on existing content:**

```
Use the newsroom-editor agent to review this YouTube title and suggest improvements:

"How to Be More Productive"
```

---

## Workflows

### Workflow 1: New Content Campaign (All Platforms)

**Scenario:** Launch new product/service across all channels

```
User Request → Newsroom Editor

Editor:
1. Strategic Planning
   - Analyze topic and goal
   - Define success criteria per platform
   - Identify key messages
   - Create unified strategy

2. Parallel Production
   - Assign to 5 creators simultaneously
   - Each creates platform-optimized content
   - Maintain brand consistency

3. Quality Review
   - Score each output (0-100)
   - Identify strengths and improvements
   - Request revisions if score <75

4. Refinement (if needed)
   - Re-invoke creators with specific feedback
   - Review revised outputs
   - Approve final versions

5. Delivery
   - Package all content
   - Add cross-platform strategy
   - Include posting schedule
   - Provide optimization notes

Timeline: ~15-30 minutes for complete 5-platform package
```

---

### Workflow 2: Content Repurposing

**Scenario:** Turn blog post into social media content

```
User Request (Blog URL) → Newsroom Editor

Editor:
1. Source Analysis
   - Read and extract key points
   - Identify most shareable elements
   - Find platform-specific angles

2. Platform Adaptation
   - YouTube: Video script from key sections
   - Instagram: Carousel with key points
   - TikTok: Hook on most surprising insight
   - X: Thread summarizing main ideas
   - Website: Already exists (maybe optimize)

3. Review & Deliver
   - Ensure each maintains core message
   - Verify platform optimization
   - Package adapted content

Timeline: ~10-20 minutes
```

---

### Workflow 3: Rapid Single-Platform

**Scenario:** Need quick TikTok for trending topic

```
User Request → Newsroom Editor or TikTok Creator directly

Creator:
1. Trend Research (WebSearch)
   - Find trending sounds
   - Check viral formats
   - Validate topic relevance

2. Script Creation
   - Craft strong hook (0.5s)
   - Fast-paced scenes
   - Viral potential optimization

3. Deliver
   - Complete script with sound
   - Caption and hashtags
   - Viral analysis

Timeline: ~3-5 minutes
```

---

## Platform Strategies

### YouTube Strategy

**Goals:**
- Discoverability (SEO)
- Watch time and retention
- Subscriber growth

**Key tactics:**
- Hook in first 5-8 seconds
- Title and thumbnail psychology
- Timestamps in description
- Strategic tags (15-20)
- End screen recommendations

**Metrics to optimize:**
- Click-through rate (CTR)
- Average view duration
- Watch time
- Engagement rate

---

### Instagram Strategy

**Goals:**
- Saves and shares (algorithm favors)
- Aesthetic consistency
- Community engagement

**Key tactics:**
- Format selection (Reel vs Post vs Carousel)
- Visual brand consistency
- Strategic hashtags (5-15)
- Strong captions with CTAs
- Interactive Stories

**Metrics to optimize:**
- Saves (highest value)
- Shares
- Comments
- Story completion rate

---

### TikTok Strategy

**Goals:**
- For You Page (FYP) placement
- Watch time and completion
- Viral potential

**Key tactics:**
- Hook in first 0.5-1 second
- Trending sounds (critical)
- Fast-paced editing
- Loop potential
- Engagement bait (comments)

**Metrics to optimize:**
- Completion rate (>80%)
- Watch time
- Shares
- Comments

---

### X Strategy

**Goals:**
- Engagement and conversation
- Thought leadership
- Community building

**Key tactics:**
- Hook in first 10-20 words
- One clear idea per tweet
- Conversation starters
- Strategic threading
- Quote tweets > retweets

**Metrics to optimize:**
- Replies (highest value)
- Retweets and QTs
- Bookmarks
- Impressions

---

### Website/Blog Strategy

**Goals:**
- Organic traffic (SEO)
- Time on page
- Conversions

**Key tactics:**
- Keyword optimization
- Scannable formatting
- Internal linking
- Strong CTAs
- High readability (Grade 8-10)

**Metrics to optimize:**
- Search rankings
- Organic traffic
- Time on page
- Conversion rate

---

## Best Practices

### 1. Always Start with Clear Goals

**Good:**
```
Create content about remote work productivity with goal of driving newsletter signups from millennial professionals.
```

**Bad:**
```
Make some content about productivity.
```

---

### 2. Provide Brand Guidelines

**Include:**
- Voice and tone (professional, casual, witty, authoritative)
- Key messaging (what must be communicated)
- Visual identity (colors, style, aesthetics)
- Target audience (demographics, psychographics)
- Constraints (what to avoid)

---

### 3. Trust Platform Expertise

Each creator knows their platform's:
- Algorithm optimization
- Best practices
- Cultural norms
- Current trends

Don't override without good reason.

---

### 4. Use Adaptation for Efficiency

**Instead of creating from scratch 5 times:**
1. Create one comprehensive piece (blog post)
2. Adapt to other platforms
3. Maintain core message with platform optimization

**Benefits:**
- Message consistency
- Time efficiency
- Content repurposing

---

### 5. Review Before Publishing

**Editor provides:**
- Quality scores
- Strengths and weaknesses
- Improvement suggestions
- Publishing recommendations

**Use this to:**
- Learn what works
- Improve future briefs
- Refine brand guidelines
- Track platform performance

---

### 6. Iterate Based on Performance

**After publishing:**
- Track metrics per platform
- Identify top performers
- Analyze what worked
- Feed insights back to system

**Next campaign:**
- Reference successful examples
- Apply learnings
- A/B test variations

---

## Examples

### Example 1: Product Launch

**Request:**
```
Use the newsroom-editor agent to create content for launching our new project management app "FlowState" across all platforms.

Goal: Generate beta signups
Tone: Modern, productivity-focused, slightly playful
Target: Startup founders and product managers
Key message: "Project management that doesn't get in the way"
```

**Editor delivers:**

**YouTube** (9:30 video script)
- Title: "I Tested 15 Project Management Tools - This One Actually Works"
- Hook showcasing painful PM tool experiences
- Demo of FlowState solving those pains
- Clear CTA to beta signup

**Instagram** (Carousel - 7 slides)
- Cover: "5 Signs Your PM Tool is Slowing You Down"
- Slides 2-6: Each sign with visual
- Slide 7: "Try FlowState Beta" CTA
- Caption with save-worthy tips

**TikTok** (28s video)
- Hook: "POV: Your PM tool just crashed before standup"
- Fast cuts showing FlowState alternative
- Trending sound about smooth operations
- Comment bait: "What's your PM tool horror story?"

**X** (Thread - 8 tweets)
- Tweet 1: Hook about PM tool frustrations
- Tweets 2-7: Problem → FlowState solution pairs
- Tweet 8: Beta signup link with value prop

**Website** (1800-word landing page)
- SEO-optimized for "simple project management software"
- Problem-Solution-Features-Pricing structure
- Case study snippets
- Multiple CTAs for beta signup
- FAQ section

**Cross-platform strategy:**
- Launch sequence: Website → X thread → YouTube → Instagram → TikTok
- Cross-promotion: Each platform mentions others
- Timing: Stagger over 3 days for momentum

---

### Example 2: Content Repurposing

**Request:**
```
Use the newsroom-editor agent to adapt this blog post to Instagram, TikTok, and X:

[3000-word article: "10 Productivity Myths Debunked"]
```

**Editor delivers:**

**Instagram** (Reel - 60s)
- Hook: "Myth #1 will surprise you..."
- Fast cuts through top 5 myths
- Text overlays for sound-off
- Caption: Full list with "save this" CTA

**TikTok** (3 videos - 30s each)
- Video 1: Myths 1-3 with reactions
- Video 2: Myths 4-6 with examples
- Video 3: Myths 7-10 rapid fire
- "Part X of 3" series strategy

**X** (Thread - 12 tweets)
- Tweet 1: Hook + thread promise
- Tweets 2-11: One myth per tweet
- Tweet 12: Recap + link to full article

**Adaptation notes:**
- Core message maintained across all
- Each platform gets unique angle
- Cross-references drive traffic
- Original article linked strategically

---

### Example 3: Platform Consultation

**Request:**
```
Use the newsroom-editor agent to consult YouTube and Instagram experts:

I have a 45-minute tutorial video. Should I:
A) Post the full video on YouTube
B) Cut it into 5-minute segments
C) Make a highlight reel for Instagram
D) All of the above?
```

**Editor consults experts and delivers:**

**YouTube Creator's advice:**
- Post full 45-min video (depth = authority)
- Create tight 8-min "highlights" version as separate video
- Both versions can rank for different keywords
- Longer video targets serious learners
- Shorter video targets quick browsers

**Instagram Creator's advice:**
- Don't post 45-min to Instagram (wrong platform)
- Create 60s Reel teasing best tip
- Create 10-slide carousel with key takeaways
- Drive traffic to full YouTube video
- Instagram = teaser, YouTube = deep dive

**Editor's strategic recommendation:**
```
Recommended approach:

1. YouTube: Full 45-min tutorial (primary content home)
2. YouTube: 8-min highlights version (entry point)
3. Instagram Reel: 60s "best hack" teaser → full video link
4. Instagram Carousel: 10 key takeaways → full video link
5. X Thread: Summary with 1 tip per tweet → full video link
6. TikTok: 30s "you're doing X wrong" → solution teaser

This maximizes reach while respecting platform contexts.
Full content lives on YouTube, socials drive traffic there.
```

---

## Troubleshooting

### Problem: Content feels generic across platforms

**Cause:** Not leveraging platform specialists' unique expertise

**Solution:**
- Let each creator optimize for their platform
- Don't force identical content
- Trust platform-specific strategies
- Review creator system prompts for platform differences

---

### Problem: Brand voice inconsistent between platforms

**Cause:** Creators not receiving clear brand guidelines

**Solution:**
- Provide detailed brand guidelines in request:
  ```
  Brand voice: Professional but approachable
  Must use: [specific phrases]
  Must avoid: [corporate jargon]
  Examples: [links to approved content]
  ```
- Editor will enforce consistency in review

---

### Problem: Content doesn't match goal

**Cause:** Unclear or missing goal specification

**Solution:**
- Always specify clear goal:
  - Traffic/SEO
  - Engagement
  - Conversion
  - Brand awareness
  - Education
- Include success metrics
- Provide context on why this content matters

---

### Problem: Output quality is low

**Cause:** Generic brief or missing context

**Solution:**
**Good brief includes:**
- ✅ Specific topic (not vague)
- ✅ Clear goal and success criteria
- ✅ Target audience details
- ✅ Tone and voice guidelines
- ✅ Examples of desired output
- ✅ Constraints and requirements

**Bad brief:**
- ❌ "Make content about marketing"
- ❌ No audience specified
- ❌ No goal defined
- ❌ No examples

---

### Problem: Need revisions on content

**Cause:** Normal quality control process

**Solution:**
- Editor will automatically request revisions for scores <75
- You can also request specific changes:
  ```
  The Instagram caption needs a stronger hook and more emojis. Please revise.
  ```
- Maximum 2 revision rounds recommended

---

### Problem: Platform creator not available

**Cause:** File path or agent name issue

**Solution:**
- Verify agents are in `.claude/agents/` directory
- Check agent file names:
  - `newsroom-editor.md`
  - `youtube-creator.md`
  - `instagram-creator.md`
  - `tiktok-creator.md`
  - `x-creator.md`
  - `website-creator.md`
- Ensure YAML frontmatter is correct

---

## Performance Metrics

### Expected Timelines

| Task | Expected Duration |
|------|-------------------|
| Single platform content | 3-5 minutes |
| All 5 platforms (parallel) | 15-30 minutes |
| Content adaptation (3 platforms) | 10-20 minutes |
| Platform consultation | 2-5 minutes |
| Revision round | 3-8 minutes |

### Quality Standards

**Editor scoring:**
- 90-100: Excellent, publish immediately
- 75-89: Good, minor improvements suggested
- 60-74: Needs revision
- <60: Major revision or reassignment

**All content should score 75+** before final delivery.

---

## Advanced Usage

### Custom Platform Mix

Don't need all 5 platforms? Specify:

```
Use the newsroom-editor agent to create content for YouTube and Instagram only about [topic]
```

---

### Sequential vs Parallel

**Parallel (default, faster):**
All platforms created simultaneously

**Sequential (when needed):**
```
Use the newsroom-editor agent to create YouTube content first, then adapt to other platforms after my review
```

---

### Seasonal/Timely Content

Include timing context:

```
Create Black Friday sale content for all platforms
Launch date: November 24
Emphasis: Urgency and limited availability
```

---

### A/B Testing

Request variations:

```
Create 3 different TikTok hooks for the same content - test which performs best
```

---

## Maintenance

### Updating Agents

Agents can be updated as platforms evolve:

1. Edit agent files in `.claude/agents/`
2. Update best practices
3. Add new features (Instagram Threads, X changes, etc.)
4. Maintain YAML frontmatter structure

### Adding New Platforms

To add new platform (e.g., LinkedIn, Pinterest):

1. Create new agent file: `linkedin-creator.md`
2. Follow existing agent structure
3. Define platform expertise
4. Update newsroom-editor to include new platform
5. Test with sample content

---

## Resources

### Related Documentation

- [Subagent System Guide](./SUBAGENT-SYSTEM-GUIDE.md) - Architecture and design patterns
- Agent files in `.claude/agents/` - Individual agent system prompts

### Platform Resources

- **YouTube:** Creator Academy, VidIQ, TubeBuddy
- **Instagram:** Creator Studio, Meta Business Suite
- **TikTok:** Creative Center, Trending sounds
- **X:** X Analytics, X Creative Studio
- **Website:** Google Analytics, Search Console, SEMrush

---

## Conclusion

The Multi-Platform Content Newsroom System enables:

✅ **Efficient Content Production** - Create for 5 platforms in 15-30 minutes
✅ **Platform Optimization** - Each piece crafted for its platform's algorithm
✅ **Quality Control** - Editor review ensures excellence
✅ **Brand Consistency** - Unified voice across channels
✅ **Strategic Flexibility** - Create, adapt, or consult as needed

### Getting Started

**First campaign:**
```
Use the newsroom-editor agent to create content about [your topic] for all platforms.

Goal: [your goal]
Tone: [your brand voice]
Target audience: [your audience]
```

**Watch the system:**
- Plan strategy
- Spawn creators in parallel
- Review outputs
- Deliver polished package

**Iterate and improve:**
- Track which platforms perform best
- Learn from creator feedback
- Refine your briefs
- Build your content library

---

**System Version:** 1.0
**Last Updated:** 2025-11-09
**Agents Location:** `.claude/agents/`
**Documentation:** `/r/NEWSROOM-SYSTEM-GUIDE.md`

---

*This system synthesizes content creation best practices across all major platforms with specialized AI agents coordinated by an intelligent editor for maximum efficiency and quality.*
