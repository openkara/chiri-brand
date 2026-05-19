# Chiri Content Agent — Build Specification
**Version:** May 2026
**Purpose:** Technical and product specification for building a fully automated content creation, scheduling, and publishing agent for Chiri AI.

---

## Overview

The Chiri Content Agent is a full-stack web application and AI agent that:
1. Accepts any input (URL, PDF, video, audio, text paste)
2. Analyzes the content and extracts key insights
3. Generates LinkedIn and X posts in the Chiri brand voice (or Mat/Matthew personal voice)
4. Generates a branded image using the Chiri visual template
5. Presents the post and image in a content feed UI
6. Allows the user to schedule, edit, or publish directly to LinkedIn and X
7. Tracks published vs. scheduled vs. draft status

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     INPUT LAYER                                  │
│  URL / PDF / Video / Audio / Text Paste / Podcast RSS           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   CONTENT EXTRACTION LAYER                       │
│  Web scraper / PDF parser / Whisper transcription / RSS reader  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   BRAND VOICE AGENT                              │
│  System prompt: Chiri_Brand_Voice_Skill.md                      │
│  Generates: LinkedIn post, X post, image brief                  │
│  Validates: No em-dashes, no retired terms, curiosity-first     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   IMAGE GENERATION LAYER                         │
│  Template: Chiri visual brand system                            │
│  Output: 1080x1080px branded quote image                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   CONTENT FEED UI                                │
│  Card-based feed with copy, image, channel, status              │
│  Actions: Edit / Schedule / Publish / Mark as Posted            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   PUBLISHING LAYER                               │
│  LinkedIn API (company page + personal profile)                 │
│  X API (company account + Mat personal)                         │
│  Buffer / Hootsuite as fallback middleware                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Phase 1 — Input Processing

### Supported Input Types

| Input Type | Processing Method | Priority |
|---|---|---|
| URL (article, blog, Substack) | Web scraper + content extraction | P0 |
| Podcast (Spotify, Apple, RSS) | Audio download + Whisper transcription | P0 |
| YouTube video | yt-dlp + Whisper transcription | P0 |
| PDF | PDF text extraction (pdfplumber) | P1 |
| Audio file upload (.mp3, .mp4, .wav) | Whisper transcription | P1 |
| Plain text paste | Direct to brand voice agent | P0 |
| Video file upload | FFmpeg audio extraction + Whisper | P2 |

### Input Processing Flow

```
User drops URL / file / text
    │
    ▼
Content Extractor
    ├── URL → scrape full text + title + author + date
    ├── Podcast URL → extract RSS feed → download audio → transcribe
    ├── YouTube → extract transcript or transcribe audio
    ├── PDF → extract text + metadata
    └── Text → pass directly
    │
    ▼
Content Summary Agent
    Input: raw extracted text
    Output: {
        title: string,
        source: string,
        author: string (if available),
        key_quotes: string[] (3-5 most quotable lines),
        key_insights: string[] (3-5 main ideas),
        relevance_to_chiri: string (why this matters for Chiri's audience),
        suggested_angle: "customer_insight" | "industry_observation" | "founder_story" | "product_philosophy" | "event_content" | "the_divide"
    }
```

---

## Phase 2 — Brand Voice Agent

### System Prompt

The brand voice agent is initialized with `Chiri_Brand_Voice_Skill.md` as its system prompt. This file contains the full Chiri voice guide, Mat's personal voice, Matthew's voice, LinkedIn vs X differences, content categories, and all retired terms.

### Generation Request Schema

```json
{
  "content_summary": { ... },
  "voice": "chiri_company" | "mat_personal" | "matthew_personal",
  "channel": "linkedin" | "twitter" | "both",
  "post_type": "customer_insight" | "industry_observation" | "founder_story" | "product_philosophy" | "event_content" | "the_divide",
  "include_chiri_proof_point": true,
  "source_attribution": {
    "name": "Anton Leicht",
    "title": "Cut Off",
    "url": "https://writing.antonleicht.me/p/cut-off",
    "linkedin_handle": "@Anton Leicht"
  }
}
```

### Generation Output Schema

