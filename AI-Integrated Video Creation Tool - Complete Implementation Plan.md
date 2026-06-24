# AI-Integrated Video Creation Tool
## Complete Implementation Plan with Gemini, Veo 3, and FFmpeg

**Date**: January 14, 2025
**Status**: Ready for Development

---

## Executive Summary

This implementation plan combines **AI generation** (Google Gemini/Veo) with **deterministic video processing** (FFmpeg) to create a flexible, cost-effective content creation tool.

### Core Philosophy

**Hybrid AI + Traditional Approach:**
- Use AI where it excels (base video/image generation, artistic text)
- Use traditional tools where they're superior (precision positioning, timeline control, cost)
- Give users choice between AI-generated and user-provided assets

### Key Capabilities

✅ **Generate base videos** with Veo 3 OR use user-provided footage
✅ **Generate base images** with Gemini 2.5 Flash OR use user assets
✅ **Simple text overlays** with Pillow (fast, free, precise)
✅ **Artistic text overlays** with AI + background removal
✅ **Precise positioning** and timeline control via FFmpeg
✅ **Template-driven** JSON-based automation
✅ **Queue-based** scalable processing

---

## Cost Analysis by Use Case

### Scenario 1: Social Media Content (AI Everything)

**Input**: Text prompt → AI generates everything
**Cost Breakdown**:
- Base video (Veo 3.1 Fast, 8s): $2.00
- Title text (Gemini, artistic): $0.04
- Subtitle text (Pillow, simple): $0.00
- Background removal: $0.02
- FFmpeg processing: $0.00
**Total: $2.06 per video**

### Scenario 2: User Footage + AI Text

**Input**: User provides video, wants artistic text overlays
**Cost Breakdown**:
- Base video (user-provided): $0.00
- 3x artistic text overlays: $0.12 ($0.04 each)
- Background removal (3x): $0.06
- FFmpeg processing: $0.00
**Total: $0.18 per video**

### Scenario 3: All Traditional (Maximum Savings)

**Input**: User provides everything, uses Pillow text
**Cost Breakdown**:
- Base video (user-provided): $0.00
- Text overlays (Pillow): $0.00
- FFmpeg processing: $0.00
**Total: $0.00 per video**

### Scenario 4: Hybrid Approach (Recommended)

**Input**: User video + mix of simple and artistic text
**Cost Breakdown**:
- Base video (user-provided): $0.00
- Title (AI artistic): $0.04
- Subtitles (Pillow): $0.00
- Logo overlay: $0.00
- Background removal (1x): $0.02
**Total: $0.06 per video**

---

## Updated Technology Stack

### Core Components (Unchanged)

| Component | Technology | Cost |
|-----------|-----------|------|
| **Language** | Python 3.11+ | Free |
| **Traditional Text** | Pillow 10+ | Free |
| **Video Processing** | FFmpeg 6.1+ | Free |
| **Filter Generation** | typed-ffmpeg | Free |
| **Job Queue** | Celery + Redis | Free (self-hosted) |
| **API Framework** | FastAPI | Free |
| **Database** | PostgreSQL | Free (self-hosted) |

### New AI Components

| Component | Technology | Cost | When to Use |
|-----------|-----------|------|-------------|
| **AI Video** | Veo 3.1 Fast | $0.25/sec | Auto-generate base videos |
| **AI Images** | Gemini 2.5 Flash | FREE (preview) | Generate backgrounds, artistic elements |
| **AI Text** | Gemini 2.5 Flash | $0.04/image | Artistic/stylized text overlays |
| **Background Removal** | Photoroom API | $0.02/image | Remove BG from AI-generated text |
| **Alt: BG Removal** | rembg (local) | FREE | High-volume (>20K/month) |

### Critical SDK Update

⚠️ **IMPORTANT**: Google's legacy SDK ends August 31, 2025

```bash
# OLD (deprecated)
pip install google-generativeai

# NEW (use this)
pip install google-genai
```

---

## Enhanced JSON Template Specification

### Example: AI-Powered Social Media Video

```json
{
  "template_version": "2.0",
  "video": {
    "source": {
      "type": "ai_generated",
      "provider": "veo_3",
      "prompt": "Cinematic dolly shot of modern workspace, natural lighting, professional atmosphere",
      "duration": 8,
      "model": "veo-3.1-fast",
      "aspect_ratio": "16:9"
    },
    "fallback": {
      "type": "file",
      "path": "s3://bucket/default-background.mp4"
    },
    "output_preset": "1080p_social"
  },
  "overlays": [
    {
      "type": "text",
      "id": "title",
      "content": "{{video_title}}",
      "rendering": {
        "method": "ai",
        "provider": "gemini_flash",
        "prompt_template": "Bold stylized text '{{content}}' in modern sans-serif, gradient from blue to purple, white background for removal",
        "fallback_method": "pillow"
      },
      "background_removal": {
        "enabled": true,
        "method": "photoroom_api",
        "fallback_method": "rembg"
      },
      "position": {
        "preset": "TOP_CENTER",
        "offset": {"x": 0, "y": 100}
      },
      "timeline": {
        "start": 1.0,
        "end": 6.0,
        "animation": {
          "in": {"type": "fade", "duration": 0.5},
          "out": {"type": "fade", "duration": 0.5}
        }
      }
    },
    {
      "type": "text",
      "id": "subtitle",
      "content": "{{description}}",
      "rendering": {
        "method": "pillow",
        "style": {
          "font": "fonts/OpenSans-Regular.ttf",
          "size": 36,
          "color": "#FFFFFF",
          "outline": {
            "color": "#000000",
            "width": 2
          }
        }
      },
      "position": {
        "preset": "BOTTOM_CENTER",
        "offset": {"x": 0, "y": -80}
      },
      "timeline": {
        "start": 2.0,
        "end": 7.5
      }
    },
    {
      "type": "image",
      "id": "logo",
      "source": {
        "type": "ai_generated",
        "provider": "gemini_flash",
        "prompt": "Minimalist tech company logo, clean design, transparent background",
        "cache_key": "brand_logo_v1"
      },
      "fallback": {
        "type": "file",
        "path": "s3://bucket/brand-logo.png"
      },
      "position": {
        "preset": "BOTTOM_RIGHT",
        "offset": {"x": -30, "y": -30}
      },
      "timeline": {
        "start": 0,
        "end": 8
      },
      "transform": {
        "scale": 0.25,
        "opacity": 0.9
      }
    }
  ],
  "variables": {
    "video_title": "Launch Your Product",
    "description": "Available Now"
  },
  "cost_optimization": {
    "cache_ai_assets": true,
    "cache_duration_hours": 168,
    "prefer_free_tier": true,
    "max_cost_per_video": 2.50
  }
}
```

