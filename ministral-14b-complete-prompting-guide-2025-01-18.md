---
title: Ministral 14B Complete Prompting Guide
date: 2025-01-18
model: mistralai/Ministral-3-14B (2512 release)
completeness: 95%
focus: Comprehensive prompting strategies, best practices, API usage, and deployment
---

# Ministral 14B Complete Prompting Guide

## Table of Contents

1. [Model Overview](#model-overview)
2. [Architecture & Specifications](#architecture--specifications)
3. [Model Variants: Base, Instruct, and Reasoning](#model-variants-base-instruct-and-reasoning)
4. [Prompting Best Practices](#prompting-best-practices)
5. [Chat Template & Special Tokens](#chat-template--special-tokens)
6. [System Prompts](#system-prompts)
7. [Temperature & Sampling Parameters](#temperature--sampling-parameters)
8. [Effective Prompting Techniques](#effective-prompting-techniques)
9. [Vision Capabilities & Multimodal Prompting](#vision-capabilities--multimodal-prompting)
10. [Function Calling & JSON Output](#function-calling--json-output)
11. [API Usage & Deployment](#api-usage--deployment)
12. [Known Limitations & Workarounds](#known-limitations--workarounds)
13. [Benchmarks & Comparisons](#benchmarks--comparisons)
14. [Practical Examples](#practical-examples)
15. [Fine-Tuning with LoRA/PEFT](#fine-tuning-with-lorapeft)
16. [Resources](#resources)

---

## Model Overview

**Ministral 14B** (officially `Ministral-3-14B-2512`) is the flagship model in Mistral AI's Ministral 3 family, released in December 2024/January 2025. It's a state-of-the-art small dense language model optimized for edge and local deployment while delivering performance comparable to much larger models.

### Key Highlights

- **Size**: 14B parameters (13.5B language core + 0.4B vision encoder)
- **Context Window**: 256,000 tokens (256K)
- **Multimodal**: Native vision support for text + image reasoning
- **License**: Apache 2.0 (fully open source)
- **Variants**: Base, Instruct, Reasoning (each optimized for different use cases)
- **Languages**: 40+ languages including English, French, Spanish, German, Italian, Portuguese, Dutch, Chinese, Japanese, Korean, Arabic
- **Special Features**:
  - Native function calling
  - JSON-structured output generation
  - Strong system prompt adherence
  - Best-in-class agentic capabilities

### Primary Use Cases

- Private/custom chat and AI assistant deployments
- Advanced local agentic systems
- Long-document analysis and synthesis
- Multimodal assistant (text + image reasoning)
- Code generation and debugging
- Mathematical and STEM problem-solving (Reasoning variant)
- Fine-tuning and specialization for domain-specific tasks
- Edge AI applications with constrained resources

---

## Architecture & Specifications

### Model Architecture

Ministral 14B combines a **13.5B dense language model** with a **0.4B vision encoder**, totaling approximately 14B parameters. This creates a unified interface for text-image reasoning tasks without requiring separate processing pipelines.

**Architecture Type**: Dense transformer (not Mixture-of-Experts)
**Attention Mechanism**: Multi-Head Attention (MHA)
**Training**: Trained on NVIDIA Hopper GPUs
**Precision**: Available in BF16 and FP8 quantized formats

### Hardware Requirements

| Precision | VRAM Required | Notes |
|-----------|---------------|-------|
| BF16 | 32GB | Full precision |
| FP8 | 24GB | Recommended for most deployments |
| Quantized (4-bit) | <16GB | Using GGUF or similar formats |

**Minimum Deployment**: Single GPU with 24GB VRAM (FP8)
**Recommended**: NVIDIA H100/H200 or RTX 5090 for optimal performance

### Context Window

- **Maximum Context**: 262,144 tokens (256K)
- **Recommended Output Length**:
  - Instruct variant: 16,384 tokens
  - Reasoning variant: 32,768 tokens (can be increased for complex reasoning)

### Performance

- **Inference Speed**: Up to 385 tokens/second on RTX 5090 GPU
- **AIME 2025 Score**: 85% (Reasoning variant) - state-of-the-art for 14B class
- **General Knowledge**: 98.5% accuracy
- **Ethics**: 99.0% accuracy
- **Coding**: 87.0% accuracy

---

## Model Variants: Base, Instruct, and Reasoning

Ministral 3 comes in three specialized variants for different use cases:

### 1. Base Model (`Ministral-3-14B-Base-2512`)

**Purpose**: Foundation model for fine-tuning and specialized adaptation
**Format**: BF16 weights
**Best For**:
- Custom fine-tuning for specialized domains
- Research and experimentation
- Building task-specific models

**When NOT to Use**: If you need immediate out-of-box functionality without additional training

### 2. Instruct Model (`Ministral-3-14B-Instruct-2512`)

**Purpose**: Optimized for general conversational AI and instruction-following
**Format**: FP8 weights (recommended) or BF16
**Best For**:
- Chatbots and virtual assistants
- General conversational AI
- Instruction-based tasks
- Ready-to-use applications without fine-tuning
- Lower latency requirements

**Recommended Settings**:
- Temperature: 0.1 - 0.15 (low for consistency)
- Output Length: 16,384 tokens
- Top-P: 0.9 - 0.95

### 3. Reasoning Model (`Ministral-3-14B-Reasoning-2512`)

**Purpose**: Post-trained for multi-step logical reasoning, math, and code generation
**Format**: BF16 weights
**Best For**:
- Mathematical problem-solving (STEM tasks)
- Complex coding and debugging
- Multi-step logical reasoning
- Tasks requiring extended "thinking time"
- Scientific and analytical workflows

**Recommended Settings**:
- Temperature: 0.7 - 1.0 (higher for diverse reasoning paths)
- Output Length: 32,768 tokens
- Top-P: 0.95
- **Important**: Keep reasoning traces in context for multi-turn conversations

**Special Feature**: Uses `[THINK]` and `[/THINK]` tags to separate reasoning content from final answers. The model can "think longer" for higher accuracy on complex problems.

### Variant Comparison Table

| Feature | Base | Instruct | Reasoning |
|---------|------|----------|-----------|
| Out-of-box usability | No | Yes | Yes |
| Fine-tuning ready | Yes | Optional | Optional |
| Conversational fluency | N/A | High | Medium |
| Reasoning depth | N/A | Medium | High |
| Math/STEM performance | N/A | Good | Excellent (85% AIME) |
| Recommended temp | N/A | 0.1-0.15 | 0.7-1.0 |
| Output length | N/A | 16K | 32K |

---

## Prompting Best Practices

### General Guidelines

1. **Be Specific and Clear**: Ministral 14B excels when given precise instructions
2. **Use System Prompts**: The model has strong adherence to system prompts - leverage them
3. **Limit Tool Sets**: For agentic applications, keep tools to 3-7 well-defined functions
4. **Provide Examples**: Few-shot examples significantly improve performance on specialized tasks
5. **Control Temperature**: Use low temps (0.1-0.15) for consistent outputs, higher (0.7-1.0) for creativity
6. **Leverage Context Window**: With 256K context, you can include entire codebases or lengthy documents

### Model-Specific Recommendations

**For Instruct Variant**:
- Use temperature below 0.1 for production environments
- Define clear environment and use case in system prompt
- Include guidance on tool usage for agentic systems
- Validate JSON outputs before execution

**For Reasoning Variant**:
- Use temperature of 1.0 for most environments
- Keep reasoning traces in context during multi-turn conversations
- Use the provided system prompt and append your custom instructions
- Allow sufficient output length (32K tokens recommended)

### Prompt Structure

```
<s>[INST] {Your instruction here} [/INST] {Model response}</s>[INST] {Follow-up instruction} [/INST]
```

- `<s>` and `</s>` are special BOS (beginning of string) and EOS (end of string) tokens
- `[INST]` and `[/INST]` are regular string delimiters indicating user instructions
- The very first instruction begins with BOS `<s>`, subsequent instructions do not
- System prompts are prepended to the last user message by default (in newer tokenizers)

---

## Chat Template & Special Tokens

### Mistral Chat Template Format

Ministral 14B uses Mistral's standard instruction format with specific tokens:

**Basic Template**:
```
<s>[INST] User message [/INST] Assistant response</s>[INST] Follow-up message [/INST]
```

**Multi-turn Conversation**:
```
<s>[INST] First user message [/INST] First assistant response</s>
[INST] Second user message [/INST] Second assistant response</s>
[INST] Third user message [/INST]
```

### Special Tokens

- `<s>`: Beginning of string (BOS) - marks the start of a conversation
- `</s>`: End of string (EOS) - marks the end of a response
- `[INST]`: Start of user instruction (regular string, not a special token)
- `[/INST]`: End of user instruction (regular string, not a special token)
- `[THINK]`: Start of reasoning trace (Reasoning variant only)
- `[/THINK]`: End of reasoning trace (Reasoning variant only)

### Whitespace Handling

Important: The tokenizer adds leading whitespaces by default (from SentencePiece).

When encoding: `[INST] user message [/INST]`
Actually encodes: `_[INST] user message [/INST]` (where `_` represents whitespace)

**Critical**: The model is quite sensitive to token layout - even missing or additional spaces can impact output quality.

### Using Chat Templates with Transformers

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("mistralai/Ministral-3-14B-Instruct-2512")

messages = [
    {"role": "user", "content": "What is the capital of France?"}
]

formatted_prompt = tokenizer.apply_chat_template(messages, tokenize=False)
```

**Note**: Ensure `mistral-common >= 1.8.6` is installed for proper tokenizer support.

### Tokenizer Versions

- **V2 Tokenizer**: Introduced control tokens for `[INST]`/`[/INST]`, system prompt prepended to last user message
- **V3 Tokenizer**: Powers Ministral models, similar to V2 with adjusted tool use
- **Tekken Tokenizer**: Newer tiktoken-based tokenizer (used by some Mistral models), larger vocabulary, no leading whitespaces

---

## System Prompts

Ministral 14B has **strong adherence to system prompts**, making them essential for optimal performance.

### System Prompt Best Practices

1. **Define Clear Environment**: Specify the role, context, and objectives
2. **Include Tool Guidance**: For agentic systems, describe when and how to use tools
3. **Set Behavioral Guidelines**: Define tone, format, and constraints
4. **Provide Examples**: Include sample interactions for complex tasks
5. **Keep It Concise**: While the model supports 256K context, focused system prompts perform better

### System Prompt Template

```
You are {role definition}.

Your primary objectives:
- {Objective 1}
- {Objective 2}
- {Objective 3}

Guidelines:
- {Guideline 1}
- {Guideline 2}

Available tools: {tool descriptions}

Always respond in {format} and follow {specific constraints}.
```

### Example: Code Assistant System Prompt

```
You are an expert Python developer specializing in clean, maintainable code.

Your primary objectives:
- Write production-ready code with proper error handling
- Follow PEP 8 style guidelines
- Provide clear docstrings and comments
- Suggest performance optimizations when relevant

Guidelines:
- Always validate inputs and handle edge cases
- Use type hints for function signatures
- Prefer built-in libraries over external dependencies
- Explain your reasoning for design decisions

Available tools: code_analyzer, linter, test_runner

Always respond with code blocks using markdown formatting.
```

### Example: Reasoning Assistant System Prompt (Reasoning Variant)

```
You are a mathematical reasoning assistant.

When solving problems:
1. Break down the problem into clear steps
2. Show your reasoning process using [THINK] tags
3. Verify your calculations
4. Provide the final answer separately

For multi-step problems:
- State assumptions clearly
- Check intermediate results
- Use multiple approaches when possible

Think carefully through each problem before providing your final answer.
```

### System Prompt Files

Mistral provides `SYSTEM_PROMPT.txt` files with recommended system prompts for each variant. You should append these to your custom system prompts for best results (especially for the Reasoning variant).

---

## Temperature & Sampling Parameters

### Temperature

Temperature controls randomness/creativity in the model's outputs by scaling log probabilities.

**How It Works**:
- Low temperature (0.0-0.3): More deterministic, focused, predictable
- Medium temperature (0.4-0.7): Balanced creativity and coherence
- High temperature (0.8-2.0): More diverse, creative, potentially less coherent

**Ministral 14B Recommendations**:

| Variant | Recommended Temp | Use Case |
|---------|------------------|----------|
| Instruct | 0.1 - 0.15 | Production, daily-driver environments |
| Instruct | 0.5 - 0.8 | Creative writing, brainstorming |
| Reasoning | 0.7 - 1.0 | Math, logic, multi-step reasoning |
| Reasoning | 1.0 | Default for most reasoning tasks |

### Top-P (Nucleus Sampling)

Controls the cumulative probability threshold for token selection.

**How It Works**:
- Low Top-P (0.3-0.5): Only considers very high-probability tokens (focused)
- Medium Top-P (0.7-0.9): Balanced diversity
- High Top-P (0.9-0.99): Broader token selection (more varied)

**Recommendations**:
- **Instruct**: 0.9 - 0.95 (standard)
- **Reasoning**: 0.95 (recommended by Mistral)

### Top-K

Limits sampling to the K most probable tokens.

**How It Works**:
- Lower K (10-20): More focused, less random
- Medium K (40-50): Standard setting
- Higher K (80-100): More diverse options

**Typical Range**: 40-50 for most use cases

### Parameter Combinations

**For Factual/Consistent Output**:
```json
{
  "temperature": 0.1,
  "top_p": 0.9,
  "top_k": 40
}
```

**For Creative Tasks**:
```json
{
  "temperature": 0.8,
  "top_p": 0.95,
  "top_k": 60
}
```

**For Reasoning Tasks (Reasoning Variant)**:
```json
{
  "temperature": 1.0,
  "top_p": 0.95,
  "top_k": 50
}
```

**For Code Generation**:
```json
{
  "temperature": 0.2,
  "top_p": 0.9,
  "top_k": 40
}
```

### Context Length Settings

- **Default max-model-len**: 262,144 tokens (quite large, not necessary for most scenarios)
- **Recommended**: Set `--max-model-len` based on your actual needs to preserve memory
- **Batching**: Use `--max-num-batched-tokens` to balance throughput and latency

---

## Effective Prompting Techniques

### Zero-Shot Prompting

Direct instruction without examples. Ministral 14B performs well on zero-shot tasks due to strong instruction-tuning.

**Example**:
```
[INST] Translate the following English text to French: "Good morning, how are you?" [/INST]
```

**Best For**: Simple, well-defined tasks; general knowledge queries; straightforward instructions.

### Few-Shot Prompting

Provide 2-5 examples before the actual task to guide the model's behavior.

**Example**:
```
[INST] Convert these measurements:

Q: 5 feet to meters
A: 1.524 meters

Q: 100 pounds to kilograms
A: 45.36 kilograms

Q: 3 gallons to liters
A: [/INST]
```

**Best For**: Specialized formats; domain-specific tasks; consistent output structure; edge cases.

### Chain-of-Thought (CoT) Prompting

Encourage the model to show step-by-step reasoning.

**Zero-Shot CoT**:
```
[INST] Solve this problem. Think step by step.

If a train travels 120 miles in 2 hours, and then 180 miles in 3 hours, what is its average speed? [/INST]
```

**Few-Shot CoT**:
```
[INST] Solve these problems:

Q: A store has 45 apples. They sell 17. How many remain?
Reasoning: Start with 45 apples. Subtract 17 sold. 45 - 17 = 28
A: 28 apples

Q: A recipe needs 3 eggs per cake. How many eggs for 7 cakes?
Reasoning: [/INST]
```

**Best For**: Math problems; logical reasoning; debugging; complex multi-step tasks.

**Special Note for Reasoning Variant**: The model uses `[THINK]...[/THINK]` tags automatically for internal reasoning. You can prompt it to expose this reasoning or let it work internally.

### Role-Playing and Persona Prompts

Assign the model a specific role or expertise.

**Example**:
```
[INST] You are a senior DevOps engineer with 10 years of experience in Kubernetes.

Explain how to debug a CrashLoopBackOff error in a production pod. [/INST]
```

**Best For**: Domain expertise; tone control; specialized knowledge; educational content.

### Task-Specific Prompting

#### Code Generation
```
[INST] Write a Python function that:
- Takes a list of integers as input
- Returns the list sorted in descending order
- Handles empty lists gracefully
- Includes docstring and type hints [/INST]
```

#### Analysis
```
[INST] Analyze the following customer feedback and extract:
1. Main sentiment (positive/negative/neutral)
2. Key issues mentioned
3. Suggested improvements

Feedback: "The app is great but crashes when uploading large files..." [/INST]
```

#### Creative Writing
```
[INST] Write a compelling product description for an AI-powered calendar app that:
- Highlights automation features
- Uses conversational tone
- Includes a call-to-action
- Max 150 words [/INST]
```

#### Document Summarization (Leverage 256K Context)
```
[INST] Summarize the following technical documentation in 3-5 bullet points, focusing on:
- Key features
- Breaking changes
- Migration steps

{paste entire documentation here - up to 256K tokens} [/INST]
```

### Structured Output Prompting

Request specific formats like JSON, markdown tables, or code.

**JSON Output**:
```
[INST] Extract entities from this text and return as JSON:

Text: "Apple Inc. announced a new iPhone model on September 12th in Cupertino."

Format:
{
  "organizations": [],
  "products": [],
  "dates": [],
  "locations": []
} [/INST]
```

**Markdown Table**:
```
[INST] Compare these three programming languages in a markdown table with columns: Language, Typing, Best Use Case

Languages: Python, Rust, JavaScript [/INST]
```

---

## Vision Capabilities & Multimodal Prompting

Ministral 14B includes a **0.4B vision encoder** for native multimodal understanding.

### Vision Features

- **Supported Input**: Up to 10 images per prompt
- **Image Types**: Screenshots, diagrams, documents, technical drawings, photos
- **Use Cases**:
  - Visual question answering
  - Diagram analysis
  - Document understanding with mixed content
  - Technical drawing interpretation
  - OCR and text extraction from images
  - Visual debugging (screenshot analysis)

### Image Prompting Best Practices

1. **Aspect Ratio**: Maintain close to 1:1 (width-to-height) for optimal performance
2. **Avoid Extreme Ratios**: Crop overly thin or wide images before processing
3. **Image Quality**: Higher resolution increases VRAM consumption - balance quality with resources
4. **Context**: Provide clear text instructions alongside images

### Multimodal Prompt Examples

**Visual Question Answering**:
```
[INST] {image of a circuit diagram}

Analyze this circuit diagram and explain:
1. What type of circuit is this?
2. What is the purpose of the capacitor?
3. Calculate the total resistance if R1=100Ω and R2=220Ω [/INST]
```

**Document Analysis**:
```
[INST] {image of invoice}

Extract the following information from this invoice:
- Invoice number
- Date
- Total amount
- Line items with quantities and prices

Return as JSON. [/INST]
```

**Screenshot Debugging**:
```
[INST] {screenshot of error message}

This error appears when deploying my app. What is the root cause and how do I fix it? [/INST]
```

**Diagram Understanding**:
```
[INST] {image of database schema}

Describe this database schema and identify:
- Primary and foreign key relationships
- Potential normalization issues
- Suggested indexes for performance [/INST]
```

### Vision Limitations

- Model processes images through 0.4B encoder (smaller than dedicated vision models)
- Best for technical/analytical visual tasks rather than creative image generation
- Performance varies based on image complexity and clarity

---

## Function Calling & JSON Output

Ministral 14B has **best-in-class agentic capabilities** with native function calling and JSON output.

### Function Calling Overview

The model can:
- Parse tool/function definitions from system prompts
- Determine when to call which function
- Generate structured JSON outputs with function names and parameters
- Handle multi-step tool orchestration workflows

### Best Practices for Function Calling

1. **Limit Tool Set**: Keep to 3-7 well-defined functions (avoid overloading)
2. **Clear Descriptions**: Provide explicit function descriptions and parameter schemas
3. **Include Examples**: Show sample function calls in system prompts
4. **Validate Outputs**: Always validate JSON before execution
5. **Error Handling**: Implement robust error handling for unexpected responses
6. **Tool Guidance**: Define in system prompt when and how to use each tool

### Function Definition Format

Define tools with clear schemas in your system prompt:

```json
{
  "name": "get_weather",
  "description": "Retrieve current weather for a location",
  "parameters": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "City name or zip code"
      },
      "unit": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "Temperature unit"
      }
    },
    "required": ["location"]
  }
}
```

### System Prompt with Tools

```
You are an AI assistant with access to the following tools:

1. get_weather(location: str, unit: str = "celsius")
   - Retrieves current weather for the specified location
   - Returns: temperature, conditions, humidity

2. search_database(query: str, limit: int = 10)
   - Searches the knowledge database
   - Returns: list of relevant documents

3. send_email(to: str, subject: str, body: str)
   - Sends an email
   - Returns: success status

When a user request requires a tool, respond with JSON in this format:
{
  "function": "function_name",
  "parameters": { ... }
}

Use tools only when necessary. For general questions, respond normally.
```

### Example Interaction

**User**: "What's the weather in Paris?"

**Model Output**:
```json
{
  "function": "get_weather",
  "parameters": {
    "location": "Paris",
    "unit": "celsius"
  }
}
```

**After Function Execution** (you provide result):
```
[INST] Tool result: {"temperature": 18, "conditions": "partly cloudy", "humidity": 65} [/INST]

The weather in Paris is currently 18°C with partly cloudy conditions and 65% humidity.
```

### Enabling Function Calling with vLLM

```bash
vllm serve mistralai/Ministral-3-14B-Instruct-2512 \
  --tokenizer_mode mistral \
  --config_format mistral \
  --load_format mistral \
  --enable-auto-tool-choice \
  --tool-call-parser mistral
```

### JSON Output Without Function Calling

You can also request structured JSON for data extraction:

```
[INST] Extract product information from this description and return as JSON:

"MacBook Pro 16-inch, M3 Max chip, 32GB RAM, 1TB SSD, Space Black - $3,499"

Format:
{
  "product": "",
  "specs": {},
  "color": "",
  "price": 0
} [/INST]
```

**Model Response**:
```json
{
  "product": "MacBook Pro 16-inch",
  "specs": {
    "chip": "M3 Max",
    "ram": "32GB",
    "storage": "1TB SSD"
  },
  "color": "Space Black",
  "price": 3499
}
```

---

## API Usage & Deployment

### Available Platforms

Ministral 14B is available on:
- **Mistral AI Studio** (official API)
- **Hugging Face** (model weights + Inference API)
- **Together AI** (API endpoint)
- **Amazon Bedrock**
- **Azure AI Foundry**
- **NVIDIA NIM**
- **Modal**
- **OpenRouter**
- **Fireworks AI**
- **Ollama** (local deployment)
- **LM Studio** (local deployment)

### Mistral API

**Endpoint**: Use Mistral's official API with model identifier `ministral-14b-instruct-2512`

**Pricing** (as of Dec 2024): $0.20 per million tokens (input and output)

**Example**:
```python
import os
from mistralai.client import MistralClient

client = MistralClient(api_key=os.environ["MISTRAL_API_KEY"])

response = client.chat(
    model="ministral-14b-instruct-2512",
    messages=[
        {"role": "user", "content": "Explain quantum computing in simple terms"}
    ],
    temperature=0.15,
    max_tokens=500
)

print(response.choices[0].message.content)
```

### Together AI API

**Endpoint**: `https://api.together.xyz/v1/chat/completions`
**Model ID**: `mistralai/Ministral-3-14B-Instruct-2512`

**Example**:
```python
import requests

url = "https://api.together.xyz/v1/chat/completions"
headers = {
    "Authorization": f"Bearer {TOGETHER_API_KEY}",
    "Content-Type": "application/json"
}

payload = {
    "model": "mistralai/Ministral-3-14B-Instruct-2512",
    "messages": [
        {"role": "user", "content": "Write a Python function to reverse a string"}
    ],
    "temperature": 0.1,
    "max_tokens": 1024
}

response = requests.post(url, json=payload, headers=headers)
print(response.json()["choices"][0]["message"]["content"])
```

### Hugging Face Transformers

**Installation**:
```bash
pip install transformers mistral-common>=1.8.6 torch
```

**Usage**:
```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model_id = "mistralai/Ministral-3-14B-Instruct-2512"

tokenizer = AutoTokenizer.from_pretrained(model_id)
model = AutoModelForCausalLM.from_pretrained(
    model_id,
    device_map="auto",
    torch_dtype="auto"
)

messages = [
    {"role": "user", "content": "What is the capital of France?"}
]

inputs = tokenizer.apply_chat_template(
    messages,
    return_tensors="pt",
    add_generation_prompt=True
).to(model.device)

outputs = model.generate(
    inputs,
    max_new_tokens=256,
    temperature=0.15,
    do_sample=True
)

response = tokenizer.decode(outputs[0], skip_special_tokens=True)
print(response)
```

### vLLM Deployment (Recommended)

vLLM is **recommended by Mistral** for optimal performance and OpenAI API compatibility.

**Installation**:
```bash
pip install vllm
```

**Serving Instruct Variant**:
```bash
vllm serve mistralai/Ministral-3-14B-Instruct-2512 \
  --tokenizer_mode mistral \
  --config_format mistral \
  --load_format mistral \
  --enable-auto-tool-choice \
  --tool-call-parser mistral \
  --max-model-len 131072  # Adjust based on needs (default is 262144)
```

**Serving Reasoning Variant**:
```bash
vllm serve mistralai/Ministral-3-14B-Reasoning-2512 \
  --tensor-parallel-size 2 \  # For 2xH200 GPUs
  --tokenizer_mode mistral \
  --config_format mistral \
  --load_format mistral \
  --enable-auto-tool-choice \
  --tool-call-parser mistral \
  --reasoning-parser mistral
```

**OpenAI-Compatible Client**:
```python
from openai import OpenAI

client = OpenAI(
    api_key="EMPTY",
    base_url="http://localhost:8000/v1"
)

response = client.chat.completions.create(
    model="mistralai/Ministral-3-14B-Instruct-2512",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain async/await in Python"}
    ],
    temperature=0.15,
    max_tokens=1024
)

print(response.choices[0].message.content)
```

### Ollama (Local Deployment)

**Installation**:
```bash
ollama pull ministral-3:14b-instruct-2512-fp16
# or for quantized version:
ollama pull ministral-3:14b
```

**Usage**:
```bash
ollama run ministral-3:14b-instruct-2512-fp16
```

**API**:
```python
import requests

response = requests.post(
    "http://localhost:11434/api/chat",
    json={
        "model": "ministral-3:14b-instruct-2512-fp16",
        "messages": [
            {"role": "user", "content": "Hello!"}
        ]
    }
)
print(response.json())
```

### Modal Serverless Deployment

Modal enables serverless deployment with autoscaling:

```python
import modal

app = modal.App("ministral-14b")

@app.function(
    gpu="H100",
    timeout=600,
    container_idle_timeout=60
)
def serve_ministral():
    from vllm import LLM

    llm = LLM(
        model="mistralai/Ministral-3-14B-Instruct-2512",
        tokenizer_mode="mistral",
        config_format="mistral",
        load_format="mistral"
    )
    return llm

@app.local_entrypoint()
def main():
    llm = serve_ministral.remote()
    # Use the model
```

### Hardware Recommendations by Deployment Type

| Deployment | GPU | VRAM | Precision | Notes |
|------------|-----|------|-----------|-------|
| Local Development | RTX 4090 | 24GB | FP8 | Single user |
| Production (Small) | H100 | 80GB | FP8 | Multi-user |
| Production (Large) | 2xH200 | 141GB each | BF16 | Full context (256K) |
| Edge Deployment | RTX 5090 | 32GB | 4-bit GGUF | Quantized |
| Serverless | H100 (Modal/Modal) | 80GB | FP8 | Auto-scaling |

---

## Known Limitations & Workarounds

### Hardware & Memory Limitations

**Issue**: High VRAM requirements (24GB minimum for FP8)

**Workaround**:
- Use quantized formats (GGUF 4-bit/8-bit) to reduce VRAM to <16GB
- Deploy via cloud APIs (Mistral, Together AI) instead of local
- Use smaller Ministral variants (8B or 3B) if 14B is too large

**Issue**: Ollama "failed to commit memory" errors on Windows 11

**Workaround**:
- Close other GPU-intensive applications
- Increase virtual memory/page file size
- Use GGUF quantized models instead of full precision
- Update to latest Ollama version

### Context Length Limitations

**Issue**: Default 262K context consumes excessive memory

**Workaround**:
- Set `--max-model-len` to actual needs (e.g., 16K, 32K, 64K)
- For Reasoning variant, use 2xGPUs if full 256K context is needed
- Process long documents in chunks if full context isn't required

### Vision Limitations

**Issue**: Vision encoder is relatively small (0.4B) compared to dedicated vision models

**Workaround**:
- Best for technical/analytical visual tasks (diagrams, documents, screenshots)
- For creative visual understanding, consider dedicated vision models
- Maintain 1:1 aspect ratio for images
- Crop or resize extreme aspect ratios before processing

### Tokenizer Sensitivity

**Issue**: Model sensitive to whitespace and token layout

**Workaround**:
- Use `tokenizer.apply_chat_template()` instead of manual formatting
- Ensure `mistral-common>=1.8.6` is installed
- Test prompts with minor variations if getting unexpected results

### Deployment Dependencies

**Issue**: Requires `mistral-common>=1.8.6` for tokenizer support

**Workaround**:
```bash
pip install mistral-common>=1.8.6
```

**Issue**: vLLM requires specific flags for proper operation

**Workaround**: Always include minimum flags:
```bash
--tokenizer_mode mistral --config_format mistral --load_format mistral
```

### Model-Specific Limitations

**General Observations**:
- Not as strong as 70B+ models for highly complex reasoning (though excellent for 14B class)
- Creative writing may be less "human-like" than specialized creative models
- World knowledge cutoff (early 2024 based on training)

**Workarounds**:
- For cutting-edge information, integrate with web search or RAG
- For creative tasks, use higher temperature (0.7-0.9) and creative system prompts
- Chain multiple prompts for very complex reasoning tasks

---

## Benchmarks & Comparisons

### Ministral 14B Performance

| Benchmark | Score | Notes |
|-----------|-------|-------|
| AIME 2025 (Reasoning) | 85.0% | State-of-the-art for 14B class |
| General Knowledge | 98.5% | Excellent factual accuracy |
| Ethics | 99.0% | Strong alignment |
| Coding | 87.0% | Strong code generation |
| MMLU | High | Competitive with larger models |

### Comparison with Other 14B-Class Models

**Ministral 14B vs. Gemma 2 9B**:
- Ministral 14B: Better multilingual support (40+ languages), native vision, larger context (256K vs 8K)
- Gemma 2 9B: Slightly smaller, efficient for single-language tasks

**Ministral 14B vs. Phi-3 Medium 14B**:
- Ministral 14B: Superior multilingual, vision capabilities, better for general chat
- Phi-3 Medium: Strong reasoning on benchmarks but weaker world knowledge (3.0/100 SimpleQA vs 7.6 for Phi-3)

**Ministral 14B vs. Qwen 2.5 14B**:
- Ministral 14B: Better AIME score (85.0 vs 73.7 for Qwen "Thinking"), native vision
- Qwen 2.5 14B: Strong overall performance, good for specific tasks

**Ministral 14B vs. Llama 3.1 8B**:
- Ministral 14B: Larger, better performance on complex tasks, vision support
- Llama 3.1 8B: Smaller, faster, good for resource-constrained environments

### Key Differentiators

1. **Vision**: Native multimodal support (most competitors are text-only at this size)
2. **Context Window**: 256K tokens (far exceeds most 14B-class models)
3. **Multilingual**: Strong support for 40+ languages
4. **Reasoning Variant**: Specialized model with 85% AIME score
5. **License**: Apache 2.0 (fully open, unlike some competitors)
6. **Inference Speed**: Up to 385 tok/s on RTX 5090

### When to Choose Ministral 14B

**Choose Ministral 14B if**:
- You need multimodal (text + vision) capabilities
- Large context window (256K) is valuable for your use case
- Multilingual support is important
- Local/edge deployment with strong performance
- Apache 2.0 license is required
- You need specialized reasoning variant

**Choose alternatives if**:
- You only need text (no vision): Consider Llama 3.1 8B or Qwen 2.5 14B
- Budget is extremely tight: Use smaller models (3B/8B variants)
- You need absolute cutting-edge performance: Consider 70B+ models or proprietary APIs

---

## Practical Examples

### Example 1: Code Generation (Instruct Variant)

**Prompt**:
```
[INST] Write a Python function that implements a retry decorator with exponential backoff.

Requirements:
- Configurable max retries (default: 3)
- Exponential backoff (2^attempt seconds)
- Catch specific exceptions only
- Log retry attempts
- Include type hints and docstring [/INST]
```

**Response**:
```python
import time
import logging
from functools import wraps
from typing import Callable, Type, Tuple, Any

logger = logging.getLogger(__name__)

def retry_with_backoff(
    max_retries: int = 3,
    exceptions: Tuple[Type[Exception], ...] = (Exception,)
) -> Callable:
    """
    Decorator that retries a function with exponential backoff.

    Args:
        max_retries: Maximum number of retry attempts (default: 3)
        exceptions: Tuple of exception types to catch (default: all exceptions)

    Returns:
        Decorated function with retry logic

    Example:
        @retry_with_backoff(max_retries=5, exceptions=(ConnectionError, TimeoutError))
        def fetch_data(url: str) -> dict:
            # function implementation
            pass
    """
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    if attempt == max_retries - 1:
                        logger.error(f"Final attempt failed: {e}")
                        raise

                    wait_time = 2 ** attempt
                    logger.warning(
                        f"Attempt {attempt + 1}/{max_retries} failed: {e}. "
                        f"Retrying in {wait_time}s..."
                    )
                    time.sleep(wait_time)

        return wrapper
    return decorator
```

**Settings Used**: `temperature=0.1`, `max_tokens=1024`

---

### Example 2: Mathematical Reasoning (Reasoning Variant)

**Prompt**:
```
[INST] Solve this problem step by step:

A rectangle has a perimeter of 50 cm. If the length is 5 cm more than twice the width, what are the dimensions of the rectangle? [/INST]
```

**Response**:
```
[THINK]
Let me define variables:
- Let w = width of the rectangle
- Let l = length of the rectangle

Given information:
1. Perimeter = 50 cm
2. Length = 2 × width + 5

Step 1: Write perimeter formula
Perimeter = 2(length + width)
50 = 2(l + w)
25 = l + w

Step 2: Express length in terms of width
l = 2w + 5

Step 3: Substitute into perimeter equation
25 = (2w + 5) + w
25 = 3w + 5
20 = 3w
w = 20/3 = 6.67 cm

Step 4: Find length
l = 2(6.67) + 5
l = 13.33 + 5
l = 18.33 cm

Step 5: Verify
Perimeter = 2(18.33 + 6.67) = 2(25) = 50 ✓
[/THINK]

The dimensions of the rectangle are:
- Width: 6.67 cm (or 20/3 cm)
- Length: 18.33 cm (or 55/3 cm)
```

**Settings Used**: `temperature=1.0`, `max_tokens=2048`

---

### Example 3: Vision Analysis (Instruct Variant)

**Prompt**:
```
[INST] {image of database ER diagram}

Analyze this entity-relationship diagram and provide:
1. List of entities and their attributes
2. Relationships and cardinalities
3. Potential normalization issues
4. Recommended indexes for performance [/INST]
```

**Response**:
```
Based on the ER diagram:

**Entities and Attributes:**

1. **Users**
   - user_id (PK)
   - username
   - email
   - created_at

2. **Orders**
   - order_id (PK)
   - user_id (FK)
   - order_date
   - total_amount
   - status

3. **Products**
   - product_id (PK)
   - name
   - price
   - category
   - stock_quantity

4. **OrderItems**
   - order_item_id (PK)
   - order_id (FK)
   - product_id (FK)
   - quantity
   - unit_price

**Relationships:**

- Users → Orders: One-to-Many (1:N)
  A user can have multiple orders

- Orders → OrderItems: One-to-Many (1:N)
  An order can contain multiple items

- Products → OrderItems: One-to-Many (1:N)
  A product can appear in multiple orders

**Normalization Issues:**

1. **Potential Denormalization**: `unit_price` in OrderItems duplicates data from Products table
   - Reason: Historical pricing (price at time of order)
   - Status: Acceptable for this use case

2. **Category in Products**: Should be normalized to separate Categories table if there are many products per category

**Recommended Indexes:**

1. `CREATE INDEX idx_orders_user_id ON Orders(user_id)` - For user order lookups
2. `CREATE INDEX idx_orders_date ON Orders(order_date)` - For date-range queries
3. `CREATE INDEX idx_orderitems_order ON OrderItems(order_id)` - For order details
4. `CREATE INDEX idx_orderitems_product ON OrderItems(product_id)` - For product sales analytics
5. `CREATE INDEX idx_products_category ON Products(category)` - For category browsing
```

**Settings Used**: `temperature=0.15`, `max_tokens=2048`, aspect ratio maintained at 1:1

---

### Example 4: JSON Extraction (Instruct Variant)

**Prompt**:
```
[INST] Extract structured information from this job posting and return as JSON:

"Senior Full-Stack Developer at TechCorp (Remote)
$120,000 - $160,000/year

Requirements:
- 5+ years experience with React and Node.js
- Strong TypeScript skills
- Experience with PostgreSQL and Redis
- AWS/Azure cloud experience preferred

Benefits: Health insurance, 401k matching, unlimited PTO

Apply by: January 31, 2025
Contact: jobs@techcorp.com"

Format:
{
  "title": "",
  "company": "",
  "location": "",
  "salary": {"min": 0, "max": 0, "currency": ""},
  "requirements": [],
  "benefits": [],
  "deadline": "",
  "contact": ""
} [/INST]
```

**Response**:
```json
{
  "title": "Senior Full-Stack Developer",
  "company": "TechCorp",
  "location": "Remote",
  "salary": {
    "min": 120000,
    "max": 160000,
    "currency": "USD"
  },
  "requirements": [
    "5+ years experience with React and Node.js",
    "Strong TypeScript skills",
    "Experience with PostgreSQL and Redis",
    "AWS/Azure cloud experience preferred"
  ],
  "benefits": [
    "Health insurance",
    "401k matching",
    "Unlimited PTO"
  ],
  "deadline": "2025-01-31",
  "contact": "jobs@techcorp.com"
}
```

**Settings Used**: `temperature=0.1`, `max_tokens=512`

---

### Example 5: Function Calling (Instruct Variant)

**System Prompt**:
```
You are a weather assistant with access to the following tools:

1. get_current_weather(location: str, unit: str = "celsius")
   - Returns current weather conditions

2. get_forecast(location: str, days: int = 7)
   - Returns weather forecast for specified days

3. get_air_quality(location: str)
   - Returns air quality index and pollutant levels

When users ask weather-related questions, use the appropriate tool. Respond with JSON function calls when needed.
```

**User Prompt**:
```
[INST] What's the weather like in Tokyo for the next 5 days? [/INST]
```

**Model Response**:
```json
{
  "function": "get_forecast",
  "parameters": {
    "location": "Tokyo",
    "days": 5
  }
}
```

**After Tool Execution**:
```
[INST] Tool result:
{
  "location": "Tokyo, Japan",
  "forecast": [
    {"date": "2025-01-19", "high": 12, "low": 5, "conditions": "Sunny"},
    {"date": "2025-01-20", "high": 14, "low": 6, "conditions": "Partly Cloudy"},
    {"date": "2025-01-21", "high": 11, "low": 4, "conditions": "Rain"},
    {"date": "2025-01-22", "high": 9, "low": 3, "conditions": "Cloudy"},
    {"date": "2025-01-23", "high": 13, "low": 5, "conditions": "Sunny"}
  ]
} [/INST]

Here's the 5-day forecast for Tokyo, Japan:

- **Jan 19**: Sunny, High 12°C / Low 5°C
- **Jan 20**: Partly Cloudy, High 14°C / Low 6°C
- **Jan 21**: Rain, High 11°C / Low 4°C
- **Jan 22**: Cloudy, High 9°C / Low 3°C
- **Jan 23**: Sunny, High 13°C / Low 5°C

Expect rain on January 21st, so bring an umbrella! Otherwise, mostly pleasant weather with temperatures ranging from 3-14°C.
```

**Settings Used**: `temperature=0.1`, `max_tokens=1024`, function calling enabled

---

### Example 6: Creative Writing with Higher Temperature (Instruct Variant)

**Prompt**:
```
[INST] Write a compelling 100-word product description for a smart fitness mirror that uses AI to provide real-time form corrections during workouts. Target audience: busy professionals. Tone: motivational and modern. [/INST]
```

**Response**:
```
Transform your home into a personal gym with the AI Fitness Mirror—your intelligent workout companion. Using advanced computer vision and real-time biomechanics analysis, it watches your form and delivers instant corrections, ensuring every rep counts. Perfect for busy professionals, enjoy guided strength training, yoga, HIIT, and more—all from your living room. The sleek, space-saving design blends seamlessly with your decor when not in use. No more guessing if you're doing it right. Train smarter, not harder, and achieve your fitness goals faster with personalized coaching that adapts to your progress. Your transformation starts now.
```

**Settings Used**: `temperature=0.8`, `max_tokens=256`

---

### Example 7: Multilingual Support (Instruct Variant)

**Prompt**:
```
[INST] Translate the following product description to French, German, and Japanese:

"Our eco-friendly water bottle keeps drinks cold for 24 hours and hot for 12 hours. Made from recycled stainless steel." [/INST]
```

**Response**:
```
**French:**
Notre bouteille d'eau écologique garde les boissons froides pendant 24 heures et chaudes pendant 12 heures. Fabriquée en acier inoxydable recyclé.

**German:**
Unsere umweltfreundliche Wasserflasche hält Getränke 24 Stunden kalt und 12 Stunden heiß. Hergestellt aus recyceltem Edelstahl.

**Japanese:**
私たちのエコフレンドリーなウォーターボトルは、飲み物を24時間冷たく、12時間温かく保ちます。リサイクルされたステンレス鋼で作られています。
```

**Settings Used**: `temperature=0.15`, `max_tokens=512`

---

## Fine-Tuning with LoRA/PEFT

Ministral 14B can be fine-tuned efficiently using Parameter-Efficient Fine-Tuning (PEFT) methods like LoRA.

### Why Fine-Tune?

- Specialize for domain-specific tasks (medical, legal, finance)
- Improve performance on custom datasets
- Adapt to specific writing styles or formats
- Add knowledge not in the base model

### LoRA Overview

**Low-Rank Adaptation (LoRA)** trains small adapter matrices instead of full model weights:
- Reduces trainable parameters by 99%
- Saves 50-70% VRAM during training
- Maintains near-full-finetuning accuracy
- Enables training on consumer GPUs

### Hardware Requirements for Fine-Tuning

**Text-Only Fine-Tuning**:
- Minimum: 1x A100 40GB (with LoRA + quantization)
- Recommended: 1x A100 80GB or H100 80GB

**Vision + Text Fine-Tuning**:
- Minimum: 1x A100 80GB (vision encoder increases memory load)
- Recommended: 1x H100 80GB or 2x A100 80GB

**QLoRA (Quantized LoRA)**:
- Enables fine-tuning on 24GB GPUs (RTX 4090, etc.)
- Uses 4-bit quantization to reduce memory

### Fine-Tuning Example (Hugging Face PEFT)

**Installation**:
```bash
pip install transformers peft accelerate bitsandbytes torch datasets
```

**LoRA Configuration**:
```python
from peft import LoraConfig, get_peft_model
from transformers import AutoModelForCausalLM, AutoTokenizer, TrainingArguments, Trainer

model_id = "mistralai/Ministral-3-14B-Base-2512"

# Load model
model = AutoModelForCausalLM.from_pretrained(
    model_id,
    device_map="auto",
    torch_dtype="auto",
    load_in_8bit=True  # Use 8-bit quantization to save memory
)

tokenizer = AutoTokenizer.from_pretrained(model_id)

# LoRA config
lora_config = LoraConfig(
    r=16,  # Rank (higher = more parameters, better quality, more memory)
    lora_alpha=32,  # Scaling factor
    target_modules=["q_proj", "v_proj", "k_proj", "o_proj"],  # Which layers to adapt
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

# Apply LoRA
model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# Output: trainable params: 6,815,744 || all params: 13,894,815,744 || trainable%: 0.049%
```

**Prepare Dataset**:
```python
from datasets import load_dataset

# Load your custom dataset
dataset = load_dataset("json", data_files="training_data.jsonl")

# Format for instruction tuning
def format_instruction(example):
    prompt = f"<s>[INST] {example['instruction']} [/INST] {example['response']}</s>"
    return {"text": prompt}

formatted_dataset = dataset.map(format_instruction)
```

**Training Arguments**:
```python
training_args = TrainingArguments(
    output_dir="./ministral-14b-finetuned",
    per_device_train_batch_size=4,
    gradient_accumulation_steps=4,
    num_train_epochs=3,
    learning_rate=2e-4,
    fp16=True,
    save_steps=500,
    logging_steps=100,
    warmup_steps=100,
    save_total_limit=3,
    push_to_hub=True,  # Push to Hugging Face Hub
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=formatted_dataset["train"],
    tokenizer=tokenizer,
)

# Train
trainer.train()

# Save LoRA adapter
model.save_pretrained("./ministral-14b-lora-adapter")
```

### QLoRA for Memory-Constrained GPUs

```python
from transformers import BitsAndBytesConfig

# 4-bit quantization config
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype="float16",
    bnb_4bit_use_double_quant=True
)

model = AutoModelForCausalLM.from_pretrained(
    model_id,
    quantization_config=bnb_config,
    device_map="auto"
)
```

### Vision Fine-Tuning Considerations

For multimodal (vision + text) fine-tuning:
- Use smaller image resolutions to reduce VRAM (e.g., 224x224 or 512x512)
- Reduce batch size (1-2 per GPU)
- Use gradient checkpointing to save memory
- Maintain 1:1 aspect ratio for images

### Best Practices

1. **Start with Base Model**: Use `Ministral-3-14B-Base-2512` for fine-tuning, not Instruct
2. **Dataset Quality**: 1,000+ high-quality examples > 100,000 low-quality
3. **Evaluation**: Test on held-out validation set, not just training metrics
4. **Learning Rate**: Start low (1e-4 to 5e-4) to avoid catastrophic forgetting
5. **Save Checkpoints**: Regularly save to avoid losing progress
6. **Inference Format**: Match inference endpoint to training format (don't mix completion/chat templates)

### Deploying Fine-Tuned Model

**Load LoRA Adapter**:
```python
from peft import PeftModel

base_model = AutoModelForCausalLM.from_pretrained("mistralai/Ministral-3-14B-Base-2512")
model = PeftModel.from_pretrained(base_model, "./ministral-14b-lora-adapter")
model = model.merge_and_unload()  # Merge LoRA weights for faster inference

# Use as normal
```

**Push to Hugging Face Hub**:
```python
model.push_to_hub("your-username/ministral-14b-custom")
tokenizer.push_to_hub("your-username/ministral-14b-custom")
```

---

## Resources

### Official Documentation

- [Mistral AI - Introducing Mistral 3](https://mistral.ai/news/mistral-3)
- [Ministral 3 14B Official Docs](https://docs.mistral.ai/models/ministral-3-14b-25-12)
- [Mistral Prompting Guide](https://docs.mistral.ai/guides/prompting_capabilities)
- [Mistral Sampling Parameters](https://docs.mistral.ai/guides/sampling)

### Model Repositories

- [Hugging Face: Ministral-3-14B-Instruct-2512](https://huggingface.co/mistralai/Ministral-3-14B-Instruct-2512)
- [Hugging Face: Ministral-3-14B-Reasoning-2512](https://huggingface.co/mistralai/Ministral-3-14B-Reasoning-2512)
- [Hugging Face: Ministral-3-14B-Base-2512](https://huggingface.co/mistralai/Ministral-3-14B-Base-2512)

### Deployment Guides

- [vLLM: Ministral-3 Instruct Usage Guide](https://docs.vllm.ai/projects/recipes/en/latest/Mistral/Ministral-3-Instruct.html)
- [vLLM: Ministral-3 Reasoning Usage Guide](https://docs.vllm.ai/projects/recipes/en/latest/Mistral/Ministral-3-Reasoning.html)
- [vLLM: OpenAI-Compatible Server](https://docs.vllm.ai/en/latest/serving/openai_compatible_server/)
- [Unsloth: Ministral 3 How to Run Guide](https://docs.unsloth.ai/models/ministral-3)
- [Modal: Serverless Ministral 3 with vLLM](https://modal.com/docs/examples/ministral3_inference)

### API Platforms

- [Together AI: Ministral 3 14B Instruct](https://www.together.ai/models/ministral-3-14b-instruct-2512)
- [NVIDIA NIM: Ministral-14B-Instruct-2512](https://build.nvidia.com/mistralai/ministral-14b-instruct-2512/modelcard)

### Fine-Tuning Resources

- [DataCamp: Fine-Tuning Ministral 3 Guide](https://www.datacamp.com/tutorial/fine-tuning-ministral-3)
- [Hugging Face PEFT Documentation](https://huggingface.co/docs/transformers/en/peft)
- [NVIDIA: Parameter-Efficient Fine-Tuning](https://docs.nvidia.com/nim/large-language-models/latest/peft.html)

### Benchmarks & Analysis

- [Benchable: Ministral 3 14B Benchmarks](https://benchable.ai/models/mistralai/ministral-14b-2512)
- [LLM Stats: Ministral vs Phi-4 Comparison](https://llm-stats.com/models/compare/ministral-14b-latest-vs-phi-4-reasoning)
- [NVIDIA Blog: Mistral 3 Models Overview](https://developer.nvidia.com/blog/nvidia-accelerated-mistral-3-open-models-deliver-efficiency-accuracy-at-any-scale/)

### Community & Support

- [Mistral AI Discord](https://discord.com/invite/mistralai)
- [Hugging Face Discussions](https://huggingface.co/mistralai/Ministral-3-14B-Instruct-2512/discussions)
- [Ollama Library: Ministral-3](https://ollama.com/library/ministral-3:14b)

### Prompting Techniques (General)

- [Prompt Engineering Guide](https://www.promptingguide.ai/)
- [Chain-of-Thought Prompting Guide](https://www.promptingguide.ai/techniques/cot)
- [Mistral System Prompt Best Practices](https://blog.promptlayer.com/mistral-system-prompt/)

---

## Quick Reference Card

### Model Selection

| Use Case | Recommended Variant |
|----------|---------------------|
| General chat/assistant | Instruct |
| Math, coding, STEM | Reasoning |
| Custom fine-tuning | Base |
| Creative writing | Instruct (temp 0.7-0.9) |
| JSON extraction | Instruct (temp 0.1) |
| Function calling/agents | Instruct |
| Multimodal (vision) | Instruct or Reasoning |

### Temperature Quick Guide

| Task Type | Temperature |
|-----------|-------------|
| Factual/Consistent | 0.1 - 0.15 |
| Code Generation | 0.1 - 0.2 |
| Analysis | 0.15 - 0.3 |
| Creative Writing | 0.7 - 0.9 |
| Reasoning (Reasoning variant) | 0.7 - 1.0 |

### Hardware Requirements

| Precision | VRAM | GPU Examples |
|-----------|------|--------------|
| BF16 | 32GB | RTX 5090, A100 40GB |
| FP8 | 24GB | RTX 4090, L40S |
| 4-bit GGUF | 16GB | RTX 4060 Ti 16GB |

### vLLM Serving Commands

**Instruct**:
```bash
vllm serve mistralai/Ministral-3-14B-Instruct-2512 \
  --tokenizer_mode mistral --config_format mistral --load_format mistral
```

**Reasoning**:
```bash
vllm serve mistralai/Ministral-3-14B-Reasoning-2512 \
  --tokenizer_mode mistral --config_format mistral --load_format mistral \
  --reasoning-parser mistral
```

### Chat Template

```
<s>[INST] {instruction} [/INST] {response}</s>[INST] {next instruction} [/INST]
```

---

**Document Version**: 1.0
**Last Updated**: January 18, 2025
**Model Version**: Ministral-3-14B-2512 (December 2024 release)
