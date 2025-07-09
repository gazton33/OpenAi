# OpenAI Platform & GPT Creation Knowledge Base

*Last updated: 3 Jul 2025*

---

## 1 · API Quick‑Start Essentials

### 1.1 Authentication & SDK setup

- **API keys** are required for every request; store them in environment variables or a secret manager. fileciteturn0file10
- Official SDKs (`openai` for Python / `openai` for Node.js) handle retries, streaming, and beta headers automatically. fileciteturn0file0turn0file1

### 1.2 Basic text generation (Responses API)

```python
from openai import OpenAI
client = OpenAI()
resp = client.responses.create(
    model="gpt-4.1",
    input="Write a one‑sentence bedtime story about a unicorn."
)
print(resp.output_text)
```

Same request in JavaScript and cURL are shown in the docs. fileciteturn0file0turn0file8

### 1.3 Multimodal input (images)

```javascript
const response = await client.responses.create({
  model: "gpt-4.1",
  input: [
    { role: "user", content: "What two teams are playing?" },
    { role: "user", content: [{ type: "input_image", image_url: "https://…" }] }
  ]
});
```

Use `type:"input_image"` objects inside the `input` array. fileciteturn0file0

---

## 2 · Models & Selection

| Family                  | Strengths                          | Typical use           | Notes                                      |
| ----------------------- | ---------------------------------- | --------------------- | ------------------------------------------ |
| **o3 / o4‑mini**        | Long‑term planning, hard reasoning | Agents                | Higher latency fileciteturn0file1       |
| **GPT‑4.1**             | Balanced agentic execution         | General GPTs          | Default choice                             |
| **GPT‑4.1‑mini / nano** | Low latency, cheaper               | Mobile, near‑realtime | Less reasoning depth fileciteturn0file1 |

Pin your production workload to a **specific model snapshot** (e.g. `gpt‑4.1‑2025‑04‑14`) to guarantee deterministic behaviour across deploys. fileciteturn0file8

---

## 3 · Prompt Engineering Patterns

1. **Clear role separation** – use `developer`, `user`, `assistant` message roles or the top‑level `instructions` field to reflect the [chain‑of‑command] hierarchy. fileciteturn0file8
2. **Few‑shot examples** – provide labelled exemplars inside the prompt to guide style & format. fileciteturn0file9
3. **Structured Outputs** – request JSON by adding `response_format:{type:"json_object"}` or via function definitions. fileciteturn0file8
4. **Tool‑aware prompting** – phrase tasks so the model will pick the right tool when `tools:[…]` is enabled. fileciteturn0file6
5. **Context anchoring** – always include fresh or proprietary context (via file search, vector store, or web search) to ground the answer.

Example (pirate tone):

```javascript
await client.responses.create({
  model:"gpt-4.1",
  instructions:"Talk like a pirate.",
  input:"Are semicolons optional in JavaScript?"
})
```

fileciteturn0file8

---

## 4 · Conversation State & Context Windows

- **Stateless** calls: include prior turns manually inside `input`.
- **Threaded** calls: use `previous_response_id` to let the platform handle truncation and billing. fileciteturn0file5
- Track **token limits**: output tokens + input tokens ≤ model context window (e.g. 16 384 for `gpt‑4o‑2024‑08‑06`). fileciteturn0file5

---

## 5 · Tools & Function Calling

| Built‑in Tool            | Purpose                          | Minimal config                              |
| ------------------------ | -------------------------------- | ------------------------------------------- |
| **web\_search\_preview** | Fetch current web info           | `{type:"web_search_preview"}`               |
| **file\_search**         | Semantic search in uploaded docs | `{type:"file_search", vector_store_id:"…"}` |
| **code\_interpreter**    | Run Python in a sandbox          | `{type:"code_interpreter"}`                 |
| **mcp**                  | Use remote MCP servers           | see remote MCP example below                |

Custom **functions** are declared in the `tools` array; the model emits JSON arguments when it decides to call them. fileciteturn0file6

*Example – Web search tool:*

```python
response = client.responses.create(
  model="gpt-4.1",
  tools=[{"type":"web_search_preview"}],
  input="Positive tech news today?"
)
```

fileciteturn0file6

*Remote MCP call (DeepWiki):* see full cURL/JS/Python sample. fileciteturn0file6

---

## 6 · Agent Primitives

OpenAI provides composable primitives across **models, tools, knowledge/memory, audio, guardrails, orchestration**. fileciteturn0file1

Key takeaways:

- Start with *o3 / o4‑mini* for heavy‑reasoning agents, or *GPT‑4.1‑mini* for responsiveness.
- Add **function calling** and built‑in tools for real‑world actions.
- Store long‑term context in **vector stores** and retrieve via **file search**. fileciteturn0file1
- Guard with **Moderation** API and instruction hierarchy libraries.
- Orchestrate via the **Python or TypeScript Agents SDK**, using tracing & evals for observability.

---

## 7 · Assistants API (beta)

Workflow:

1. **Create** an Assistant with `instructions`, `model`, and tools.
2. **Create** a persistent Thread per user session.
3. **Add** Messages to the Thread.
4. **Run** the Assistant; inspect Run Steps. fileciteturn0file7

Example – personal math tutor:

```python
assistant = client.beta.assistants.create(
  name="Math Tutor",
  instructions="You are a personal math tutor. Write and run code to answer math questions.",
  tools=[{"type":"code_interpreter"}],
  model="gpt-4o"
)
```

fileciteturn0file7

> **Migration note:** Assistants API will be deprecated once feature‑parity with Responses API is reached (sunset H1 2026). fileciteturn0file7

---

## 8 · GPT Actions (Custom GPTs)

- Reside inside **Custom GPTs**, bridging natural‑language intents to external REST APIs via Function Calling. fileciteturn0file3
- Dev defines JSON schema + authentication; GPT handles parameter filling and call execution.
- Ideal for data retrieval (e.g. querying a warehouse) or operational tasks (e.g. filing a Jira ticket).

*Weather.gov example (lat‑long → forecast):* two chained API calls `/points` and `/forecast`. fileciteturn0file3

---

## 9 · Codex (Cloud Software‑Engineering Agent)

- Containerised agent that clones repos, runs test loops, and returns PR‑ready diffs. fileciteturn0file2
- Modes: **ask** (read‑only) and **code** (full env).
- Requires GitHub app installation with *clone* + *PR* permissions.
- Internet access configurable per run (`agent-network`).

---

## 10 · Data Controls & Retention

| Endpoint                                                                                                              | Abuse‑log retention | App state retention | Zero‑Data‑Retention eligible |
| --------------------------------------------------------------------------------------------------------------------- | ------------------- | ------------------- | ---------------------------- |
| `/v1/chat/completions`                                                                                                | 30 d                | none¹               | ✅                            |
| `/v1/responses`                                                                                                       | 30 d                | 30 d²               | ✅                            |
| *…see full table in docs…*                                                                                            |                     |                     |                              |
| ¹ Audio outputs stored 1 h; schemas stored when Structured Outputs on.  ² unless `store:false`. fileciteturn0file4 |                     |                     |                              |

---

## 11 · Model Optimization Flywheel

1. **Evals** – measure current performance. fileciteturn0file9
2. **Prompt engineering** – iterate with contextual examples and instructions.
3. **Fine‑tune** – SFT, Vision FT, DPO, RFT depending on use‑case. fileciteturn0file9
4. Loop: eval → tweak prompt / dataset → re‑eval.

*Why fine‑tune?*  more examples than context allows, cheaper inference, proprietary data privacy. fileciteturn0file9

---

## 12 · Production Best Practices

- **Environment separation** – staging vs prod projects; custom spend & rate limits. fileciteturn0file10
- **Horizontal vs vertical scaling**, caching, and load balancing considerations.
- **Latency levers**: smaller models, lower `max_tokens`, stop sequences, streaming, infrastructure locality, batching. fileciteturn0file10
- **API‑key hygiene** – never commit keys; rotate and monitor usage. fileciteturn0file10

---

## 13 · Quick Reference – Common Code Snippets

```python
# Function calling skeleton
client.responses.create(
  model="gpt-4.1",
  tools=[{
    "type":"function",
    "function":{
      "name":"get_weather",
      "parameters":{…}
    }
  }],
  input="What should I pack for DC this weekend?"
)
```

```javascript
// Streaming with web search
await client.responses.create({
  model:"gpt-4.1",
  stream:true,
  tools:[{type:"web_search_preview"}],
  input:"Latest LLM research this week"
});
```

```bash
# Zero‑data‑retention chat completion
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{"model":"gpt-4.1","store":false,"input":"Hello"}'
```

---

### How to Update This Knowledge Base

When OpenAI releases new docs:

1. Drop the raw MD files into the project.
2. Search for the relevant section header or code snippet cited above (e.g. *"web\_search\_preview"* example).
3. Replace / append the updated content, preserving the citation marker so cross‑refs remain intact.

---

> **End of document**