### Example: Hybrid Video (User Footage + AI Text)

```json
{
  "template_version": "2.0",
  "video": {
    "source": {
      "type": "file",
      "path": "{{user_video_path}}"
    },
    "output_preset": "1080p_social"
  },
  "overlays": [
    {
      "type": "text",
      "id": "artistic_title",
      "content": "{{title}}",
      "rendering": {
        "method": "ai",
        "provider": "gemini_flash",
        "prompt_template": "Graffiti-style text '{{content}}' with dripping paint effect, vibrant colors, white background",
        "cost_limit": 0.05
      },
      "background_removal": {
        "enabled": true,
        "method": "auto"
      },
      "position": {
        "preset": "CENTER"
      },
      "timeline": {
        "start": 2.0,
        "end": 8.0
      }
    },
    {
      "type": "text",
      "id": "simple_subtitle",
      "content": "{{subtitle}}",
      "rendering": {
        "method": "pillow",
        "style": {
          "font": "fonts/Roboto-Bold.ttf",
          "size": 48,
          "color": "#FFFFFF"
        }
      },
      "position": {
        "preset": "BOTTOM_CENTER",
        "offset": {"x": 0, "y": -100}
      },
      "timeline": {
        "start": 0,
        "end": 10
      }
    }
  ],
  "variables": {
    "user_video_path": "uploads/user_video_123.mp4",
    "title": "Epic Gaming Moment",
    "subtitle": "Watch Until The End!"
  }
}
```

---

## Updated System Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                      CLIENT LAYER                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │   REST   │  │ GraphQL  │  │WebSocket │  │   CLI    │       │
│  │   API    │  │   API    │  │(Progress)│  │  Tool    │       │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘       │
└───────┼─────────────┼─────────────┼─────────────┼─────────────┘
        │             │             │             │
┌───────▼─────────────▼─────────────▼─────────────▼─────────────┐
│                  API GATEWAY (FastAPI)                         │
│  • JWT Authentication                                          │
│  • Rate Limiting (Tier-based)                                  │
│  • Cost Tracking & Budgets                                     │
└───────┬────────────────────────────────────────────────────────┘
        │
┌───────▼────────────────────────────────────────────────────────┐
│                   BUSINESS LOGIC LAYER                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   Template   │  │     Asset    │  │     Cost     │         │
│  │   Manager    │  │    Manager   │  │   Optimizer  │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└───────┬────────────────────────────────────────────────────────┘
        │
┌───────▼────────────────────────────────────────────────────────┐
│              JOB QUEUE (Celery + Redis)                        │
│  ┌──────────────────────────────────────────────────┐          │
│  │  Task Types:                                      │          │
│  │  • generate_ai_video (Veo 3)                      │          │
│  │  • generate_ai_image (Gemini)                     │          │
│  │  • generate_ai_text (Gemini)                      │          │
│  │  • remove_background (Photoroom/rembg)            │          │
│  │  • render_traditional_text (Pillow)               │          │
│  │  • composite_video (FFmpeg)                       │          │
│  │  • validate_output (VMAF/ffprobe)                 │          │
│  └──────────────────────────────────────────────────┘          │
└───────┬────────────────────────────────────────────────────────┘
        │
