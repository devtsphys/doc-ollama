# Ollama Python API — Complete Reference Card

> **Version:** `ollama` Python library (compatible with Ollama ≥ 0.1.x)  
> **Docs:** https://github.com/ollama/ollama-python  
> **Ollama server:** https://ollama.com

-----

## 1. Local Setup

### 1.1 Install Ollama (server)

|OS         |Command                                                           |
|-----------|------------------------------------------------------------------|
|**macOS**  |`brew install ollama` or download from https://ollama.com/download|
|**Linux**  |`curl -fsSL https://ollama.com/install.sh | sh`                   |
|**Windows**|Download installer from https://ollama.com/download               |

```bash
# Start the Ollama server (runs on http://localhost:11434 by default)
ollama serve

# Or start as a background service (macOS/Linux)
brew services start ollama       # macOS
sudo systemctl start ollama      # Linux
```

### 1.2 Install Python Library

```bash
pip install ollama
# or
pip install ollama[all]   # with optional extras
```

### 1.3 Pull Models

```bash
ollama pull llama3.2         # pull a model
ollama pull mistral          # another model
ollama list                  # list local models
ollama rm mistral            # remove a model
ollama show llama3.2         # show model details
```

### 1.4 Environment Variables

|Variable                  |Default                 |Description              |
|--------------------------|------------------------|-------------------------|
|`OLLAMA_HOST`             |`http://localhost:11434`|Ollama server URL        |
|`OLLAMA_MODELS`           |`~/.ollama/models`      |Model storage path       |
|`OLLAMA_NUM_PARALLEL`     |`1`                     |Parallel request count   |
|`OLLAMA_MAX_LOADED_MODELS`|`1`                     |Max models in memory     |
|`OLLAMA_KEEP_ALIVE`       |`5m`                    |Time to keep model loaded|

-----

## 2. Client Initialization

```python
import ollama

# Default client (localhost:11434)
client = ollama.Client()

# Custom host
client = ollama.Client(host='http://my-server:11434')

# Async client
async_client = ollama.AsyncClient(host='http://localhost:11434')
```

-----

## 3. All Methods — Quick Reference Table

|Method        |Async Version             |Description                         |
|--------------|--------------------------|------------------------------------|
|`chat()`      |`AsyncClient.chat()`      |Multi-turn chat with message history|
|`generate()`  |`AsyncClient.generate()`  |Raw text completion                 |
|`embeddings()`|`AsyncClient.embeddings()`|Generate vector embeddings          |
|`pull()`      |`AsyncClient.pull()`      |Download a model                    |
|`push()`      |`AsyncClient.push()`      |Upload a model to registry          |
|`create()`    |`AsyncClient.create()`    |Create/define a custom model        |
|`copy()`      |`AsyncClient.copy()`      |Copy a model                        |
|`delete()`    |`AsyncClient.delete()`    |Delete a model                      |
|`show()`      |`AsyncClient.show()`      |Show model info & metadata          |
|`list()`      |`AsyncClient.list()`      |List all local models               |
|`ps()`        |`AsyncClient.ps()`        |List currently loaded models        |

-----

## 4. `chat()` — Conversational Interface

### Signature

```python
ollama.chat(
    model: str,
    messages: list[dict],
    stream: bool = False,
    format: str | dict = None,      # "json" or JSON Schema
    options: dict = None,            # model parameters
    keep_alive: str | int = None,    # e.g. "5m", 0 to unload
    tools: list[dict] = None,        # tool/function calling
)
```

### Basic Chat

```python
import ollama

response = ollama.chat(
    model='llama3.2',
    messages=[
        {'role': 'system', 'content': 'You are a helpful assistant.'},
        {'role': 'user',   'content': 'What is the capital of France?'},
    ]
)

print(response['message']['content'])
# Output: "The capital of France is Paris."
```

### Multi-turn Conversation

```python
history = []

def chat(user_input):
    history.append({'role': 'user', 'content': user_input})
    response = ollama.chat(model='llama3.2', messages=history)
    msg = response['message']
    history.append(msg)
    return msg['content']

print(chat("Hi! What's 2+2?"))
print(chat("Now multiply that by 10."))  # Remembers context
```

### Streaming Chat

