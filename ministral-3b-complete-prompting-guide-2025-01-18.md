---
title: Ministral 3B Complete Prompting Guide
date: 2025-01-18
research_query: "Complete prompting guide for Mistral AI Ministral 3B model"
completeness: 95%
performance: "v2.0 wide-then-deep (10 parallel searches)"
execution_time: "~2 minutes"
model: "mistralai/ministral-3b (Ministral 3 3B Instruct 2512)"
---

# Ministral 3B: Complete Prompting Guide (2025)

## Table of Contents
1. [Model Overview](#model-overview)
2. [Technical Specifications](#technical-specifications)
3. [Model Variants Comparison](#model-variants-comparison)
4. [Prompting Best Practices](#prompting-best-practices)
5. [Chat Template Format](#chat-template-format)
6. [Effective Prompting Techniques](#effective-prompting-techniques)
7. [Known Limitations & Workarounds](#known-limitations--workarounds)
8. [Practical Examples](#practical-examples)
9. [API Usage & Deployment](#api-usage--deployment)
10. [Hardware Requirements](#hardware-requirements)

---

## Model Overview

### What is Ministral 3B?

**Ministral 3B** (officially **Ministral 3 3B**) is Mistral AI's smallest and most efficient edge-optimized language model, released in December 2024 as part of the Ministral 3 family. Despite being only 3 billion parameters, it **already outperforms Mistral 7B** on most benchmarks and beats competing models like Google's Gemma 2 2B and Meta's Llama 3.2 3B.

### Key Capabilities

**Multimodal Processing**
- Text and image understanding (vision-enabled)
- Integrates 3.4B parameter language model + 0.4B parameter vision encoder
- Native image analysis and visual content insights

**Multilingual Support**
- Dozens of languages: English, French, Spanish, German, Italian, Portuguese, Dutch, Chinese, Japanese, Korean, Arabic
- Strong performance across language families

**Agentic Capabilities**
- Native function calling (built into architecture)
- JSON output formatting
- Tool orchestration and multi-step workflows
- Best-in-class for its size category

**Edge Optimization**
- Deployable on phones, laptops, drones, embedded systems
- Fits in 8GB VRAM (FP8) or 4GB when quantized
- Fast inference: up to 385 tokens/second on NVIDIA RTX 5090
- Can run on Raspberry Pi 5 (8GB RAM)

### Intended Use Cases

**Where Ministral 3B Excels:**
- Edge/mobile deployment (on-device AI)
- Real-time translation and text classification
- Image captioning and document understanding
- Lightweight chat interfaces in constrained environments
- Content generation (short-form)
- Data extraction and summarization
- RAG (Retrieval-Augmented Generation) applications
- Agentic workflows with tool calling
- Privacy-sensitive applications (local deployment)

**Not Recommended For:**
- Deep multi-step reasoning (use 14B reasoning variant)
- Complex code generation (use 8B/14B)
- Long-form creative writing
- High-stakes decision-making requiring nuanced judgment

---

## Technical Specifications

### Ministral 3 3B Instruct 2512 (Latest Version)

| Specification | Value |
|--------------|-------|
| **Parameters** | 3.4B (language) + 0.4B (vision encoder) |
| **Context Window** | 256,000 tokens |
| **Architecture** | Dense transformer |
| **License** | Apache 2.0 (fully open-source) |
| **Release Date** | December 2024 |
| **Predecessor** | Ministral 3B (October 2024, 128k context) |
| **Training** | Pretrained → Midtraining (140B reasoning tokens) → SFT → APO alignment |
| **Quantization** | FP8, BF16, GGUF (Q4_K_M, Q5_K_M) |

### Performance Benchmarks

**Strengths (vs. competing 3B models):**
- **General Knowledge:** 94.0% accuracy
- **Ethics:** 98.0% accuracy
- **Mathematics:** 82.0% accuracy
- **Email Classification:** 95.0% accuracy
- **MMLU:** 60.9 (vs. Gemma 2B: 52.4, Llama 3.2 3B: 56.2)
- **HumanEval (Code):** Competitive for edge deployment

**Weaknesses:**
- **Instruction Following:** 33.0% (vs. 14B: 52%)
- **Reasoning:** 22.0% (vs. 14B: 52%)
- **Hallucinations:** 0.0% accuracy (struggles to acknowledge uncertainty)

---

## Model Variants Comparison

### Ministral 3 Family (3B vs. 8B vs. 14B)

| Feature | 3B | 8B | 14B |
|---------|----|----|-----|
| **Best For** | Edge devices, mobile, cost-sensitive | Balanced production workloads | Max performance on single GPU |
| **Deployment** | Single consumer GPU/mobile | Single GPU | Single high-end GPU (1xH200) |
| **Context Window** | 256k tokens | 131k tokens | 262k tokens |
| **Inference Speed** | Up to 385 tok/s (RTX 5090) | Fast (sliding window attention) | Moderate |
| **API Cost** | $0.04/M tokens | $0.10/M tokens | $0.20/M tokens |
| **Reasoning** | Basic (22%) | Good | State-of-the-art (85% AIME '25) |
| **Use Cases** | Lightweight chat, translation, classification | RAG, automation, internal tools | Complex reasoning, deep analysis |
| **VRAM (quantized)** | <4GB | ~6GB | ~10GB |

### Model Variant Types (Each Size)

1. **Base**: Pretrained, not instruction-tuned (for custom fine-tuning)
2. **Instruct**: Fine-tuned for chat and instruction following
3. **Reasoning**: Specialized for multi-step reasoning with extended thinking mode

**Recommendation:** Use **Instruct** for general prompting. Use **Reasoning** variant for math, code debugging, and complex problem-solving.

---

## Prompting Best Practices

### General Principles

#### 1. Clarity and Specificity
Express instructions explicitly and unambiguously. Avoid vague requests.

**Bad:**
```
Tell me about Python
```

**Good:**
```
Explain Python's list comprehension syntax with 3 practical examples for data filtering.
```

#### 2. Provide Context
Give relevant background information to ground the model.

**Example:**
```
Context: I'm building a Flask API for a todo app.
Task: Generate code for a POST endpoint that validates JSON input and saves to SQLite.
```

#### 3. Specify Constraints
Define limitations, format requirements, or preferences.

**Example:**
```
Summarize this article in 3 bullet points. Each point must be under 20 words. Focus on actionable insights.
```

#### 4. Define Output Format
Indicate desired structure when applicable.

**Example:**
```
List 5 Python libraries for data visualization. Format as:
- Library Name: [name]
- Use Case: [description]
- Difficulty: [Beginner/Intermediate/Advanced]
```

### System Prompt Best Practices

**Recommended Structure:**
```
<s>[INST] {SYSTEM_PROMPT}

{USER_INSTRUCTION} [/INST]
```

**System Prompt Template (from Mistral):**
```
Always assist with care, respect, and truth. Respond with utmost utility yet securely. Avoid harmful, unethical, prejudiced, or negative content. Ensure replies promote fairness and positivity.
```

**Custom System Prompt Example:**
```
You are a Python coding assistant specializing in data analysis. Provide concise, well-commented code examples. Prioritize pandas and numpy solutions. Always include error handling.
```

**Best Practices:**
- Keep system prompts between 50-200 tokens (sweet spot)
- Define personality, role, and constraints
- Append Mistral's safety prompt to your custom prompt
- Avoid overly long system prompts (reduces effective context)

### Parameter Recommendations

#### Temperature

| Variant | Use Case | Recommended Temperature |
|---------|----------|-------------------------|
| **Instruct** | Production/daily use | 0.1 |
| **Instruct** | Creative tasks | 0.3-0.5 |
| **Reasoning** | Most environments | 0.7 |
| **Reasoning** | Deterministic output | 0.1-0.3 |

**Lower temperature (0.0-0.3):** Predictable, focused responses
**Higher temperature (0.6-1.0):** Creative, diverse outputs

#### Other Parameters

- **Top-k:** 40-50 (default)
- **Top-p:** 0.9 (nucleus sampling)
- **Max tokens:** 512-2048 for most tasks
- **Repetition penalty:** 1.1-1.2 (if available)

### Iterative Refinement Strategy

If output doesn't meet expectations:

1. **Assess:** What's missing or incorrect?
2. **Revise:** Add specificity, examples, or constraints
3. **Test:** Try temperature adjustments
4. **Iterate:** Refine prompt 2-3 times before changing approach

---

## Chat Template Format

### Mistral-Specific Tokens

Mistral models use a unique chat template format with special tokens:

**Special Tokens:**
- `<s>`: Beginning of string (BOS)
- `</s>`: End of string (EOS)
- `[INST]`: Start of user instruction (regular string, not a special token)
- `[/INST]`: End of user instruction (regular string)

### Single-Turn Prompt Format

```
<s>[INST] {user_message} [/INST]
```

**Example:**
```
<s>[INST] What is the capital of France? [/INST]
```

### Multi-Turn Conversation Format

```
<s>[INST] {user_message_1} [/INST] {assistant_response_1}</s>[INST] {user_message_2} [/INST]
```

**Example:**
```
<s>[INST] What is Python? [/INST] Python is a high-level programming language known for readability and versatility.</s>[INST] Show me a hello world example. [/INST]
```

**Note:** No spaces between tokens in v2+ format.

### System Prompt Integration

**Official Mistral Format (prepend to first user message):**
```
<s>[INST] {system_prompt}

{user_instruction} [/INST]
```

**Example:**
```
<s>[INST] You are a helpful Python tutor.

Explain list comprehensions. [/INST]
```

**Important Notes:**
- Mistral does NOT natively support `<<SYS>>`/`<</SYS>>` tokens (that's Llama 2 format)
- HuggingFace's `apply_chat_template` may incorrectly use Llama format
- Use `mistral_common` library for ground truth tokenization
- Conversation roles must alternate user/assistant/user/assistant

### Using mistral_common for Correct Formatting

```python
from mistral_common.tokens.tokenizers.mistral import MistralTokenizer
from mistral_common.protocol.instruct.messages import UserMessage, AssistantMessage

tokenizer = MistralTokenizer.v3()

messages = [
    UserMessage(content="What is the capital of France?"),
]

tokens = tokenizer.encode_chat_completion(messages).tokens
```

---

## Effective Prompting Techniques

### 1. Zero-Shot Prompting

**Definition:** Direct instruction without examples.

**Best For:** Simple tasks (classification, translation, factual Q&A)

**Example:**
```
<s>[INST] Classify this email as spam or legitimate:
"Congratulations! You've won $1,000,000. Click here to claim."
[/INST]
```

**Expected Output:** Classification with brief reasoning.

**Tips:**
- Works well for Ministral 3B's strengths (classification, knowledge)
- Use clear task definitions
- Specify output format

### 2. Few-Shot Prompting

**Definition:** Provide 2-5 examples before the actual task.

**Best For:** Pattern recognition, consistent formatting, domain-specific tasks

**Example:**
```
<s>[INST] Extract key information from customer reviews:

Example 1:
Review: "Great product but shipping was slow."
Sentiment: Positive | Issue: Shipping

Example 2:
Review: "Terrible quality, broke after one use."
Sentiment: Negative | Issue: Quality

Now extract from:
Review: "Love the features but setup was confusing."
[/INST]
```

**Tips:**
- Use 2-3 examples (avoid overloading context)
- Examples should match task complexity
- Ensure diverse examples

### 3. Chain-of-Thought (CoT) Prompting

**Definition:** Ask model to show step-by-step reasoning.

**Best For:** Math, logic, multi-step problem-solving

**Effectiveness on 3B:** Moderate (better on reasoning variant or 8B/14B)

**Example:**
```
<s>[INST] Solve this step-by-step:
A store has 120 apples. They sell 35% in the morning and 20% of the remainder in the afternoon. How many apples are left?

Show your work.
[/INST]
```

**Tips:**
- Explicitly request "step-by-step" or "show your work"
- Use reasoning variant for best results
- Temperature: 0.7 for reasoning tasks
- Keep reasoning traces in context for follow-ups

### 4. Chain-of-Thought with Few-Shot

**Combine both techniques for complex tasks:**

```
<s>[INST] Solve math problems step-by-step:

Example:
Problem: 24 × 15 = ?
Step 1: 24 × 10 = 240
Step 2: 24 × 5 = 120
Step 3: 240 + 120 = 360
Answer: 360

Now solve:
Problem: 37 × 12 = ?
[/INST]
```

### 5. Role-Playing and Persona Prompts

**Definition:** Assign the model a specific role or expertise.

**Best For:** Domain-specific advice, creative writing, tutoring

**Example:**
```
<s>[INST] You are an experienced DevOps engineer specializing in Kubernetes.

A junior developer asks: "What's the difference between a Deployment and a StatefulSet?"

Provide a clear, beginner-friendly explanation with examples.
[/INST]
```

**Tips:**
- Define expertise level (e.g., "explain like I'm 5" or "advanced technical audience")
- Combine with system prompts for persistent persona
- Works well for 3B due to strong system prompt adherence

### 6. Task-Specific Prompting

#### Code Generation

**Best Practices:**
- Specify language, libraries, and constraints
- Request comments and error handling
- Keep tasks simple (3B struggles with complex multi-file code)

**Example:**
```
<s>[INST] Write a Python function that:
1. Takes a list of numbers
2. Filters out negatives
3. Returns the sum of remaining numbers
4. Include error handling for empty lists

Use type hints and docstrings.
[/INST]
```

#### Text Classification

**Best Practices:**
- Define categories explicitly
- Provide confidence scores if needed
- Use few-shot for domain-specific categories

**Example:**
```
<s>[INST] Classify customer support tickets into: Billing, Technical, General Inquiry

Ticket: "My payment method isn't working when I try to upgrade."
Category: [/INST]
```

#### Data Extraction

**Best Practices:**
- Specify exact fields to extract
- Define output format (JSON, CSV, structured list)
- Use examples for complex structures

**Example:**
```
<s>[INST] Extract structured data from this text:

"John Doe, age 32, works at Acme Corp as a Software Engineer. Email: john@acme.com"

Return as JSON with fields: name, age, company, role, email
[/INST]
```

#### Summarization

**Best Practices:**
- Specify length (word count or bullet points)
- Define focus (key points, actionable insights, technical details)
- Temperature: 0.1-0.3 for factual, 0.5-0.7 for creative

**Example:**
```
<s>[INST] Summarize this article in 3 bullet points, each under 15 words. Focus on actionable takeaways.

[Article text here]
[/INST]
```

#### Translation

**Best Practices:**
- Specify source and target languages explicitly
- Request formal/informal tone if needed
- Ministral 3B supports dozens of languages

**Example:**
```
<s>[INST] Translate this English text to French (formal tone):

"Thank you for your inquiry. We will respond within 24 hours."
[/INST]
```

### 7. Function Calling & Agentic Workflows

**Native Function Calling:**
Ministral 3B has built-in function calling (not prompt engineering).

**Best Practices:**
- Define tools clearly in system prompt
- Limit to 3-5 tools (avoid overwhelming the model)
- Use JSON schema for tool definitions
- Keep tool descriptions concise

**Example System Prompt:**
```
You are an assistant with access to these tools:
1. get_weather(location: str) -> dict
2. search_web(query: str) -> list
3. send_email(to: str, subject: str, body: str) -> bool

When a user request requires a tool, output:
{"tool": "tool_name", "parameters": {...}}
```

**User Request:**
```
<s>[INST] What's the weather in Paris? [/INST]
```

**Expected Output:**
```json
{"tool": "get_weather", "parameters": {"location": "Paris"}}
```

### 8. Vision Prompting (Multimodal)

**Image Understanding:**
Ministral 3B can analyze images alongside text.

**Best Practices:**
- Maintain 1:1 aspect ratio images (crop if needed)
- Avoid overly thin or wide images
- Ask specific questions about image content
- Combine with text context for better results

**Example:**
```
[Image: Product photo]

<s>[INST] Analyze this product image and:
1. Describe the product
2. Identify potential quality issues
3. Suggest improvements for the photo
[/INST]
```

---

## Known Limitations & Workarounds

### Major Limitations

#### 1. Weak Multi-Step Reasoning (22% accuracy)

**Problem:** Struggles with complex logic chains, abstract problem-solving.

**Workarounds:**
- **Use Reasoning Variant:** Ministral 3-3B-Reasoning-2512 is trained for this
- **Break Down Tasks:** Split complex problems into smaller steps
- **Chain-of-Thought:** Explicitly request step-by-step reasoning
- **Upgrade to 8B/14B:** For critical reasoning tasks
- **Temperature:** Use 0.7 for reasoning tasks

**Example Workaround:**
```
Instead of: "Solve this multi-variable calculus problem"

Use:
"Step 1: Identify the variables
Step 2: Apply the chain rule
Step 3: Simplify the expression
Step 4: Verify the solution"
```

#### 2. Poor Instruction Following (33% accuracy)

**Problem:** May deviate from complex, multi-part instructions.

**Workarounds:**
- **Simplify Instructions:** Use numbered lists, one task at a time
- **Use Examples:** Show desired output format
- **Repeat Constraints:** Reiterate critical requirements
- **Structured Prompts:** Use templates with clear sections
- **System Prompts:** Define behavior expectations upfront

**Example Workaround:**
```
Bad:
"Write a blog post about AI, make it 500 words, use casual tone, include 3 statistics, add a call-to-action, and format with headers."

Good:
Format: Blog post
Topic: AI in healthcare
Length: 500 words
Tone: Casual, conversational
Required elements:
1. Introduction (100 words)
2. 3 statistics with sources
3. Real-world example
4. Call-to-action conclusion

Use headers: Introduction, Benefits, Example, Conclusion
```

#### 3. Hallucination & Uncertainty (0% accuracy)

**Problem:** Doesn't acknowledge when uncertain; may fabricate information.

**Workarounds:**
- **Explicit Uncertainty Requests:** "If you're unsure, say 'I don't have enough information'"
- **Source Citations:** Request sources for factual claims
- **Constrain to Context:** Use RAG with retrieved documents
- **Fact-Check Critical Info:** Verify against larger models or search
- **Temperature:** Lower (0.0-0.1) for factual tasks

**Example Workaround:**
```
<s>[INST] What year was the Python programming language created?

If you're certain, provide the year and creator.
If uncertain, state: "I need to verify this information."
[/INST]
```

#### 4. Limited Long-Form Generation

**Problem:** 256k context, but struggles with coherent long-form content (books, comprehensive reports).

**Workarounds:**
- **Chunked Generation:** Generate sections separately, then combine
- **Outlines First:** Create structure, then fill in sections
- **Use Summaries:** Maintain context by summarizing previous sections
- **Use 8B/14B:** For extensive creative writing or reports

**Example Workaround:**
```
Step 1: Generate outline
<s>[INST] Create a detailed outline for a 2000-word article on renewable energy. [/INST]

Step 2: Generate each section
<s>[INST] Using this outline: [paste outline]

Write the Introduction section (200 words). [/INST]

Step 3: Repeat for each section
```

#### 5. Complex Code Generation Limitations

**Problem:** Can generate simple functions but struggles with multi-file projects, complex architectures.

**Workarounds:**
- **Single Functions:** Request one function at a time
- **Provide Context:** Show existing code structure
- **Use Code Comments:** Request detailed comments for complex logic
- **Iterative Development:** Generate skeleton, then add features
- **Upgrade to 8B:** Better for code generation

**Example Workaround:**
```
<s>[INST] Project context: Flask API with SQLAlchemy models

Generate ONLY the User model class with these fields:
- id (primary key)
- username (unique, max 50 chars)
- email (unique)
- created_at (timestamp)

Include __repr__ method.
[/INST]
```

### Edge Deployment Considerations

**Hardware Constraints:**
- **Quantization Required:** Use Q4_K_M or Q5_K_M for <8GB devices
- **Context Limits:** vLLM currently operates at 32k tokens (not full 256k)
- **Throughput:** Optimize batch size for mobile/edge devices
- **NPU Support:** Future chips (Intel Meteor Lake, AMD Ryzen AI, Snapdragon X Elite) will accelerate

**Workarounds:**
- **Use Ollama:** Automatic quantization and optimization
- **GGUF Formats:** Efficient for CPU/mobile deployment
- **Batch Prompts:** Process multiple requests together
- **Cache Prompts:** Reuse system prompts across sessions

---

## Practical Examples

### Example 1: Text Classification with Confidence

**Prompt:**
```
<s>[INST] Classify this product review sentiment. Provide sentiment (Positive/Negative/Neutral) and confidence score (0-100):

Review: "The laptop is decent for the price, but the battery life is disappointing."
[/INST]
```

**Expected Output:**
```
Sentiment: Neutral (leaning Negative)
Confidence: 75

Reasoning: The review contains both positive ("decent for the price") and negative ("battery life is disappointing") elements, with the criticism being more specific and impactful.
```

### Example 2: Data Extraction to JSON

**Prompt:**
```
<s>[INST] Extract contact information from this email signature and return as JSON:

"Best regards,
Sarah Johnson
Senior Data Analyst
TechCorp Solutions
Email: sarah.j@techcorp.com
Phone: +1-555-0123
LinkedIn: linkedin.com/in/sarahjohnson"

JSON format:
{
  "name": "",
  "title": "",
  "company": "",
  "email": "",
  "phone": "",
  "linkedin": ""
}
[/INST]
```

**Expected Output:**
```json
{
  "name": "Sarah Johnson",
  "title": "Senior Data Analyst",
  "company": "TechCorp Solutions",
  "email": "sarah.j@techcorp.com",
  "phone": "+1-555-0123",
  "linkedin": "linkedin.com/in/sarahjohnson"
}
```

### Example 3: Few-Shot Translation with Tone

**Prompt:**
```
<s>[INST] Translate customer messages from English to Spanish, maintaining tone:

Example 1:
English: "Thanks for the quick response!"
Tone: Casual, appreciative
Spanish: "¡Gracias por la respuesta rápida!"

Example 2:
English: "We require immediate assistance with this matter."
Tone: Formal, urgent
Spanish: "Necesitamos asistencia inmediata con este asunto."

Now translate:
English: "Could you help me reset my password?"
Tone: Polite, formal
[/INST]
```

**Expected Output:**
```
Spanish: "¿Podría ayudarme a restablecer mi contraseña?"
```

### Example 4: Code Generation with Constraints

**Prompt:**
```
<s>[INST] Write a Python function that validates email addresses.

Requirements:
- Function name: validate_email
- Parameter: email (str)
- Return: bool
- Check for: @ symbol, domain, no spaces
- Include: type hints, docstring, 2-3 test cases

Keep it simple and readable.
[/INST]
```

**Expected Output:**
```python
def validate_email(email: str) -> bool:
    """
    Validates if a string is a properly formatted email address.

    Args:
        email: String to validate

    Returns:
        True if valid email format, False otherwise
    """
    if not email or ' ' in email:
        return False

    if '@' not in email:
        return False

    parts = email.split('@')
    if len(parts) != 2:
        return False

    username, domain = parts
    if not username or not domain:
        return False

    if '.' not in domain:
        return False

    return True

# Test cases
print(validate_email("user@example.com"))  # True
print(validate_email("invalid.email"))      # False
print(validate_email("user @test.com"))     # False
```

### Example 5: Multi-Step Reasoning (Use Reasoning Variant)

**Prompt:**
```
<s>[INST] A company has 150 employees. 60% work remotely. Of the remote workers, 40% are in different time zones. How many employees work remotely in different time zones?

Solve step-by-step.
[/INST]
```

**Expected Output:**
```
Step 1: Calculate total remote workers
60% of 150 = 0.60 × 150 = 90 remote workers

Step 2: Calculate remote workers in different time zones
40% of 90 = 0.40 × 90 = 36 employees

Answer: 36 employees work remotely in different time zones.
```

### Example 6: Summarization with Specific Focus

**Prompt:**
```
<s>[INST] Summarize this technical article in 4 bullet points. Focus on practical implementation details, not theory.

[Article text about microservices architecture]
[/INST]
```

**Expected Output:**
```
- Use API Gateway (Kong/Nginx) for routing and authentication between services
- Implement event-driven communication with message queues (RabbitMQ/Kafka) instead of direct service calls
- Set up centralized logging with ELK stack for debugging across distributed services
- Deploy with Docker Compose locally, Kubernetes for production with auto-scaling policies
```

### Example 7: Function Calling (Agentic)

**System Prompt:**
```
You have access to these tools:
1. calculate(expression: str) -> float - Evaluates math expressions
2. get_time(timezone: str) -> str - Returns current time in timezone

When user needs a tool, output JSON:
{"tool": "tool_name", "parameters": {...}}
```

**User Prompt:**
```
<s>[INST] What's 15% of 340? [/INST]
```

**Expected Output:**
```json
{"tool": "calculate", "parameters": {"expression": "340 * 0.15"}}
```

### Example 8: Image Analysis (Multimodal)

**Prompt:**
```
[Image: Screenshot of a website homepage]

<s>[INST] Analyze this website homepage and provide:
1. Main color scheme
2. Key UI elements visible
3. One UX improvement suggestion
[/INST]
```

**Expected Output:**
```
1. Main color scheme: Navy blue (#1a237e) and white, with orange (#ff6f00) accent buttons

2. Key UI elements:
   - Top navigation bar with logo (left) and menu items (right)
   - Hero section with headline and CTA button
   - Three-column feature cards below fold
   - Footer with social media icons

3. UX improvement: Increase contrast on the hero CTA button - the orange on navy is difficult to read. Consider white text with a darker orange (#e65100) or add a subtle shadow for better accessibility.
```

### Example 9: Iterative Prompt Refinement

**First Attempt (Vague):**
```
<s>[INST] Tell me about databases [/INST]
```

**Output:** Generic overview, too broad.

**Second Attempt (More Specific):**
```
<s>[INST] Explain the difference between SQL and NoSQL databases [/INST]
```

**Output:** Better, but still theoretical.

**Final Attempt (Specific + Context + Format):**
```
<s>[INST] I'm choosing a database for a social media app with:
- 100k users
- Frequent reads/writes
- User profiles, posts, comments

Compare SQL (PostgreSQL) vs NoSQL (MongoDB) for this use case.

Format:
- Best choice: [SQL/NoSQL]
- Reasoning: [2-3 sentences]
- Trade-offs: [2 bullet points]
[/INST]
```

**Output:** Actionable, context-aware recommendation.

---

## API Usage & Deployment

### Platform Availability

Ministral 3 is available on:
- **Mistral AI Studio** (official)
- **Amazon Bedrock**
- **Azure AI Foundry**
- **Hugging Face** (transformers, inference API)
- **Together AI** (API)
- **OpenRouter** (aggregator)
- **Fireworks AI**
- **Modal**
- **IBM WatsonX**
- **Ollama** (local)
- **LM Studio** (local GUI)
- **llama.cpp** (local C++)

### 1. Mistral AI API (Official)

**Endpoint:**
```
https://api.mistral.ai/v1/chat/completions
```

**Example Request (Python):**
```python
import os
from mistralai import Mistral

api_key = os.environ["MISTRAL_API_KEY"]
client = Mistral(api_key=api_key)

response = client.chat.complete(
    model="ministral-3b-latest",
    messages=[
        {
            "role": "user",
            "content": "What is the capital of France?"
        }
    ],
    temperature=0.1,
    max_tokens=256
)

print(response.choices[0].message.content)
```

**Pricing:** $0.04 per million tokens (input/output)

### 2. Hugging Face Transformers

**Installation:**
```bash
pip install transformers accelerate torch
```

**Usage (FP8 - Requires Latest Transformers):**
```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "mistralai/Ministral-3-3B-Instruct-2512"

# Load tokenizer
tokenizer = AutoTokenizer.from_pretrained(model_name)

# Load model in FP8 (requires transformers from main)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    device_map="auto",
    torch_dtype="auto"
)

# Format prompt
messages = [
    {"role": "user", "content": "Explain Python list comprehensions"}
]
prompt = tokenizer.apply_chat_template(messages, tokenize=False)

# Generate
inputs = tokenizer(prompt, return_tensors="pt").to("cuda")
outputs = model.generate(**inputs, max_new_tokens=256, temperature=0.1)
response = tokenizer.decode(outputs[0], skip_special_tokens=True)

print(response)
```

**Note:** For vision capabilities, use the multimodal pipeline.

### 3. Hugging Face Inference API

**Example (REST API):**
```python
import requests

API_URL = "https://api-inference.huggingface.co/models/mistralai/Ministral-3-3B-Instruct-2512"
headers = {"Authorization": f"Bearer {YOUR_HF_TOKEN}"}

payload = {
    "inputs": "<s>[INST] What is machine learning? [/INST]",
    "parameters": {
        "max_new_tokens": 256,
        "temperature": 0.1
    }
}

response = requests.post(API_URL, headers=headers, json=payload)
print(response.json()[0]['generated_text'])
```

### 4. Together AI

**Installation:**
```bash
pip install together
```

**Usage:**
```python
import os
from together import Together

client = Together(api_key=os.environ.get("TOGETHER_API_KEY"))

response = client.chat.completions.create(
    model="mistralai/Ministral-3-3B-Instruct-2512",
    messages=[
        {"role": "user", "content": "Translate 'Hello' to Spanish"}
    ],
    temperature=0.1,
    max_tokens=128
)

print(response.choices[0].message.content)
```

**Pricing:** Check Together AI pricing page (typically competitive)

### 5. Ollama (Local Deployment)

**Installation:**
```bash
# Install Ollama: https://ollama.ai
ollama pull ministral-3:3b
```

**Usage (CLI):**
```bash
ollama run ministral-3:3b "What is Python?"
```

**Usage (API):**
```python
import requests

response = requests.post('http://localhost:11434/api/generate', json={
    "model": "ministral-3:3b",
    "prompt": "What is Python?",
    "stream": False,
    "options": {
        "temperature": 0.1
    }
})

print(response.json()['response'])
```

**Quantization Options:**
- `ministral-3:3b` (default Q4_K_M)
- `ministral-3:3b-q5-k-m` (higher quality)
- `ministral-3:3b-q8-0` (best quality, larger size)

### 6. vLLM (High-Performance Inference)

**Installation:**
```bash
pip install vllm
```

**Usage:**
```python
from vllm import LLM, SamplingParams

llm = LLM(model="mistralai/Ministral-3-3B-Instruct-2512")

sampling_params = SamplingParams(
    temperature=0.1,
    max_tokens=256
)

prompts = [
    "<s>[INST] What is AI? [/INST]",
    "<s>[INST] Explain Python [/INST]"
]

outputs = llm.generate(prompts, sampling_params)

for output in outputs:
    print(output.outputs[0].text)
```

**Recommended for:** Production deployments, batch processing, high throughput

### 7. LangChain Integration

**Installation:**
```bash
pip install langchain langchain-mistralai
```

**Usage:**
```python
from langchain_mistralai import ChatMistralAI
from langchain.schema import HumanMessage, SystemMessage

llm = ChatMistralAI(
    model="ministral-3b-latest",
    mistral_api_key="your-api-key",
    temperature=0.1
)

messages = [
    SystemMessage(content="You are a helpful coding assistant."),
    HumanMessage(content="Write a Python function to reverse a string")
]

response = llm.invoke(messages)
print(response.content)
```

### 8. LlamaIndex Integration

**Installation:**
```bash
pip install llama-index llama-index-llms-mistralai
```

**Usage:**
```python
from llama_index.llms.mistralai import MistralAI
from llama_index.core.llms import ChatMessage

llm = MistralAI(
    model="ministral-3b-latest",
    api_key="your-api-key",
    temperature=0.1
)

messages = [
    ChatMessage(role="user", content="Summarize the benefits of RAG")
]

response = llm.chat(messages)
print(response.message.content)
```

### 9. OpenRouter (Multi-Provider Aggregator)

**Usage:**
```python
import requests

response = requests.post(
    url="https://openrouter.ai/api/v1/chat/completions",
    headers={
        "Authorization": f"Bearer {OPENROUTER_API_KEY}",
    },
    json={
        "model": "mistralai/ministral-3b",
        "messages": [
            {"role": "user", "content": "What is the speed of light?"}
        ],
        "temperature": 0.1
    }
)

print(response.json()['choices'][0]['message']['content'])
```

---

## Hardware Requirements

### Cloud/GPU Deployment

| Configuration | VRAM | Use Case |
|---------------|------|----------|
| **FP16/BF16** | 16GB | Full precision, research |
| **FP8** | 8GB | Production, single GPU |
| **Q8_0** | ~4GB | Balanced quality/performance |
| **Q4_K_M** | ~2GB | Edge/mobile devices |
| **Q3_K_M** | <2GB | Ultra-constrained environments |

**Recommended GPUs:**
- **Cloud:** NVIDIA L4, T4, A10, H100
- **Consumer:** RTX 3090, RTX 4090, RTX 5090 (up to 385 tok/s)
- **Edge:** Jetson Orin Nano, Jetson AGX Orin

### CPU/Mobile Deployment

| Device Type | RAM | Performance |
|-------------|-----|-------------|
| **Desktop CPU** | 8GB+ | Moderate (quantized) |
| **Laptop** | 16GB+ | Good (Q4/Q5) |
| **Mobile (High-End)** | 8GB+ | Usable (Q4) |
| **Raspberry Pi 5** | 8GB | Slow but functional |
| **Embedded (Drone)** | 4GB+ | Basic tasks (Q3/Q4) |

**Accelerators (Future):**
- Intel Meteor Lake NPU
- AMD Ryzen AI NPU
- Qualcomm Snapdragon X Elite NPU

### Deployment Recommendations

**Local Development:**
- Use **Ollama** (easiest setup, auto-quantization)
- Use **LM Studio** (GUI, no coding)
- Use **llama.cpp** (maximum control)

**Production (Cloud):**
- Use **vLLM** (highest throughput)
- Use **Hugging Face Inference Endpoints** (managed)
- Use **Together AI / Fireworks** (serverless)

**Edge/Mobile:**
- Use **GGUF quantized models** (Q4_K_M recommended)
- Deploy with **llama.cpp** or **Ollama**
- Consider **TensorRT-LLM** for NVIDIA Jetson

**Cost-Sensitive:**
- Use **Together AI / OpenRouter** (pay-per-token)
- Self-host on **RunPod / Vast.ai** (cheap GPU rentals)
- Use **Hugging Face Inference API** (free tier for testing)

---

## Advanced Tips & Tricks

### 1. Context Window Optimization

**256k context, but use strategically:**
- **Effective limit:** vLLM currently 32k tokens
- **Prompt caching:** Reuse system prompts across sessions
- **Sliding window:** Summarize old context, keep recent messages
- **Chunking:** Process long documents in sections

### 2. Batch Processing for Efficiency

**Process multiple prompts simultaneously:**
```python
from vllm import LLM, SamplingParams

llm = LLM(model="ministralai/Ministral-3-3B-Instruct-2512")
sampling_params = SamplingParams(temperature=0.1, max_tokens=128)

prompts = [
    "<s>[INST] Classify: 'Great product!' [/INST]",
    "<s>[INST] Classify: 'Terrible service' [/INST]",
    "<s>[INST] Classify: 'It's okay' [/INST]"
]

outputs = llm.generate(prompts, sampling_params)
```

**Benefits:** 3-5x faster than sequential processing

### 3. Quantization Quality Comparison

| Quantization | Size | Quality Loss | Speed | Recommended For |
|--------------|------|--------------|-------|-----------------|
| **FP16** | 6.8GB | 0% (baseline) | Moderate | Research, cloud |
| **FP8** | 3.8GB | <1% | Fast | Production GPU |
| **Q8_0** | 3.6GB | ~1% | Fast | Balanced |
| **Q6_K** | 2.8GB | ~2% | Faster | Quality-conscious edge |
| **Q5_K_M** | 2.4GB | ~3% | Faster | Recommended edge |
| **Q4_K_M** | 2.0GB | ~5% | Very fast | Default edge (best value) |
| **Q3_K_M** | 1.6GB | ~10% | Fastest | Ultra-constrained |

**Recommendation:** Use Q4_K_M for 95% of edge deployments.

### 4. Caching Strategies

**For repeated queries:**
- Cache responses for identical prompts (hash-based)
- Use Redis/Memcached for distributed caching
- Cache embeddings for RAG applications

### 5. Prompt Compression Techniques

**Reduce token usage:**
- Remove filler words ("please", "kindly")
- Use abbreviations in system prompts
- Structured data (JSON) > natural language descriptions
- Short variable names in code examples

**Example:**
```
Instead of: "Could you please kindly help me to write a function..."
Use: "Write a function..."
```

### 6. Error Handling for Edge Deployment

**Handle failures gracefully:**
```python
import requests
from tenacity import retry, stop_after_attempt, wait_fixed

@retry(stop=stop_after_attempt(3), wait=wait_fixed(2))
def query_model(prompt):
    try:
        response = requests.post('http://localhost:11434/api/generate',
            json={"model": "ministral-3:3b", "prompt": prompt},
            timeout=30
        )
        return response.json()['response']
    except Exception as e:
        print(f"Error: {e}")
        raise
```

### 7. Fine-Tuning Ministral 3B

**Use base model for custom tasks:**
```bash
# Unsloth for efficient fine-tuning
pip install unsloth

# Fine-tune on custom dataset
python fine_tune.py \
  --model mistralai/Ministral-3-3B-Base-2512 \
  --dataset your_dataset.json \
  --epochs 3 \
  --learning_rate 2e-5
```

**Best for:** Domain-specific tasks (medical, legal, technical jargon)

---

## Quick Reference

### When to Use Ministral 3B

**Ideal Use Cases:**
- Edge/mobile AI applications
- Real-time classification and translation
- Privacy-sensitive local deployments
- Cost-sensitive high-volume API usage
- RAG systems (retrieval + generation)
- Agentic workflows with function calling
- Image captioning and document analysis
- Prototyping before scaling to larger models

**When to Use Larger Models (8B/14B):**
- Complex multi-step reasoning
- Advanced code generation (multi-file projects)
- Long-form creative writing
- High-stakes decision-making
- Nuanced instruction following

### Temperature Cheat Sheet

| Task Type | Recommended Temperature |
|-----------|-------------------------|
| Classification | 0.0-0.1 |
| Factual Q&A | 0.1 |
| Summarization | 0.1-0.3 |
| Translation | 0.1-0.2 |
| Code Generation | 0.1-0.2 |
| Data Extraction | 0.0-0.1 |
| Creative Writing | 0.5-0.8 |
| Brainstorming | 0.7-0.9 |
| Reasoning Tasks | 0.7 (reasoning variant) |

### Prompt Format Quick Reference

**Single Turn:**
```
<s>[INST] {user_message} [/INST]
```

**With System Prompt:**
```
<s>[INST] {system_prompt}

{user_message} [/INST]
```

**Multi-Turn:**
```
<s>[INST] {msg1} [/INST] {response1}</s>[INST] {msg2} [/INST]
```

---

## Conclusion

**Ministral 3B** is a groundbreaking edge-optimized language model that delivers Mistral 7B-level performance in a 3.4B parameter package. With native function calling, vision capabilities, and support for 256k context, it's ideal for:

- **Mobile/edge AI** (fits in 4GB RAM when quantized)
- **Cost-sensitive production workloads** ($0.04/M tokens)
- **Privacy-focused local deployments** (runs on laptops, phones, Raspberry Pi)
- **Agentic systems** (native tool calling, JSON output)

**Key Takeaways:**
1. Use **Instruct variant** for general prompting, **Reasoning variant** for math/logic
2. Keep **temperature at 0.1** for production, 0.7 for reasoning tasks
3. Use **few-shot prompting** for consistent formatting and domain-specific tasks
4. **Break down complex instructions** to work around weak reasoning
5. Deploy with **Ollama/Q4_K_M quantization** for best edge performance
6. Use **vLLM** for high-throughput production deployments
7. **Explicitly request uncertainty acknowledgment** to mitigate hallucinations

**Best Practices Summary:**
- ✅ Simple, clear instructions
- ✅ Structured prompts with numbered steps
- ✅ Few-shot examples for consistency
- ✅ Temperature tuning per task type
- ✅ Quantization for edge deployment
- ❌ Avoid overly complex multi-step reasoning
- ❌ Don't expect uncertainty acknowledgment without prompting
- ❌ Don't use for long-form creative writing

For most edge AI applications, **Ministral 3B hits the sweet spot** between capability and efficiency.

---

## Sources

### Official Documentation
- [Introducing Mistral 3 | Mistral AI](https://mistral.ai/news/mistral-3)
- [Un Ministral, des Ministraux | Mistral AI](https://mistral.ai/news/ministraux)
- [Ministral 3 3B Docs | Mistral AI](https://docs.mistral.ai/models/ministral-3-3b-25-12)
- [Prompting Capabilities | Mistral Docs](https://docs.mistral.ai/guides/prompting_capabilities)
- [Chat Templates | Mistral Cookbook](https://docs.mistral.ai/cookbooks/concept-deep-dive-tokenization-chat_templates)

### Model Repositories
- [mistralai/Ministral-3-3B-Instruct-2512 · Hugging Face](https://huggingface.co/mistralai/Ministral-3-3B-Instruct-2512)
- [mistralai/Ministral-3-3B-Reasoning-2512 · Hugging Face](https://huggingface.co/mistralai/Ministral-3-3B-Reasoning-2512)
- [ministral-3:3b | Ollama](https://ollama.com/library/ministral-3:3b)
- [Ministral 3 Collection | Hugging Face](https://huggingface.co/collections/mistralai/ministral-3)

### Technical Resources
- [NVIDIA-Accelerated Mistral 3 | NVIDIA Technical Blog](https://developer.nvidia.com/blog/nvidia-accelerated-mistral-3-open-models-deliver-efficiency-accuracy-at-any-scale/)
- [How to Run Mistral 3 Locally | Apidog](https://apidog.com/blog/run-mistral-3-locally/)
- [Mistral 3: Model Family, Benchmarks & More | DataCamp](https://www.datacamp.com/blog/mistral-3)
- [Ministral 3B GPU VRAM Requirements | APXML](https://apxml.com/models/ministral-3-3b)

### Prompting Guides
- [Mistral System Prompt Guide | PromptLayer](https://blog.promptlayer.com/mistral-system-prompt/)
- [Mistral 7B LLM | Prompt Engineering Guide](https://www.promptingguide.ai/models/mistral-7b)
- [How to Prompt Mistral AI | AWS Builder](https://builder.aws.com/content/2dFNOnLVQRhyrOrMsloofnW0ckZ/how-to-prompt-mistral-ai-models-and-why)

### Deployment & Benchmarks
- [Mistral AI Unveils Ministral 3B and 8B | DeepLearning.AI](https://www.deeplearning.ai/the-batch/mistral-ai-unveils-ministral-3b-and-8b-models-outperforming-rivals-in-small-scale-ai/)
- [Amazon Bedrock: Mistral Large 3 & Ministral 3 | AWS Blog](https://aws.amazon.com/blogs/aws/amazon-bedrock-adds-fully-managed-open-weight-models/)
- [Mistral 3 on Phones, Drones, Edge Devices | ThePromptBuddy](https://www.thepromptbuddy.com/prompts/mistral-3-open-models-run-advanced-ai-on-phones-drones-and-edge-devices)

### Research Papers
- [Small Language Models (SLMs) Survey | arXiv](https://arxiv.org/html/2501.05465v1)
- [Efficient Prompting Methods for LLMs | arXiv](https://arxiv.org/html/2404.01077v1)
- [SmolLM3-3B: Hybrid Reasoning | Cordatus Blog](https://blog.cordatus.ai/featured-articles/smollm3-3b-the-small-language-model-that-outperforms-its-class-with-hybrid-reasoning/)

---

**Document Version:** 1.0
**Last Updated:** 2025-01-18
**Model Version:** Ministral 3 3B Instruct 2512
**Research Methodology:** Wide-then-Deep parallel research (10 searches, 2 batches)