┌───────▼────────────────────────────────────────────────────────┐
│                      WORKER LAYER                              │
│  ┌─────────────────────────────────────────────────────┐       │
│  │          HYBRID AI + TRADITIONAL PIPELINE            │       │
│  │                                                       │       │
│  │  ┌────────────────────────────────────────────────┐ │       │
│  │  │  1. AI GENERATION STAGE (Optional)             │ │       │
│  │  │                                                 │ │       │
│  │  │  Video Generator (Veo 3):                      │ │       │
│  │  │    • google-genai SDK                          │ │       │
│  │  │    • Prompt engineering                        │ │       │
│  │  │    • Quality validation (Gemini 2.5 Pro)       │ │       │
│  │  │    • Retry with exponential backoff            │ │       │
│  │  │    • Cost tracking                             │ │       │
│  │  │                                                 │ │       │
│  │  │  Image/Text Generator (Gemini 2.5 Flash):      │ │       │
│  │  │    • Stylized text rendering                   │ │       │
│  │  │    • Background generation                     │ │       │
│  │  │    • Logo/asset creation                       │ │       │
│  │  │    • Free tier optimization                    │ │       │
│  │  │                                                 │ │       │
│  │  │  Background Remover:                           │ │       │
│  │  │    • Photoroom API ($0.02/image)               │ │       │
│  │  │    • rembg fallback (self-hosted)              │ │       │
│  │  │    • Quality validation                        │ │       │
│  │  └────────────────────────────────────────────────┘ │       │
│  │                                                       │       │
│  │  ┌────────────────────────────────────────────────┐ │       │
│  │  │  2. TRADITIONAL RENDERING STAGE                │ │       │
│  │  │                                                 │ │       │
│  │  │  Text Renderer (Pillow):                       │ │       │
│  │  │    • High-performance RGBA rendering           │ │       │
│  │  │    • Custom fonts, shadows, outlines           │ │       │
│  │  │    • Precise bounding box calculation          │ │       │
│  │  │    • 15-40x faster than FFmpeg drawtext        │ │       │
│  │  └────────────────────────────────────────────────┘ │       │
│  │                                                       │       │
│  │  ┌────────────────────────────────────────────────┐ │       │
│  │  │  3. ASSET MANAGEMENT STAGE                     │ │       │
│  │  │                                                 │ │       │
│  │  │  Smart Cache (Redis):                          │ │       │
│  │  │    • AI asset caching (7-day default)          │ │       │
│  │  │    • Pillow render caching                     │ │       │
│  │  │    • LRU eviction                              │ │       │
│  │  │    • Cache hit tracking (>70% target)          │ │       │
│  │  │                                                 │ │       │
│  │  │  Asset Optimizer:                              │ │       │
│  │  │    • PNG compression (pngquant)                │ │       │
│  │  │    • Format conversion (WebP for web)          │ │       │
│  │  │    • Resolution optimization                   │ │       │
│  │  └────────────────────────────────────────────────┘ │       │
│  │                                                       │       │
│  │  ┌────────────────────────────────────────────────┐ │       │
│  │  │  4. VIDEO COMPOSITION STAGE                    │ │       │
│  │  │                                                 │ │       │
│  │  │  FFmpeg Compositor:                            │ │       │
│  │  │    • Filter complex builder (typed-ffmpeg)     │ │       │
│  │  │    • Multi-layer overlay support               │ │       │
│  │  │    • Timeline control (enable expressions)     │ │       │
│  │  │    • Hardware acceleration (NVENC)             │ │       │
│  │  │    • Progress monitoring                       │ │       │
│  │  └────────────────────────────────────────────────┘ │       │
│  │                                                       │       │
│  │  ┌────────────────────────────────────────────────┐ │       │
│  │  │  5. QUALITY VALIDATION STAGE                   │ │       │
│  │  │                                                 │ │       │
│  │  │    • FFprobe metadata check                    │ │       │
│  │  │    • VMAF quality scoring (>85 target)         │ │       │
│  │  │    • Positioning accuracy validation           │ │       │
│  │  │    • Timeline sync verification                │ │       │
│  │  │    • Cost vs quality optimization              │ │       │
│  │  └────────────────────────────────────────────────┘ │       │
│  └─────────────────────────────────────────────────────┘       │
└───────┬────────────────────────────────────────────────────────┘
        │
┌───────▼────────────────────────────────────────────────────────┐
│                      STORAGE LAYER                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │PostgreSQL│  │  Redis   │  │    S3    │  │   CDN    │       │
│  │(Metadata)│  │ (Cache)  │  │ (Videos) │  │(Delivery)│       │
│  │+ Costs   │  │+ AI cache│  │          │  │          │       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
└────────────────────────────────────────────────────────────────┘
```

---

## Implementation Phases (Updated)

### Phase 1: Core Foundation (Weeks 1-3)

**Goal**: Traditional rendering pipeline (Pillow + FFmpeg)

**Deliverables**:
- [x] Project structure
- [x] FFmpeg subprocess wrapper
- [x] Pillow text renderer
- [x] Basic overlay compositor
- [x] Unit tests
- [x] Docker container

**New Additions**:
- [ ] Google Cloud setup (API keys, billing)
- [ ] google-genai SDK integration
- [ ] Cost tracking foundation
- [ ] Environment variable management

### Phase 2: AI Generation Integration (Weeks 4-6) **NEW**

**Goal**: Add Gemini and Veo 3 generation capabilities

**Deliverables**:
- [ ] Veo 3 video generator
  - [ ] google-genai client setup
  - [ ] Prompt engineering templates
  - [ ] Quality validation with Gemini 2.5 Pro
  - [ ] Exponential backoff retry logic
  - [ ] Cost tracking per generation

- [ ] Gemini 2.5 Flash image generator
  - [ ] Text-to-image generation
  - [ ] Stylized text rendering
  - [ ] Background asset generation
  - [ ] Free tier optimization

- [ ] Background removal service
  - [ ] Photoroom API integration
  - [ ] rembg fallback (local)
  - [ ] Quality validation
  - [ ] Cost comparison tracking

**Key Classes**:
```python
# ai/veo_generator.py
class VeoVideoGenerator:
    def generate(self, prompt: str, duration: int = 8) -> Path
    def extend_video(self, video: Path, continuation_prompt: str) -> Path
    def validate_quality(self, video: Path) -> QualityScore

