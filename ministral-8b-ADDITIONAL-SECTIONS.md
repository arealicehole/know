# Additional Sections for Ministral 8B Guide

## (Continue from Chain-of-Thought section)

```
Assistant: Let me solve this step by step:
1. First, calculate total cookies: 12 batches × 24 cookies = 288 cookies
2. Then, calculate cookies sold: 288 × 3/4 = 216 cookies
3. Finally, calculate remaining: 288 - 216 = 72 cookies

Answer: 72 cookies remain.
```

#### Few-Shot CoT

Provide one or more examples with step-by-step reasoning.

**Example:**
```
Problem: If 5 apples cost $3, how much do 12 apples cost?
Solution:
Step 1: Find cost per apple = $3 / 5 = $0.60
Step 2: Multiply by 12 = $0.60 × 12 = $7.20
Answer: $7.20

Problem: If 8 notebooks cost $12, how much do 20 notebooks cost?
Solution: [Model completes with similar structure]
```

**Note:** Few-shot CoT generally outperforms zero-shot CoT. Adding demonstrations can increase accuracy by up to 28.2% in some tasks.

### Role-Playing and Persona Prompts

Ministral 8B responds well to persona-based prompts due to its strong system prompt adherence.

**Example:**
```
System: You are an expert Python developer with 10 years of experience in data science.
Focus on writing clean, well-documented code following PEP 8 standards.

User: Write a function to calculate the mean, median, and mode of a list of numbers.
```

### Task-Specific Prompting

#### Coding Tasks

**Best practices:**
- Specify the programming language
- Include requirements (libraries, constraints)
- Request comments/documentation
- Use temperature 0.1 for production code

**Example:**
```
Write a Python function that:
1. Takes a list of dictionaries as input
2. Filters items where 'status' == 'active'
3. Sorts by 'created_date' descending
4. Returns the top 10 results

Include type hints and docstrings.
```

#### Analysis Tasks

**Best practices:**
- Provide clear context
- Specify output format (bullet points, summary, etc.)
- Use temperature 0.1-0.3 for factual analysis

**Example:**
```
Analyze the following customer feedback and:
1. Identify the top 3 themes
2. Categorize sentiment (positive/neutral/negative)
3. Suggest action items

Feedback: [Insert text]
```

#### Creative Writing

**Best practices:**
- Use higher temperature (0.5-0.8)
- Provide genre, tone, and style guidance
- Specify length constraints

**Example:**
```
Write a short science fiction story (300 words) about an AI that discovers emotions.
Tone: Thoughtful and optimistic
Style: Lyrical prose
```

---

## Known Limitations & Workarounds

### Limitations

1. **Context Window in vLLM**
   - Currently capped at 32k context size on vLLM
   - Full 128k/256k requires Mistral Inference
   - Workaround: Use Mistral Inference for long-context tasks

2. **Large Context Memory Requirements**
   - For >100k token contexts, need 80GB GPU memory
   - Workaround: Use vLLM for memory efficiency, or chunk long documents

3. **Quantization Quality Trade-offs**
   - Below Q4 quantization shows quality degradation
   - Workaround: Use Q5 or Q6 for quality-critical applications

4. **Vision Image Size Constraints**
   - Works best with 1:1 aspect ratio images
   - Performance degrades with very wide/tall images
   - Workaround: Crop or resize images to near-square before processing

5. **Tool Overload**
   - Performance degrades with too many tools (>10-15)
   - Workaround: Group related functions, use tool hierarchies

### Comparison to Other 8B Models

**vs. Llama 3.1 8B:**
- Ministral 8B better for: Function calling, edge deployment, multimodal tasks, structured output
- Llama 3.1 8B better for: Raw throughput speed, pure text generation

**vs. Gemma 2 2B:**
- Ministral 8B significantly outperforms across all benchmarks
- Better knowledge, reasoning, and multilingual capabilities

**vs. Llama 3.2 3B:**
- Ministral 8B offers superior performance in all categories
- Better suited for complex tasks requiring deeper reasoning