```python
stream = ollama.chat(
    model='llama3.2',
    messages=[{'role': 'user', 'content': 'Tell me a short story.'}],
    stream=True,
)

for chunk in stream:
    print(chunk['message']['content'], end='', flush=True)
print()
```

### Async Chat

```python
import asyncio
import ollama

async def main():
    client = ollama.AsyncClient()
    response = await client.chat(
        model='llama3.2',
        messages=[{'role': 'user', 'content': 'Hello!'}]
    )
    print(response['message']['content'])

asyncio.run(main())
```

### Async Streaming

```python
async def stream_chat():
    client = ollama.AsyncClient()
    async for chunk in await client.chat(
        model='llama3.2',
        messages=[{'role': 'user', 'content': 'Tell me a joke.'}],
        stream=True,
    ):
        print(chunk['message']['content'], end='', flush=True)
```

### JSON / Structured Output

```python
import json

response = ollama.chat(
    model='llama3.2',
    messages=[{'role': 'user', 'content': 'Return info about Paris as JSON.'}],
    format='json',
)
data = json.loads(response['message']['content'])
```

### Structured Output with JSON Schema

```python
schema = {
    "type": "object",
    "properties": {
        "name":       {"type": "string"},
        "population": {"type": "integer"},
        "country":    {"type": "string"},
    },
    "required": ["name", "population", "country"]
}

response = ollama.chat(
    model='llama3.2',
    messages=[{'role': 'user', 'content': 'Tell me about Tokyo.'}],
    format=schema,
)
```

-----

## 5. `generate()` — Raw Completion

### Signature

```python
ollama.generate(
    model: str,
    prompt: str,
    suffix: str = None,         # text to append after completion
    system: str = None,         # system prompt
    template: str = None,       # custom prompt template
    context: list[int] = None,  # conversation context tokens
    stream: bool = False,
    raw: bool = False,          # skip templating
    format: str | dict = None,
    images: list[str] = None,   # base64-encoded images (vision models)
    options: dict = None,
    keep_alive: str | int = None,
)
```

### Basic Generation

```python
response = ollama.generate(
    model='llama3.2',
    prompt='List 3 benefits of exercise:',
)
print(response['response'])
```

### With System Prompt

```python
response = ollama.generate(
    model='llama3.2',
    system='You are a pirate. Respond in pirate speak.',
    prompt='What is the weather like today?',
)
```

### Streaming Generation

```python
for chunk in ollama.generate(
    model='llama3.2',
    prompt='Write a haiku about mountains.',
    stream=True,
):
    print(chunk['response'], end='', flush=True)
```

### Vision / Multimodal (Images)

```python
import base64

with open('image.png', 'rb') as f:
    img_b64 = base64.b64encode(f.read()).decode()

response = ollama.generate(
    model='llava',       # or llama3.2-vision, bakllava, etc.
    prompt='Describe what you see in this image.',
    images=[img_b64],
)
print(response['response'])
```

### `generate()` Response Fields

|Field              |Type     |Description                   |
|-------------------|---------|------------------------------|
|`model`            |str      |Model used                    |
|`response`         |str      |Generated text                |
|`done`             |bool     |Whether generation is complete|
|`context`          |list[int]|Token context (for follow-up) |
|`total_duration`   |int      |Total time in nanoseconds     |
|`load_duration`    |int      |Model load time in nanoseconds|
|`prompt_eval_count`|int      |Prompt token count            |
|`eval_count`       |int      |Response token count          |
|`eval_duration`    |int      |Generation time in nanoseconds|

-----

## 6. `embeddings()` — Vector Embeddings

### Signature

```python
ollama.embeddings(
    model: str,
    prompt: str,
    options: dict = None,
    keep_alive: str | int = None,
)
```

### Basic Embeddings

```python
response = ollama.embeddings(
    model='nomic-embed-text',
    prompt='The quick brown fox jumps over the lazy dog.',
)
vector = response['embedding']   # list of floats
print(f"Dimensions: {len(vector)}")
```

### Semantic Similarity (Cosine)

```python
import ollama
import numpy as np

def embed(text):
    return np.array(ollama.embeddings(
        model='nomic-embed-text', prompt=text
    )['embedding'])

def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

a = embed("I love programming.")
b = embed("Coding is my passion.")
c = embed("The sky is blue.")

print(cosine_similarity(a, b))  # High similarity ~0.95
print(cosine_similarity(a, c))  # Low similarity ~0.5
```