```json
{
  "linkedin_post": {
    "copy": "string (full post text, no em-dashes)",
    "hook": "string (first line, under 210 chars)",
    "hashtags": ["#AIAdoption", "#FutureOfWork"],
    "tags_to_add": ["@Anton Leicht (search on LinkedIn)"],
    "character_count": 342
  },
  "twitter_post": {
    "copy": "string (under 280 chars or thread)",
    "is_thread": false,
    "thread_tweets": []
  },
  "image_brief": {
    "quote": "string (the single most powerful line for the image)",
    "label": "string (e.g., 'ON AI ADOPTION')",
    "background": "cream" | "dark",
    "attribution": "string (optional, e.g., '— Anton Leicht, May 2026')"
  },
  "validation": {
    "has_em_dashes": false,
    "uses_retired_terms": [],
    "opens_with_question": true,
    "has_chiri_proof_point": true
  }
}
```

### Validation Rules (Auto-enforced)

The agent must check every generated post against these rules before returning output:

1. No em-dashes anywhere in the text
2. No retired terms: "AI Sherpa", "ChiriBrain", "enterprise-grade", "autonomous enterprise"
3. LinkedIn post must open with a question
4. LinkedIn post must include a Chiri proof point if `include_chiri_proof_point` is true
5. No employee ranges (50-1,000 etc.) in any content
6. URL in X posts must always be `chiri.ai` (not `chiri.io`)
7. LinkedIn post character count must be under 3,000

---

## Phase 3 — Image Generation

### Template System

The image generation layer uses the Chiri visual brand template:

**Input:**
```json
{
  "quote": "The people who win in the AI era will not be the ones who learned to talk to AI the right way.",
  "label": "ON PASSIVE LEARNING",
  "background": "cream",
  "attribution": null
}
```

**Processing:**
1. Select background: `#F2EDE6` (cream) or `#1C1A18` (dark)
2. Load Montserrat Bold font
3. Draw orange left-border rule (6px, `#E85D22`)
4. Draw label in orange caps
5. Wrap and draw quote text in large Montserrat Bold
6. Load real Chiri logo (dark version for cream bg, light version for dark bg)
7. Place logo bottom-left
8. Export as 1080x1080px PNG

**Implementation options:**
- Python + Pillow (current implementation — see `generate_all_social_images.py`)
- Node.js + Canvas
- Replicate API with custom template
- Canva API with branded template

### Image Storage
- Store generated images in S3 or equivalent CDN
- Return CDN URL for use in content feed UI
- Retain for 90 days minimum

---

## Phase 4 — Content Feed UI

### UI Components

**Content Card:**
```
┌─────────────────────────────────────────────────────┐
│ [LINKEDIN] Post Title                    [📅 Scheduled: May 26] │
├─────────────────────────────────────────────────────┤
│ POST COPY                    │ [Image Preview]      │
│ ┌─────────────────────────┐  │ [⬇ Download Image]  │
│ │ Post text here...       │  │                      │
│ │ (scrollable, 240px max) │  │                      │
│ └─────────────────────────┘  │                      │
│ [Copy Text]                  │                      │
├─────────────────────────────────────────────────────┤
│ Not yet posted               [📅 Pick date] [Publish Now] │
└─────────────────────────────────────────────────────┘
```

**Post States:**
- **Draft** — White card, date picker + "Publish Now" button
- **Scheduled** — Blue left border, date shown, "Clear" + "Publish Now" buttons
- **Published** — Dimmed, green checkmark, "Mark as Not Published" button

**Feed Views:**
- All posts (default)
- LinkedIn only
- X only
- Scheduled (calendar view)
- Published archive

### Calendar View
- Monthly calendar grid
- Drag-and-drop posts onto dates to schedule
- Color coding: LinkedIn = blue, X = black, Both = orange
- Click any scheduled post to edit or reschedule

---

## Phase 5 — Publishing Layer

### LinkedIn Integration

**Required OAuth scopes:**
- `w_member_social` — post to personal profile
- `w_organization_social` — post to company page
- `r_organization_social` — read company page analytics

**Post to company page:**
```
POST https://api.linkedin.com/v2/ugcPosts
Authorization: Bearer {access_token}
{
  "author": "urn:li:organization:{org_id}",
  "lifecycleState": "PUBLISHED",
  "specificContent": {
    "com.linkedin.ugc.ShareContent": {
      "shareCommentary": { "text": "{post_copy}" },
      "shareMediaCategory": "IMAGE",
      "media": [{
        "status": "READY",
        "media": "urn:li:digitalmediaAsset:{asset_id}"
      }]
    }
  },
  "visibility": { "com.linkedin.ugc.MemberNetworkVisibility": "PUBLIC" }
}
```