# ai/gemini_generator.py
class GeminiImageGenerator:
    def generate_image(self, prompt: str) -> Path
    def generate_stylized_text(self, text: str, style: str) -> Path
    def optimize_for_free_tier(self) -> bool

# ai/background_removal.py
class BackgroundRemover:
    def remove_photoroom(self, image: Path) -> Path
    def remove_rembg(self, image: Path) -> Path
    def auto_select_method(self, volume: int) -> str
```

**Success Criteria**:
- Generate 8s Veo video in <60 seconds
- Generate stylized text image in <5 seconds
- Remove background in <2 seconds
- Track costs accurately (±$0.01)

### Phase 3: Template System Enhancement (Weeks 7-9)

**Goal**: Support AI + traditional rendering in templates

**Deliverables**:
- [ ] Enhanced Pydantic schemas (v2.0)
- [ ] AI source type support (`ai_generated`, `file`, `url`)
- [ ] Rendering method selection (`ai`, `pillow`)
- [ ] Fallback strategies (AI → traditional)
- [ ] Cost optimization rules
- [ ] Asset caching with TTL

**Template Features**:
```python
# templates/models.py (v2.0)
class VideoSource(BaseModel):
    type: Literal["file", "url", "ai_generated"]
    provider: Optional[str]  # "veo_3", "user"
    prompt: Optional[str]
    fallback: Optional[Dict]

class TextRendering(BaseModel):
    method: Literal["pillow", "ai"]
    provider: Optional[str]  # "gemini_flash"
    prompt_template: Optional[str]
    fallback_method: Optional[str]
    cost_limit: Optional[float]

class BackgroundRemoval(BaseModel):
    enabled: bool = False
    method: Literal["photoroom_api", "rembg", "auto"]
    fallback_method: Optional[str]
```

**Success Criteria**:
- Process AI + traditional overlays in single template
- Automatic fallback on AI failure
- Cost limits enforced (<$2.50/video default)
- Cache AI assets (>70% hit rate after warmup)

### Phase 4: Performance & Caching (Weeks 10-11)

**Goal**: Optimize AI calls and FFmpeg processing

**Deliverables**:
- [ ] Redis-based AI asset cache
- [ ] Intelligent cache key generation (prompt hash)
- [ ] NVENC hardware acceleration
- [ ] Parallel AI generation (batch)
- [ ] Progress tracking (multi-stage)
- [ ] Cost monitoring dashboard

**Caching Strategy**:
```python
# cache/ai_cache.py
class AIAssetCache:
    def get_or_generate_video(
        self,
        prompt: str,
        generator: Callable,
        ttl_hours: int = 168  # 7 days default
    ) -> Path

    def get_or_generate_image(
        self,
        prompt: str,
        generator: Callable,
        ttl_hours: int = 168
    ) -> Path

    def track_cost_savings(self) -> Dict[str, float]
```

**Performance Targets**:
- AI video cache hit: >50% for reused prompts
- AI image cache hit: >70% for brand assets
- FFmpeg with NVENC: <10s for 30s video
- Total pipeline: <90s for AI-generated video

### Phase 5: Queue & API (Weeks 12-14)

**Goal**: Production async processing with cost controls

**Deliverables**:
- [ ] Celery task queue
- [ ] FastAPI endpoints (v2)
- [ ] Job status tracking (multi-stage)
- [ ] Cost tracking per job
- [ ] Budget limits (per user/API key)
- [ ] Webhook notifications
- [ ] Rate limiting (tier-based)

**Enhanced API Endpoints**:
```python
# api/routes.py (v2)
POST   /v2/jobs                     # Create job (AI or traditional)
GET    /v2/jobs/{job_id}            # Get status + cost tracking
GET    /v2/jobs/{job_id}/result     # Get video + cost breakdown
DELETE /v2/jobs/{job_id}            # Cancel job + refund tracking

POST   /v2/templates                # Upload template (v2.0 schema)
GET    /v2/templates                # List templates
GET    /v2/templates/{id}/preview   # Preview with cost estimate

GET    /v2/usage                    # Usage stats + costs
GET    /v2/usage/costs              # Detailed cost breakdown
POST   /v2/usage/budgets            # Set budget limits
```

**Budget Control**:
```python
# api/cost_control.py
class CostController:
    def estimate_job_cost(self, template: VideoTemplate) -> float
    def check_budget(self, user_id: str, estimated_cost: float) -> bool
    def track_actual_cost(self, job_id: str, breakdown: Dict) -> None
    def get_user_spending(self, user_id: str, period: str) -> float
```

**Success Criteria**:
- Process 10 concurrent AI jobs
- Accurate cost estimation (±10%)
- Budget limits enforced
- API response <200ms (status checks)

### Phase 6: Production Hardening (Weeks 15-16)

**Goal**: Monitoring, logging, documentation

**Deliverables**:
- [ ] Structured logging (JSON)
- [ ] Prometheus metrics (AI + FFmpeg)
- [ ] Grafana dashboards
- [ ] Error tracking (Sentry)
- [ ] Cost alerting (budget overruns)
- [ ] API documentation (OpenAPI)
- [ ] User guide (MkDocs)

**New Metrics**:
- `ai_generation_cost_dollars` (counter)
- `ai_generation_duration_seconds` (histogram)
- `ai_cache_hit_rate` (gauge)
- `background_removal_cost_dollars` (counter)
- `user_spending_total` (counter by user_id)

**Alert Rules**:
```yaml
alerts:
  - name: HighAICost
    condition: ai_generation_cost_dollars > 100 per hour
    severity: critical

  - name: UserBudgetExceeded
    condition: user_spending_total > user_budget
    severity: warning

  - name: LowAICacheHitRate
    condition: ai_cache_hit_rate < 0.5
    severity: info