### Working Around Common Issues

**Issue: Inconsistent JSON output**
- Solution: Use explicit JSON schema in system prompt
- Example:
  ```
  System: Always respond with valid JSON following this exact schema:
  {
    "answer": "string",
    "confidence": "number between 0-1",
    "sources": ["array of strings"]
  }
  ```

**Issue: Model not following system prompt**
- Solution: Place critical instructions at both beginning and end
- Repeat key constraints in user prompt

**Issue: Hallucinations in knowledge tasks**
- Solution: Use lower temperature (0.1), request citations, use chain-of-thought

---

## Practical Examples

### Example 1: Function Calling with Weather API

```python
from vllm import LLM
from vllm.sampling_params import SamplingParams

model = LLM(model="mistralai/Ministral-3-8B-Instruct-2512",
            tokenizer_mode="mistral",
            config_format="mistral",
            load_format="mistral")

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string"},
                    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
                },
                "required": ["location"]
            }
        }
    }
]

messages = [
    {"role": "system", "content": "You are a helpful weather assistant."},
    {"role": "user", "content": "What's the weather in Paris?"}
]

params = SamplingParams(temperature=0.1, max_tokens=512)
output = model.chat(messages, sampling_params=params, tools=tools)
print(output[0].outputs[0].text)
```

### Example 2: Code Generation with Documentation

**Prompt:**
```
System: You are an expert Python developer. Write clean, well-documented code.

User: Create a class that manages a simple in-memory cache with:
- get(key): retrieve value
- set(key, value, ttl): store with time-to-live
- delete(key): remove item
- clear(): remove all items

Include type hints, docstrings, and handle expired items.
```

**Temperature:** 0.1

### Example 3: Multimodal Document Analysis

```python
messages = [
    {"role": "system", "content": "You are a document analysis assistant. Extract key information accurately."},
    {
        "role": "user",
        "content": [
            {"type": "image", "image": invoice_image},
            {"type": "text", "text": "Extract: invoice number, date, total amount, and vendor name. Output as JSON."}
        ]
    }
]
```

### Example 4: Multilingual Translation with Context

**Prompt:**
```
System: You are a professional translator specializing in technical documentation.

User: Translate the following English text to Spanish, maintaining technical terminology:

"The API endpoint returns a JSON response containing the user's authentication token.
This token expires after 24 hours and must be included in the Authorization header."
```

**Temperature:** 0.1

### Example 5: Reasoning Task (Math Problem)

**Using Reasoning Variant with temperature 0.7:**
```
User: A train travels from City A to City B at 60 mph. On the return trip, it travels at 40 mph.
What is the average speed for the entire journey? Show your reasoning.
```

**Expected output:** Model provides step-by-step calculation, avoiding the common trap of averaging 60 and 40.

---

## API Usage & Deployment

### Deployment Platforms

Ministral 3 8B is available on:
- **Mistral AI Studio**
- **Amazon Bedrock**
- **Azure Foundry**
- **Hugging Face**
- **IBM WatsonX**
- **OpenRouter**
- **Fireworks**
- **Unsloth AI**
- **Together AI**
- **Modal**

Coming soon: NVIDIA NIM, AWS SageMaker

### Hugging Face with vLLM (Recommended)

**Installation:**
```bash
pip install vllm>=0.6.2
```

**Basic Usage:**
```python
from vllm import LLM
from vllm.sampling_params import SamplingParams

model_name = "mistralai/Ministral-3-8B-Instruct-2512"
llm = LLM(model=model_name,
          tokenizer_mode="mistral",
          config_format="mistral",
          load_format="mistral")

sampling_params = SamplingParams(
    temperature=0.1,
    top_p=0.1,
    max_tokens=2048
)

messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Explain quantum entanglement in simple terms."}
]

outputs = llm.chat(messages, sampling_params=sampling_params)
print(outputs[0].outputs[0].text)
```