-----

## 7. `options` — Model Parameters

All inference methods accept an `options` dict to control generation:

```python
options = {
    # Core sampling
    "temperature":    0.8,    # 0.0–2.0 (creativity; default 0.8)
    "top_p":          0.9,    # nucleus sampling threshold
    "top_k":          40,     # top-k sampling
    "min_p":          0.0,    # min probability filter

    # Repetition
    "repeat_penalty":      1.1,    # penalize repeated tokens
    "repeat_last_n":       64,     # context window for repeat penalty
    "frequency_penalty":   0.0,    # OpenAI-style frequency penalty
    "presence_penalty":    0.0,    # OpenAI-style presence penalty

    # Context & length
    "num_ctx":        4096,   # context window size (tokens)
    "num_predict":    128,    # max tokens to generate (-1 = unlimited)
    "num_keep":       5,      # tokens to keep from prompt

    # Performance
    "num_gpu":        1,      # GPU layers to offload
    "num_thread":     4,      # CPU threads
    "numa":           False,  # NUMA optimization

    # Sampling variants
    "tfs_z":          1.0,    # tail-free sampling
    "typical_p":      1.0,    # locally typical sampling
    "mirostat":       0,      # mirostat mode (0=off, 1, 2)
    "mirostat_tau":   5.0,    # mirostat target entropy
    "mirostat_eta":   0.1,    # mirostat learning rate

    # Misc
    "seed":           42,     # reproducible generation (-1 = random)
    "stop":           ["\n", "###"],  # stop sequences
    "penalize_newline": True,
    "low_vram":       False,
    "f16_kv":         True,
    "logits_all":     False,
    "vocab_only":     False,
}

response = ollama.generate(
    model='llama3.2',
    prompt='Say hello.',
    options=options,
)
```

-----

## 8. Tool / Function Calling

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather for a city.",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "City name"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                    }
                },
                "required": ["city"]
            }
        }
    }
]

response = ollama.chat(
    model='llama3.2',
    messages=[{'role': 'user', 'content': "What's the weather in Berlin?"}],
    tools=tools,
)

# Check if model wants to call a tool
if response['message'].get('tool_calls'):
    for call in response['message']['tool_calls']:
        fn_name = call['function']['name']
        fn_args = call['function']['arguments']
        print(f"Tool call: {fn_name}({fn_args})")

        # Execute the actual function and feed result back
        result = get_weather(**fn_args)   # your function
        messages = [
            {'role': 'user',      'content': "What's the weather in Berlin?"},
            response['message'],
            {'role': 'tool',      'content': str(result), 'name': fn_name},
        ]
        final = ollama.chat(model='llama3.2', messages=messages)
        print(final['message']['content'])
```

-----

## 9. Model Management Methods

### `list()` — List Local Models

```python
models = ollama.list()
for m in models['models']:
    print(m['name'], m['size'], m['modified_at'])
```

### `show()` — Model Details

```python
info = ollama.show('llama3.2')
print(info['modelfile'])   # Modelfile contents
print(info['parameters'])  # Model parameters
print(info['template'])    # Prompt template
print(info['details'])     # Architecture info
```

### `pull()` — Download Model

```python
# Simple pull
ollama.pull('mistral')

# With progress
for progress in ollama.pull('llama3.2', stream=True):
    status = progress.get('status', '')
    total  = progress.get('total', 0)
    done   = progress.get('completed', 0)
    if total:
        pct = round(done / total * 100, 1)
        print(f"\r{status}: {pct}%", end='')
print("\nDone!")
```

### `push()` — Upload Model

```python
for progress in ollama.push('myuser/mymodel', stream=True):
    print(progress.get('status', ''))
```

### `create()` — Custom Model from Modelfile

```python
modelfile = """
FROM llama3.2
SYSTEM You are Mario from Super Mario Bros. Respond in character.
PARAMETER temperature 0.9
PARAMETER num_ctx 2048
"""