```

---

## Project Structure (Updated)

```
ai-video-creator/
├── README.md
├── pyproject.toml
├── docker-compose.yml
├── Dockerfile
├── .env.example
│
├── src/
│   ├── __init__.py
│   │
│   ├── ai/                         # NEW: AI generation
│   │   ├── __init__.py
│   │   ├── veo_generator.py        # Veo 3 video generation
│   │   ├── gemini_generator.py     # Gemini image/text generation
│   │   ├── background_removal.py   # Photoroom + rembg
│   │   ├── quality_validator.py    # AI quality assessment
│   │   └── cost_tracker.py         # Per-API cost tracking
│   │
│   ├── core/                       # Traditional rendering
│   │   ├── __init__.py
│   │   ├── text_renderer.py        # Pillow-based text
│   │   ├── ffmpeg_compositor.py    # FFmpeg wrapper
│   │   ├── pipeline.py             # Main rendering pipeline
│   │   └── validators.py
│   │
│   ├── templates/                  # Template system (v2.0)
│   │   ├── __init__.py
│   │   ├── models.py               # Enhanced Pydantic schemas
│   │   ├── parser.py
│   │   ├── renderer.py             # Hybrid AI + traditional
│   │   └── cost_estimator.py       # NEW: Estimate before render
│   │
│   ├── cache/                      # Enhanced caching
│   │   ├── __init__.py
│   │   ├── asset_cache.py          # Redis cache
│   │   ├── ai_cache.py             # NEW: AI-specific caching
│   │   └── strategies.py
│   │
│   ├── queue/
│   │   ├── __init__.py
│   │   ├── celery_app.py
│   │   ├── tasks.py                # Enhanced with AI tasks
│   │   └── monitoring.py
│   │
│   ├── api/                        # API v2
│   │   ├── __init__.py
│   │   ├── app.py
│   │   ├── routes/
│   │   │   ├── jobs.py             # Enhanced with cost tracking
│   │   │   ├── templates.py        # v2.0 schema support
│   │   │   ├── usage.py            # NEW: Usage and billing
│   │   │   └── webhooks.py
│   │   ├── auth.py
│   │   ├── rate_limit.py
│   │   └── cost_control.py         # NEW: Budget enforcement
│   │
│   ├── storage/
│   │   ├── __init__.py
│   │   ├── s3_storage.py
│   │   ├── local_storage.py
│   │   └── cdn.py
│   │
│   └── utils/
│       ├── __init__.py
│       ├── logging.py
│       ├── metrics.py              # Enhanced with AI metrics
│       ├── config.py
│       └── retry.py                # NEW: Exponential backoff
│
├── tests/
│   ├── unit/
│   │   ├── test_ai_generation.py
│   │   ├── test_background_removal.py
│   │   └── test_cost_tracking.py
│   ├── integration/
│   │   └── test_hybrid_pipeline.py
│   └── fixtures/
│       ├── templates/
│       │   ├── ai_video.json
│       │   ├── hybrid.json
│       │   └── traditional.json
│       └── media/
│
├── docs/
│   ├── index.md
│   ├── ai-integration.md           # NEW: AI features guide
│   ├── cost-optimization.md        # NEW: Cost best practices
│   ├── template-v2-guide.md        # NEW: v2.0 template docs
│   └── deployment.md
│
├── config/
│   ├── prompts/                    # NEW: Prompt templates
│   │   ├── video_generation.yaml
│   │   ├── text_styling.yaml
│   │   └── background_generation.yaml
│   └── cost_limits.yaml            # NEW: Default budget rules
│
└── templates/                      # Example templates
    ├── ai-social-post.json
    ├── hybrid-tutorial.json
    └── traditional-product.json
```

---

## Code Examples

### 1. Veo 3 Video Generation

```python
# src/ai/veo_generator.py
from google import genai
from pathlib import Path
import time
from typing import Optional

class VeoVideoGenerator:
    def __init__(self, api_key: str):
        self.client = genai.Client(api_key=api_key)
        self.model = "veo-3.1-fast"
        self.cost_per_second = 0.25  # Updated pricing

    def generate(
        self,
        prompt: str,
        duration: int = 8,
        aspect_ratio: str = "16:9"
    ) -> tuple[Path, float]:
        """
        Generate video with Veo 3.

        Returns:
            (video_path, cost_usd)
        """
        try:
            # Generate video
            response = self.client.models.generate_videos(
                model=self.model,
                prompt=prompt,
                config={
                    "duration": f"{duration}s",
                    "aspect_ratio": aspect_ratio,
                    "include_audio": True
                }
            )

            # Poll for completion (async in production)
            video_data = self._wait_for_completion(response.operation_id)

            # Save to disk
            output_path = Path(f"/tmp/veo_{int(time.time())}.mp4")
            output_path.write_bytes(video_data)

            # Calculate cost
            cost = duration * self.cost_per_second

            return output_path, cost

        except Exception as e:
            raise VideoGenerationError(f"Veo generation failed: {e}")

    def _wait_for_completion(
        self,
        operation_id: str,
        timeout: int = 600
    ) -> bytes:
        """Poll operation until complete."""
        start = time.time()
        while time.time() - start < timeout:
            status = self.client.operations.get(operation_id)
            if status.done:
                return status.response.video_data
            time.sleep(5)
        raise TimeoutError("Video generation timeout")