**Serving via API:**
```bash
vllm serve mistralai/Ministral-3-8B-Instruct-2512 \
    --tokenizer_mode mistral \
    --config_format mistral \
    --load_format mistral
```

**Multi-GPU Support:**
```bash
vllm serve mistralai/Ministral-3-8B-Instruct-2512 \
    --tokenizer_mode mistral \
    --config_format mistral \
    --load_format mistral \
    --tensor_parallel 2
```

### Reasoning Variant Deployment

```bash
vllm serve mistralai/Ministral-3-8B-Reasoning-2512 \
    --tokenizer_mode mistral \
    --config_format mistral \
    --load_format mistral \
    --enable-auto-tool-choice \
    --tool-call-parser mistral \
    --reasoning-parser mistral
```

### Ollama Deployment

**Installation:**
```bash
ollama pull ministral-3:8b-instruct-2512-fp16
```

**Usage:**
```bash
ollama run ministral-3:8b-instruct-2512-fp16
```

**Features:**
- Built-in quantization (no manual Hugging Face cloning)
- Supports Modelfiles for custom fine-tuning
- Fast iteration, low latency
- Zero downtime local deployment

### Together AI API

**Example:**
```python
import together

together.api_key = "YOUR_API_KEY"

response = together.Complete.create(
    model="mistralai/Ministral-3-8B-Instruct-2512",
    prompt="Translate 'Hello, world!' to Japanese:",
    temperature=0.1,
    max_tokens=100
)

print(response['output']['choices'][0]['text'])
```

### Hardware Requirements by Deployment Type

| Deployment Type | VRAM Required | Hardware Example |
|----------------|---------------|------------------|
| Full Precision (BF16) | 24GB | RTX 3090, RTX 4090, A5000 |
| FP8 Precision | 12GB | RTX 4060 Ti 16GB, RTX 4070 |
| Q8 Quantization | 9GB | RTX 3060 12GB |
| Q6 Quantization | 7GB | RTX 3060 8GB |
| Q5 Quantization | 6GB | GTX 1660 Ti |
| Q4 Quantization | 5GB | GTX 1060 6GB |

**Recommended GPU for production:**
- Single GPU: NVIDIA A100 (40/80GB) or RTX 6000 Ada
- Consumer: RTX 4090 (24GB)
- Budget: RTX 3060 12GB with Q6/Q8 quantization

### Performance Benchmarks

**Consumer Hardware (RTX 4090):**
- Tokens/second: 50-60 (FP8)
- Latency (256 tokens): ~3-5 seconds
- Cold start: ~30 seconds

**Server Hardware (A100 80GB):**
- Tokens/second: 75-85 (BF16)
- Latency (256 tokens): ~2-3 seconds
- Supports larger batch sizes

---

## Quantization & Memory Requirements

### GGUF Quantization Levels

**Available quantizations for Ministral-3-8B-Instruct-2512:**

| Quantization | File Size | RAM/VRAM | Quality | Best For |
|--------------|-----------|----------|---------|----------|
| **BF16/FP16** | 17GB | 24GB | 100% | Maximum accuracy, server deployment |
| **Q8_0** | 9.03GB | 12GB | 98% | Near-original quality, production use |
| **Q6_K** | 6.59GB | 9GB | 95% | Balanced performance/quality |
| **Q5_K_M** | 6.06GB | 8GB | 93% | Good quality, moderate resources |
| **Q4_K_M** | 5.2GB | 7GB | 88% | Most common, good trade-off |
| **Q4_K_S** | 4.69GB | 6GB | 85% | Smaller footprint, acceptable quality |

**Reasoning Variant (Ministral-3-8B-Reasoning-2512):**

| Quantization | File Size |
|--------------|-----------|
| BF16 | 17GB |
| Q8_0 | 9.03GB |
| Q5_K_M | 6.06GB |
| Q4_K_M | 5.2GB |

### Quantization Method Comparison