**Scheduled posting:** LinkedIn API does not support native scheduling. Use a cron job or Zapier to trigger the API call at the scheduled time.

### X / Twitter Integration

**Required OAuth 2.0 scopes:**
- `tweet.write` — post tweets
- `users.read` — read user profile
- `offline.access` — refresh tokens

**Post a tweet:**
```
POST https://api.twitter.com/2/tweets
Authorization: Bearer {access_token}
{
  "text": "{tweet_copy}",
  "media": { "media_ids": ["{media_id}"] }
}
```

**Scheduled posting:** X API v2 supports scheduled tweets natively via `scheduled_at` parameter.

### Fallback: Buffer API

If direct API integration is not available, use Buffer as middleware:
```
POST https://api.bufferapp.com/1/updates/create.json
{
  "profile_ids": ["{linkedin_profile_id}", "{twitter_profile_id}"],
  "text": "{post_copy}",
  "media": { "photo": "{image_url}" },
  "scheduled_at": "2026-05-26T09:00:00Z"
}
```

---

## Phase 6 — Analytics (Future)

### Metrics to Track

| Metric | Source | Frequency |
|---|---|---|
| LinkedIn impressions | LinkedIn Analytics API | Daily |
| LinkedIn engagement rate | LinkedIn Analytics API | Daily |
| LinkedIn link clicks | LinkedIn Analytics API | Daily |
| X impressions | X Analytics API | Daily |
| X engagement rate | X Analytics API | Daily |
| Landing page traffic by UTM | Google Analytics / Plausible | Daily |
| Email cadence open rates | HubSpot / Mailchimp API | Per send |
| Lead source attribution | CRM API | Real-time |

### UTM Parameter Auto-injection

When generating posts, automatically append UTM parameters to any chiri.ai links:
```
chiri.ai?utm_source=linkedin&utm_medium=social&utm_campaign=passive-learning&utm_content=company-post
```

---

## Tech Stack Recommendation

### Backend
- **Runtime:** Python 3.11 or Node.js 22
- **Framework:** FastAPI (Python) or Express (Node)
- **LLM:** Claude 3.5 Sonnet or GPT-4o via API
- **Transcription:** OpenAI Whisper API or local Whisper model
- **Web scraping:** Playwright + BeautifulSoup
- **Image generation:** Python Pillow (current) or Replicate API
- **Queue:** Redis + Celery (for scheduled publishing jobs)
- **Storage:** S3 or Cloudflare R2 (for images)
- **Database:** PostgreSQL (post history, scheduling, analytics)

### Frontend
- **Framework:** React + TypeScript
- **Styling:** Tailwind CSS
- **Drag-and-drop:** dnd-kit
- **Calendar:** FullCalendar or react-big-calendar
- **State:** Zustand or React Query

### Deployment
- **Hosting:** Manus (current) or Railway / Render
- **Cron jobs:** Manus Schedules or Railway Cron
- **Secrets:** Environment variables for all API keys

---

## Data Model

```sql
-- Posts table
CREATE TABLE posts (
  id UUID PRIMARY KEY,
  title TEXT,
  channel TEXT CHECK (channel IN ('linkedin', 'twitter', 'both')),
  voice TEXT CHECK (voice IN ('chiri_company', 'mat_personal', 'matthew_personal')),
  post_type TEXT,
  copy_linkedin TEXT,
  copy_twitter TEXT,
  image_url TEXT,
  image_filename TEXT,
  tags TEXT[],
  source_url TEXT,
  source_author TEXT,
  status TEXT CHECK (status IN ('draft', 'scheduled', 'published')) DEFAULT 'draft',
  scheduled_at TIMESTAMPTZ,
  published_at TIMESTAMPTZ,
  linkedin_post_id TEXT,
  twitter_post_id TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Analytics table
CREATE TABLE post_analytics (
  id UUID PRIMARY KEY,
  post_id UUID REFERENCES posts(id),
  platform TEXT,
  impressions INTEGER,
  engagements INTEGER,
  link_clicks INTEGER,
  recorded_at TIMESTAMPTZ DEFAULT NOW()
);

-- Source inputs table
CREATE TABLE source_inputs (
  id UUID PRIMARY KEY,
  input_type TEXT,
  raw_url TEXT,
  extracted_text TEXT,
  key_quotes TEXT[],
  key_insights TEXT[],
  processed_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## API Endpoints

```
POST /api/process-input
  Body: { url?, file?, text? }
  Returns: { source_summary, key_quotes, key_insights, suggested_angle }