```

### 2. Gemini Image/Text Generation

```python
# src/ai/gemini_generator.py
from google import genai
from pathlib import Path
import hashlib

class GeminiImageGenerator:
    def __init__(self, api_key: str):
        self.client = genai.Client(api_key=api_key)
        self.model = "gemini-2.5-flash"
        self.cost_per_image = 0.0  # FREE during preview

    def generate_stylized_text(
        self,
        text: str,
        style: str = "modern bold gradient",
        background: str = "white"
    ) -> tuple[Path, float]:
        """
        Generate artistic text image.

        Args:
            text: The text to render
            style: Style description
            background: Background color (for removal)

        Returns:
            (image_path, cost_usd)
        """
        prompt = f"""
        Create stylized text artwork with these specifications:

        Text: "{text}"
        Style: {style}
        Background: Solid {background} background for easy removal

        Requirements:
        - Text must be clearly legible
        - High contrast against background
        - Professional quality
        - Centered composition
        """

        try:
            response = self.client.models.generate_images(
                model=self.model,
                prompt=prompt,
                config={
                    "number_of_images": 1,
                    "aspect_ratio": "16:9",
                    "safety_filter_level": "block_only_high"
                }
            )

            # Save image
            image_data = response.generated_images[0].image.data
            output_path = Path(f"/tmp/gemini_text_{self._hash_prompt(prompt)}.png")
            output_path.write_bytes(image_data)

            return output_path, self.cost_per_image

        except Exception as e:
            raise ImageGenerationError(f"Gemini generation failed: {e}")

    def _hash_prompt(self, prompt: str) -> str:
        """Create cache key from prompt."""
        return hashlib.md5(prompt.encode()).hexdigest()[:12]
```

### 3. Background Removal

```python
# src/ai/background_removal.py
import requests
from pathlib import Path
from rembg import remove
from PIL import Image
import io

class BackgroundRemover:
    def __init__(self, photoroom_api_key: Optional[str] = None):
        self.photoroom_api_key = photoroom_api_key
        self.photoroom_cost = 0.02
        self.rembg_cost = 0.0  # Free (self-hosted)

    def remove(
        self,
        image_path: Path,
        method: str = "auto"
    ) -> tuple[Path, float]:
        """
        Remove background from image.

        Args:
            image_path: Input image
            method: "photoroom_api", "rembg", or "auto"

        Returns:
            (transparent_image_path, cost_usd)
        """
        if method == "auto":
            method = "photoroom_api" if self.photoroom_api_key else "rembg"

        if method == "photoroom_api":
            return self._remove_photoroom(image_path)
        elif method == "rembg":
            return self._remove_rembg(image_path)
        else:
            raise ValueError(f"Unknown method: {method}")

    def _remove_photoroom(self, image_path: Path) -> tuple[Path, float]:
        """Use Photoroom API (fast, paid)."""
        url = "https://sdk.photoroom.com/v1/segment"

        with open(image_path, "rb") as f:
            response = requests.post(
                url,
                files={"image_file": f},
                headers={"x-api-key": self.photoroom_api_key}
            )

        response.raise_for_status()

        output_path = image_path.with_suffix(".transparent.png")
        output_path.write_bytes(response.content)

        return output_path, self.photoroom_cost

    def _remove_rembg(self, image_path: Path) -> tuple[Path, float]:
        """Use rembg library (free, slower)."""
        input_image = Image.open(image_path)
        output_image = remove(input_image)

        output_path = image_path.with_suffix(".transparent.png")
        output_image.save(output_path)

        return output_path, self.rembg_cost
```

### 4. Hybrid Pipeline Orchestrator

```python
# src/core/pipeline.py (enhanced)
from dataclasses import dataclass
from pathlib import Path
from typing import List, Dict

@dataclass
class PipelineCost:
    video_generation: float = 0.0
    image_generation: float = 0.0
    background_removal: float = 0.0
    total: float = 0.0

class HybridRenderPipeline:
    def __init__(
        self,
        veo_generator: VeoVideoGenerator,
        gemini_generator: GeminiImageGenerator,
        bg_remover: BackgroundRemover,
        text_renderer: TextRenderer,
        ffmpeg_compositor: FFmpegCompositor
    ):
        self.veo = veo_generator
        self.gemini = gemini_generator
        self.bg_remover = bg_remover
        self.text_renderer = text_renderer
        self.ffmpeg = ffmpeg_compositor
        self.cost = PipelineCost()

    def process(self, template: VideoTemplate) -> tuple[Path, PipelineCost]:
        """
        Process video template with hybrid AI + traditional rendering.

        Returns:
            (output_video_path, cost_breakdown)
        """
        # Stage 1: Get base video
        if template.video.source.type == "ai_generated":
            base_video, cost = self.veo.generate(
                prompt=template.video.source.prompt,
                duration=template.video.source.duration
            )
            self.cost.video_generation = cost
        else:
            base_video = Path(template.video.source.path)

        # Stage 2: Generate overlays
        overlay_assets = []

        for overlay in template.overlays:
            if overlay.type == "text":
                asset = self._process_text_overlay(overlay)
                overlay_assets.append((asset, overlay))

        # Stage 3: Composite with FFmpeg
        output_path = self.ffmpeg.composite(
            base_video,
            overlay_assets,
            output_preset=template.video.output_preset
        )

        self.cost.total = sum([
            self.cost.video_generation,
            self.cost.image_generation,
            self.cost.background_removal
        ])

        return output_path, self.cost

    def _process_text_overlay(self, overlay: TextOverlay) -> Path:
        """Generate text overlay (AI or traditional)."""
        if overlay.rendering.method == "ai":
            # AI-generated stylized text
            image_path, cost = self.gemini.generate_stylized_text(
                text=overlay.content,
                style=overlay.rendering.prompt_template
            )
            self.cost.image_generation += cost

            # Remove background if requested
            if overlay.background_removal.enabled:
                image_path, cost = self.bg_remover.remove(
                    image_path,
                    method=overlay.background_removal.method
                )
                self.cost.background_removal += cost

            return image_path

        else:
            # Traditional Pillow rendering
            return self.text_renderer.render_to_png(
                text=overlay.content,
                style=overlay.rendering.style
            )