**K-Quants vs I-Quants:**
- **K-quants:** More widely compatible, stable performance
- **I-quants:** Better performance-to-size ratio, newer technology
- **Recommendation:** Use I-quants for Nvidia/AMD GPUs if available; otherwise K-quants

### Choosing the Right Quantization

**For Maximum Quality (Research, Critical Applications):**
- Use Q8_0 or higher
- 12-16GB VRAM required
- ~98% of full precision quality

**For Production (Balanced Performance):**
- Use Q5_K_M or Q6_K
- 8-10GB VRAM required
- 93-95% quality retention

**For Resource-Constrained Environments:**
- Use Q4_K_M or Q4_K_S
- 6-7GB VRAM required
- 85-88% quality (acceptable for most tasks)

**Below Q4:** Not recommended - significant quality degradation

### Memory Optimization Tips

1. **Use vLLM for efficient memory management**
2. **Enable tensor parallelism** for multi-GPU setups
3. **Batch requests** when possible to amortize overhead
4. **Quantize to Q5/Q6** for production (minimal quality loss)
5. **Use FP8** instead of BF16 (50% memory savings, <1% quality loss)

---

## Best Practices Summary

### Quick Reference

**For Chat Applications:**
- Variant: Instruct
- Temperature: 0.1
- Context: Use strong system prompts
- Format: Standard chat message format

**For Code Generation:**
- Variant: Instruct
- Temperature: 0.1
- Technique: Specify language, requirements, documentation needs
- Output: Request type hints, comments, tests

**For Math/Reasoning:**
- Variant: Reasoning
- Temperature: 0.7
- Technique: Chain-of-thought, keep reasoning traces in context
- Format: Request step-by-step solutions

**For Creative Writing:**
- Variant: Instruct
- Temperature: 0.5-0.8
- Technique: Persona prompts, style/tone specification
- Context: Provide examples for desired format

**For Multimodal Tasks:**
- Variant: Instruct (with vision)
- Images: Near 1:1 aspect ratio
- Temperature: 0.1
- Format: Clear, specific questions about visual content

**For Function Calling:**
- Variant: Instruct
- Temperature: 0.1
- Tools: Limit to <10-15 well-defined functions
- Format: Follow Mistral's function calling schema

### Common Pitfalls to Avoid

1. Using Base variant for direct instruction tasks
2. Overloading model with too many tools/functions
3. Not keeping reasoning traces in context (Reasoning variant)
4. Using high temperature for factual/production tasks
5. Forgetting to specify output format
6. Weak or missing system prompts
7. Not leveraging few-shot examples for complex tasks
8. Using extreme aspect ratios for vision tasks

---

## Sources