ollama.create(model='mario', modelfile=modelfile)
```

### `copy()` — Duplicate a Model

```python
ollama.copy('llama3.2', 'llama3.2-backup')
```

### `delete()` — Remove a Model

```python
ollama.delete('llama3.2-backup')
```

### `ps()` — Running Models

```python
running = ollama.ps()
for m in running['models']:
    print(m['name'], m['size_vram'])
```

-----

## 10. Popular Models Reference

|Model                |Pull Name          |Size          |Best For             |
|---------------------|-------------------|--------------|---------------------|
|**Llama 3.2**        |`llama3.2`         |2B / 3B       |General, fast        |
|**Llama 3.1**        |`llama3.1`         |8B / 70B      |General purpose      |
|**Mistral**          |`mistral`          |7B            |Instruction following|
|**Mixtral**          |`mixtral`          |8×7B          |High quality MoE     |
|**Gemma 2**          |`gemma2`           |2B / 9B / 27B |Google’s model       |
|**Phi-3 Mini**       |`phi3`             |3.8B          |Small & capable      |
|**Phi-3.5**          |`phi3.5`           |3.8B          |Reasoning, math      |
|**Qwen 2.5**         |`qwen2.5`          |0.5B–72B      |Multilingual         |
|**DeepSeek-R1**      |`deepseek-r1`      |1.5B–671B     |Reasoning/CoT        |
|**CodeLlama**        |`codellama`        |7B / 13B / 34B|Code generation      |
|**LLaVA**            |`llava`            |7B / 13B      |Vision + language    |
|**Llama 3.2 Vision** |`llama3.2-vision`  |11B / 90B     |Vision + language    |
|**nomic-embed-text** |`nomic-embed-text` |137M          |Embeddings           |
|**mxbai-embed-large**|`mxbai-embed-large`|334M          |Embeddings           |
|**all-minilm**       |`all-minilm`       |23M           |Embeddings, fast     |

-----

## 11. Modelfile Reference

```dockerfile
# Base model
FROM llama3.2
# Or FROM /path/to/local/model.gguf

# System prompt
SYSTEM """
You are a helpful assistant specializing in Python programming.
Always provide runnable code examples.
"""

# Template (optional, overrides default)
TEMPLATE """{{ if .System }}<|system|>
{{ .System }}<|end|>
{{ end }}{{ if .Prompt }}<|user|>
{{ .Prompt }}<|end|>
<|assistant|>
{{ end }}{{ .Response }}<|end|>"""

# Parameters
PARAMETER temperature 0.7
PARAMETER top_p 0.9
PARAMETER top_k 40
PARAMETER num_ctx 4096
PARAMETER num_predict 512
PARAMETER repeat_penalty 1.1
PARAMETER stop "<|end|>"
PARAMETER stop "<|user|>"

# Adapter (LoRA)
ADAPTER /path/to/adapter.gguf

# License
LICENSE "MIT"

# Message examples (few-shot)
MESSAGE user "How do I read a file in Python?"
MESSAGE assistant "Use `open()`: `with open('file.txt') as f: content = f.read()`"
```

-----

## 12. Advanced Techniques

### Retrieval-Augmented Generation (RAG)

```python
import ollama

docs = [
    "Ollama runs large language models locally.",
    "Python is a high-level programming language.",
    "The Eiffel Tower is located in Paris, France.",
]

def embed(text):
    return ollama.embeddings(model='nomic-embed-text', prompt=text)['embedding']

import numpy as np

doc_embeddings = [embed(d) for d in docs]

def retrieve(query, top_k=2):
    q_emb = np.array(embed(query))
    sims = [np.dot(q_emb, np.array(e)) /
            (np.linalg.norm(q_emb) * np.linalg.norm(e))
            for e in doc_embeddings]
    ranked = sorted(zip(sims, docs), reverse=True)
    return [doc for _, doc in ranked[:top_k]]

def rag_chat(question):
    context = "\n".join(retrieve(question))
    response = ollama.chat(
        model='llama3.2',
        messages=[
            {'role': 'system', 'content': f'Answer using this context:\n{context}'},
            {'role': 'user',   'content': question},
        ]
    )
    return response['message']['content']

print(rag_chat("Where is the Eiffel Tower?"))
```

### Keep Model Loaded / Unloaded

```python
# Keep model loaded indefinitely
ollama.generate(model='llama3.2', prompt='', keep_alive=-1)