```

### 5. Cost Estimation

```python
# src/templates/cost_estimator.py
class CostEstimator:
    def estimate(self, template: VideoTemplate) -> Dict[str, float]:
        """
        Estimate cost before rendering.

        Returns:
            {
                "video_generation": 2.00,
                "image_generation": 0.12,
                "background_removal": 0.06,
                "total": 2.18,
                "cached_savings": 0.50
            }
        """
        costs = {
            "video_generation": 0.0,
            "image_generation": 0.0,
            "background_removal": 0.0,
            "cached_savings": 0.0
        }

        # Base video cost
        if template.video.source.type == "ai_generated":
            duration = template.video.source.duration
            costs["video_generation"] = duration * 0.25  # Veo 3.1 Fast

        # Overlay costs
        for overlay in template.overlays:
            if overlay.rendering.method == "ai":
                # Check cache first
                cache_key = self._get_cache_key(overlay)
                if self._is_cached(cache_key):
                    costs["cached_savings"] += 0.04
                else:
                    costs["image_generation"] += 0.04  # Gemini (if not free)

                if overlay.background_removal.enabled:
                    if overlay.background_removal.method == "photoroom_api":
                        costs["background_removal"] += 0.02

        costs["total"] = sum([
            costs["video_generation"],
            costs["image_generation"],
            costs["background_removal"]
        ])

        return costs
```

---

## API Usage Examples

### Example 1: Generate AI Video with Traditional Text

```bash
curl -X POST "https://api.yourservice.com/v2/jobs" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "video": {
        "source": {
          "type": "ai_generated",
          "provider": "veo_3",
          "prompt": "Professional office workspace, modern design, natural lighting",
          "duration": 8
        }
      },
      "overlays": [
        {
          "type": "text",
          "content": "Welcome to Our Company",
          "rendering": {
            "method": "pillow",
            "style": {
              "font": "fonts/Montserrat-Bold.ttf",
              "size": 72,
              "color": "#FFFFFF"
            }
          },
          "position": {"preset": "CENTER"},
          "timeline": {"start": 2.0, "end": 6.0}
        }
      ]
    }
  }'
```

**Response**:
```json
{
  "job_id": "job_abc123",
  "status": "processing",
  "estimated_cost": 2.00,
  "estimated_duration_seconds": 70,
  "created_at": "2025-01-14T10:00:00Z"
}
```

### Example 2: User Video + AI Stylized Text

```bash
curl -X POST "https://api.yourservice.com/v2/jobs" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "template": {
      "video": {
        "source": {
          "type": "file",
          "path": "uploads/user_video.mp4"
        }
      },
      "overlays": [
        {
          "type": "text",
          "content": "EPIC MOMENT",
          "rendering": {
            "method": "ai",
            "provider": "gemini_flash",
            "prompt_template": "Bold graffiti-style text \"{{content}}\" with paint drips, vibrant colors, white background"
          },
          "background_removal": {
            "enabled": true,
            "method": "photoroom_api"
          },
          "position": {"preset": "TOP_CENTER"},
          "timeline": {"start": 1.0, "end": 5.0}
        }
      ]
    }
  }'
```

**Response**:
```json
{
  "job_id": "job_def456",
  "status": "processing",
  "estimated_cost": 0.06,
  "cost_breakdown": {
    "video_generation": 0.00,
    "image_generation": 0.04,
    "background_removal": 0.02
  },
  "estimated_duration_seconds": 15
}
```

### Example 3: Get Cost Estimate Before Rendering

```bash
curl -X POST "https://api.yourservice.com/v2/templates/estimate" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{...template...}'
```

**Response**:
```json
{
  "estimated_cost": 2.18,
  "breakdown": {
    "video_generation": 2.00,
    "image_generation": 0.12,
    "background_removal": 0.06
  },
  "cached_savings": 0.50,
  "effective_cost": 1.68
}
```

---

## Cost Optimization Strategies

### 1. Aggressive Caching

```python
# Cache AI assets for 7 days (default)
CACHE_CONFIG = {
    "veo_videos": {
        "ttl_hours": 168,  # 7 days
        "enabled": True
    },
    "gemini_images": {
        "ttl_hours": 168,
        "enabled": True
    },
    "background_removed": {
        "ttl_hours": 336,  # 14 days (static assets)
        "enabled": True
    }
}
```

**Savings**: 70% cost reduction after warmup period

### 2. Free Tier Optimization

```python
# Use Gemini 2.5 Flash (FREE during preview)
if overlay.rendering.provider == "gemini_flash":
    # Free tier has rate limits
    await rate_limiter.acquire("gemini_free", rpm=15)
    cost = 0.0  # FREE