POST /api/generate-post
  Body: { source_summary, voice, channel, post_type, include_proof_point }
  Returns: { linkedin_post, twitter_post, image_brief, validation }

POST /api/generate-image
  Body: { quote, label, background, attribution? }
  Returns: { image_url, image_filename }

GET /api/posts
  Query: { status?, channel?, limit?, offset? }
  Returns: { posts[], total }

PATCH /api/posts/:id
  Body: { status?, scheduled_at?, copy_linkedin?, copy_twitter? }
  Returns: { post }

POST /api/posts/:id/publish
  Body: { channel: 'linkedin' | 'twitter' | 'both' }
  Returns: { linkedin_post_id?, twitter_post_id?, published_at }

DELETE /api/posts/:id/schedule
  Returns: { post }
```

---

## Agent Workflow (End-to-End)

```
User Input: "Here's a podcast episode about AI adoption"
    │
    ▼
1. Extract: Transcribe podcast audio via Whisper
    │
    ▼
2. Summarize: Extract key quotes, insights, and relevance to Chiri
    │
    ▼
3. Select angle: "industry_observation" (podcast content)
    │
    ▼
4. Generate: LinkedIn post (company voice) + X post
    - Opens with question
    - References podcast and speaker by name
    - Includes Chiri proof point
    - No em-dashes
    - Validated against all brand rules
    │
    ▼
5. Generate image: Most powerful quote from the post
    - Warm cream background
    - Orange left rule
    - Montserrat Bold
    - Real Chiri logo
    │
    ▼
6. Present in content feed UI
    - User sees post copy + image preview
    - User can edit copy inline
    - User picks a date or clicks "Publish Now"
    │
    ▼
7. Publish or schedule
    - LinkedIn API call with image upload
    - X API call with media upload
    - Status updated to "scheduled" or "published"
    │
    ▼
8. Track analytics (future)
    - Pull impressions and engagement 24h after publish
    - Display in analytics dashboard
```

---

## MVP Scope (What to Build First)

**Phase 1 MVP (4-6 weeks):**
- URL input processing (web scraper)
- Brand voice agent (Claude API + Chiri_Brand_Voice_Skill.md system prompt)
- Image generation (Python Pillow template)
- Content feed UI with copy, image, channel, status
- Manual "Mark as Posted" toggle
- Date picker for scheduling (stored in localStorage or DB)

**Phase 2 (6-8 weeks):**
- Podcast/audio transcription (Whisper API)
- LinkedIn API integration (company page publishing)
- X API integration (company account publishing)
- Scheduled publishing via cron job

**Phase 3 (8-12 weeks):**
- Calendar drag-and-drop view
- Mat personal profile publishing
- Analytics dashboard
- UTM auto-injection
- HubSpot/CRM integration

---

## Security and Access Control

- All API keys stored as environment variables, never in code
- LinkedIn and X OAuth tokens stored encrypted in database
- Only authorized users (Kara, Mat) can publish
- All published posts logged with timestamp and user ID
- Image CDN URLs are public (no auth required for viewing)

---

## Files to Load as Agent Context

When initializing the content agent, load these files as context:

1. `Chiri_Brand_Voice_Skill.md` — Full brand voice guide (system prompt)
2. `brand-voice/positioning-notes.md` — Competitive positioning and messaging hierarchy
3. `Chiri_Master_Brand_Handoff.md` — Full brand and content context

---

## Contact and Ownership

- **Product owner:** Kara Burney (kara@chiri.ai)
- **CTO:** Matthew Wimberly
- **GitHub:** github.com/openkara/chiri-brand
- **Content hub (current):** chirilanding-3xipdmvm.manus.space/content-hub.html
- **Landing pages:** chirilanding-3xipdmvm.manus.space