### Official Documentation
- [Mistral AI Models Overview](https://mistral.ai/models)
- [Ministral 8B Documentation](https://docs.mistral.ai/models/ministral-8b-24-1)
- [Ministral 3 8B Documentation](https://docs.mistral.ai/models/ministral-3-8b-25-12)
- [Mistral Function Calling Guide](https://docs.mistral.ai/capabilities/function_calling)
- [Mistral Sampling Parameters Guide](https://docs.mistral.ai/guides/sampling)
- [Introducing Mistral 3 Announcement](https://mistral.ai/news/mistral-3)
- [Un Ministral, des Ministraux](https://mistral.ai/news/ministraux)

### Hugging Face Model Cards
- [ministralai/Ministral-8B-Instruct-2410](https://huggingface.co/mistralai/Ministral-8B-Instruct-2410)
- [ministralai/Ministral-3-8B-Instruct-2512](https://huggingface.co/mistralai/Ministral-3-8B-Instruct-2512)
- [ministralai/Ministral-3-8B-Base-2512](https://huggingface.co/mistralai/Ministral-3-8B-Base-2512)
- [ministralai/Ministral-3-8B-Reasoning-2512](https://huggingface.co/mistralai/Ministral-3-8B-Reasoning-2512)

### GGUF Quantizations
- [bartowski/Ministral-8B-Instruct-2410-GGUF](https://huggingface.co/bartowski/Ministral-8B-Instruct-2410-GGUF)
- [bartowski/ministralai_Ministral-3-8B-Instruct-2512-GGUF](https://huggingface.co/bartowski/mistralai_Ministral-3-8B-Instruct-2512-GGUF)
- [bartowski/ministralai_Ministral-3-8B-Reasoning-2512-GGUF](https://huggingface.co/bartowski/mistralai_Ministral-3-8B-Reasoning-2512-GGUF)

### Deployment & Technical Resources
- [NVIDIA: Mistral 3 Open Models](https://developer.nvidia.com/blog/nvidia-accelerated-mistral-3-open-models-deliver-efficiency-accuracy-at-any-scale/)
- [Ollama Ministral 8B](https://ollama.com/nchapman/ministral-8b-instruct-2410)
- [Ministral 3 8B on Ollama](https://ollama.com/library/ministral-3:8b-instruct-2512-fp16)
- [Together AI Ministral 3 8B API](https://www.together.ai/models/ministral-3-8b-instruct-2512)
- [Unsloth Ministral 3 Guide](https://docs.unsloth.ai/models/ministral-3)

### Comparative Analysis
- [Galaxy.ai: Ministral 3 14B vs 8B Comparison](https://blog.galaxy.ai/compare/ministral-14b-2512-vs-ministral-8b)
- [Artificial Analysis: Ministral 8B Performance](https://artificialanalysis.ai/models/ministral-8b)
- [DataCamp: Mistral 3 Overview](https://www.datacamp.com/blog/mistral-3)
- [TechHQ: Mistral Edge Models Analysis](https://techhq.com/2024/10/does-mistral-ai-have-the-worlds-best-edge-models/)

### Tutorials & Guides
- [Guide to Mistral System Prompts](https://blog.promptlayer.com/mistral-system-prompt/)
- [How to Run Ministral Using Ollama](https://medium.com/@monsuralirana/how-to-run-ministral-3b-and-8b-model-using-ollama-huggingface-gguf-and-langchain-a-step-by-step-1a750af6a00e)
- [Running Mistral 3 Locally](https://apidog.com/blog/run-mistral-3-locally/)

### General Prompting Techniques
- [Prompt Engineering Guide: Few-Shot Prompting](https://www.promptingguide.ai/techniques/fewshot)
- [Prompt Engineering Guide: Chain-of-Thought](https://www.promptingguide.ai/techniques/cot)
- [DataCamp: Few-Shot Prompting Tutorial](https://www.datacamp.com/tutorial/few-shot-prompting)
- [Codecademy: Zero-Shot vs Few-Shot Prompting](https://www.codecademy.com/article/prompt-engineering-101-understanding-zero-shot-one-shot-and-few-shot)

---

## Conclusion

Ministral 8B represents a significant advancement in edge AI, offering best-in-class performance for models under 10B parameters. With its multimodal vision capabilities, strong multilingual support, excellent code generation, and native function calling, it's an ideal choice for:

- Edge deployment scenarios
- Privacy-first applications
- Resource-constrained environments
- Agentic workflows
- Local AI assistants
- On-device applications

**Key Takeaways:**
1. Use the **Instruct variant** (Ministral-3-8B-Instruct-2512) for most use cases
2. Set **temperature to 0.1** for production/factual tasks
3. Leverage **strong system prompts** for consistent behavior
4. Use **Q5/Q6 quantization** for optimal quality-to-size ratio
5. Deploy with **vLLM** for production-ready inference
6. Keep **reasoning traces in context** for the Reasoning variant
7. Limit **tools to <15** for best function-calling performance

The model is fully open-source under Apache 2.0, making it an excellent choice for commercial applications requiring local deployment, data privacy, and cost-effective inference.

---

**Guide Version:** 1.0
**Last Updated:** 2025-01-18
**Model Version:** Ministral-3-8B-2512 series
**License:** Apache 2.0