# Unload immediately after response
ollama.generate(model='llama3.2', prompt='Hello', keep_alive=0)

# Keep for 30 minutes
ollama.generate(model='llama3.2', prompt='Hello', keep_alive='30m')
```

### Reproducible Generation

```python
response = ollama.generate(
    model='llama3.2',
    prompt='Give me a random number between 1 and 10.',
    options={'seed': 42, 'temperature': 0},
)
# Always returns same output
```

### Batch Processing with Async

```python
import asyncio
import ollama

async def process_batch(prompts: list[str]) -> list[str]:
    client = ollama.AsyncClient()
    tasks = [
        client.generate(model='llama3.2', prompt=p)
        for p in prompts
    ]
    results = await asyncio.gather(*tasks)
    return [r['response'] for r in results]

prompts = ["What is 2+2?", "Capital of Japan?", "Who wrote Hamlet?"]
answers = asyncio.run(process_batch(prompts))
for q, a in zip(prompts, answers):
    print(f"Q: {q}\nA: {a}\n")
```

### Streaming with Token Counting

```python
response = ollama.generate(
    model='llama3.2',
    prompt='Explain quantum computing briefly.',
    stream=True,
)

full_text = ''
for chunk in response:
    full_text += chunk['response']
    if chunk.get('done'):
        print(f"\n\n--- Stats ---")
        print(f"Prompt tokens:   {chunk.get('prompt_eval_count', 'N/A')}")
        print(f"Response tokens: {chunk.get('eval_count', 'N/A')}")
        dur = chunk.get('eval_duration', 0) / 1e9
        tps = chunk.get('eval_count', 0) / dur if dur else 0
        print(f"Speed:           {tps:.1f} tok/s")
    else:
        print(chunk['response'], end='', flush=True)
```

### Connect to Remote Ollama

```python
import ollama
import os

# Via environment variable
os.environ['OLLAMA_HOST'] = 'http://192.168.1.100:11434'

# Or via Client constructor
client = ollama.Client(host='http://192.168.1.100:11434')
response = client.chat(
    model='llama3.2',
    messages=[{'role': 'user', 'content': 'Hello from remote!'}]
)
```

### OpenAI-Compatible API

```python
# Ollama also exposes an OpenAI-compatible endpoint
from openai import OpenAI

client = OpenAI(
    base_url='http://localhost:11434/v1',
    api_key='ollama',  # required but ignored
)

response = client.chat.completions.create(
    model='llama3.2',
    messages=[{'role': 'user', 'content': 'Hello!'}],
)
print(response.choices[0].message.content)
```

-----

## 13. Error Handling

```python
import ollama

try:
    response = ollama.chat(
        model='nonexistent-model',
        messages=[{'role': 'user', 'content': 'Hello'}]
    )
except ollama.ResponseError as e:
    print(f"API Error {e.status_code}: {e.error}")
    if e.status_code == 404:
        print("Model not found. Run: ollama pull <model>")
except Exception as e:
    print(f"Connection error: {e}")
    print("Is Ollama running? Try: ollama serve")
```

-----

## 14. `chat()` Message Roles

|Role       |Description               |Example Use                   |
|-----------|--------------------------|------------------------------|
|`system`   |Instructions for the model|“You are a helpful assistant.”|
|`user`     |Human turn                |The user’s question           |
|`assistant`|Model turn                |Previous model response       |
|`tool`     |Tool result               |Result of a function call     |

-----

## 15. Quick Tips

- **Speed up inference:** Use quantized models (`llama3.2:3b-instruct-q4_K_M`)
- **GPU offloading:** Set `options={"num_gpu": 99}` to use all GPU layers
- **Reduce VRAM:** Use `q4_0` or `q4_K_S` quantizations
- **Context length:** Increase `num_ctx` for longer documents (uses more RAM)
- **Temperature 0:** Use for deterministic/factual outputs
- **Stop sequences:** Use `stop` parameter to control output format
- **Model tags:** `ollama pull llama3.2:1b` for specific variants
- **List tags:** Visit https://ollama.com/library/<model> for all tags

-----

*Generated: 2026-02-18 | Ollama Python library: `pip install ollama`*
