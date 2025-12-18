# Claude Agent SDK Structured Outputs - Comprehensive Guide

**Date:** 2025-12-12
**Source:** Official Claude Platform Documentation

---

## 1. What Are Structured Outputs?

Structured outputs constrain Claude's responses to follow a specific JSON schema, guaranteeing valid, parseable output.

**Benefits:**
- Always receive valid JSON matching your schema
- Type-safe with guaranteed field types
- No retry logic needed
- Clean separation of output requirements

---

## 2. Defining Schemas with Pydantic

```python
from pydantic import BaseModel, Field
from typing import List, Literal

class SocialMediaContent(BaseModel):
    hook: str = Field(description="First 3 seconds grabber")
    body: str = Field(description="Main message")
    cta: str = Field(description="Call-to-action")
    hashtags: List[str] = Field(min_items=3, max_items=10)
    platform: Literal["instagram", "tiktok", "linkedin", "twitter"]
    image_prompt: str
```

---

## 3. Using with API

```python
import anthropic

client = anthropic.Anthropic()

response = client.beta.messages.parse(
    model="claude-opus-4-5-20251101",
    max_tokens=1024,
    betas=["structured-outputs-2025-11-13"],  # Required beta flag
    messages=[{
        "role": "user",
        "content": "Create a TikTok post about AI tools"
    }],
    output_format=SocialMediaContent,  # Pass Pydantic model
)

# Access validated output
content = response.parsed_output
print(f"Hook: {content.hook}")
print(f"Platform: {content.platform}")
```

---

## 4. Using with Agent SDK

```python
async for message in query(
    prompt="Create viral Instagram content",
    options={
        "output_format": {
            "type": "json_schema",
            "schema": SocialMediaContent.model_json_schema()
        }
    }
):
    if hasattr(message, 'structured_output'):
        content = SocialMediaContent.model_validate(message.structured_output)
```

---

## 5. Nested Schemas

```python
class ImageSpec(BaseModel):
    prompt: str
    style: str
    dimensions: str

class CTAButton(BaseModel):
    text: str
    link_type: str
    action: str

class SocialMediaContent(BaseModel):
    hook: str
    body: str
    cta: CTAButton          # Nested object
    image: ImageSpec        # Nested object
    hashtags: List[str]
```

---

## 6. Error Handling

```python
# Check for refusals
if message.subtype == 'error_max_structured_output_retries':
    print("Could not generate valid content format")

# Pydantic validation
try:
    result = SocialMediaContent.model_validate(response.parsed_output)
except ValidationError as e:
    print(f"Validation failed: {e}")
```

---

## Key Notes

- **Beta flag required**: `betas=["structured-outputs-2025-11-13"]`
- **First request latency**: Additional time for grammar compilation
- **Caching**: Grammars cached for 24 hours
- **Works with**: Claude Opus 4.5, Sonnet 4.5, Haiku 4.5