else:
    # Paid tier (Imagen 3)
    cost = 0.03
```

**Savings**: 100% on image generation during preview

### 3. Batch Processing

```python
# Generate multiple overlays in parallel
async def generate_overlays_batch(overlays: List[TextOverlay]):
    tasks = [
        gemini.generate_stylized_text(o.content, o.style)
        for o in overlays
    ]
    return await asyncio.gather(*tasks)
```

**Savings**: 40% time reduction, better resource utilization

### 4. Smart Fallbacks

```yaml
# config/cost_limits.yaml
fallback_strategy:
  max_cost_per_video: 2.50

  on_budget_exceeded:
    - downgrade_veo_3_to_user_fallback
    - downgrade_ai_text_to_pillow
    - skip_non_critical_overlays
```

**Savings**: Prevents unexpected cost overruns

### 5. Background Removal Optimization

```python
def choose_bg_removal_method(monthly_volume: int) -> str:
    """
    Auto-select based on volume:
    - <20,000/month: Use Photoroom API ($0.02/image)
    - >20,000/month: Use rembg self-hosted ($0.00/image)
    """
    if monthly_volume > 20_000:
        return "rembg"
    else:
        return "photoroom_api"
```

**Break-even**: 20,000 images/month

---

## Production Deployment

### Environment Variables

```bash
# .env.production
# Google AI
GOOGLE_AI_API_KEY=your_google_api_key
GOOGLE_PROJECT_ID=your_project_id
ENABLE_VEO_3=true
ENABLE_GEMINI_FLASH=true

# Background Removal
PHOTOROOM_API_KEY=your_photoroom_key
BG_REMOVAL_METHOD=auto  # "photoroom_api", "rembg", or "auto"

# Cost Controls
MAX_COST_PER_VIDEO=2.50
MONTHLY_BUDGET_USD=1000.00
ENABLE_COST_TRACKING=true

# Caching
REDIS_URL=redis://localhost:6379
AI_CACHE_TTL_HOURS=168
CACHE_ENABLED=true

# FFmpeg
FFMPEG_NVENC_ENABLED=true
FFMPEG_THREADS=4

# API
API_RATE_LIMIT_RPM=60
API_ENABLE_WEBHOOKS=true
```

### Docker Compose (Production)

```yaml
version: '3.8'

services:
  api:
    build: .
    environment:
      - GOOGLE_AI_API_KEY=${GOOGLE_AI_API_KEY}
      - PHOTOROOM_API_KEY=${PHOTOROOM_API_KEY}
    depends_on:
      - redis
      - postgres
    ports:
      - "8000:8000"

  worker:
    build: .
    command: celery -A src.queue.celery_app worker --concurrency=4
    environment:
      - GOOGLE_AI_API_KEY=${GOOGLE_AI_API_KEY}
      - FFMPEG_NVENC_ENABLED=true
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              capabilities: [gpu]

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=video_creator
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  redis_data:
  postgres_data:
```

---

## Next Steps

### Week 1 Actions

1. **Set up Google Cloud**
   ```bash
   # Create project
   gcloud projects create your-project-id

   # Enable APIs
   gcloud services enable generativelanguage.googleapis.com
   gcloud services enable aiplatform.googleapis.com

   # Create API key
   gcloud auth application-default login
   ```

2. **Install new SDK**
   ```bash
   pip install google-genai  # Not google-generativeai!
   ```

3. **Test Veo 3 generation**
   ```python
   from google import genai
   client = genai.Client(api_key="YOUR_KEY")
   # Test generation...
   ```

4. **Set up Photoroom account**
   - Sign up at photoroom.com/api
   - Get API key
   - Test background removal

5. **Clone project structure**
   ```bash
   mkdir ai-video-creator
   cd ai-video-creator
   # Set up structure from above
   ```

### Month 1 Goals

- Complete Phase 1 (traditional pipeline)
- Complete Phase 2 (AI integration)
- Process first AI-generated video end-to-end
- Establish cost tracking baseline

### Long-Term Vision

- **Month 3**: Production launch with both AI and traditional capabilities
- **Month 6**: Scale to 1,000 videos/day with optimized caching
- **Month 12**: Add advanced features (AI voice-over, auto-subtitles, scene detection)

---

## Conclusion

This implementation plan provides a **flexible, cost-effective** approach to video content creation:

✅ **Use AI when it adds value** (artistic text, base video generation)
✅ **Use traditional tools when they're superior** (precision, speed, cost)
✅ **Give users choice** (AI or user-provided assets)
✅ **Optimize costs aggressively** (caching, free tiers, smart fallbacks)
✅ **Scale efficiently** (queue-based, hardware acceleration)

**Estimated Costs**:
- Development: 16 weeks, $20K-30K
- Infrastructure: $50-200/month (depending on volume)
- Per-video: $0.00 - $2.50 (user choice)

**Key Differentiators**:
- Hybrid AI + traditional (unique positioning)
- Cost transparency and controls
- Template-driven automation
- Production-grade quality

The tool is ready for development with all research completed and architecture defined!

---

**Research Completed**: January 14, 2025
**Total Research Documents**: 10
**Ready for**: Phase 1 Development
