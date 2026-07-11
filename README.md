# TokenLab n8n Templates

Importable n8n workflows for TokenLab's public model catalog, OpenAI-compatible chat API, and asynchronous media tasks.

These templates are intentionally credential-free. Before running them, make `TOKENLAB_API_KEY` available to the n8n process or replace the `Authorization` header expressions with an n8n credential. Public model discovery does not require a key; chat and media generation do.

## Templates

| Workflow | Use case | Trigger |
| --- | --- | --- |
| `tokenlab-model-cost-router.json` | Read the live model catalog, choose a model by budget tier, then call chat | Webhook `POST /webhook/tokenlab-model-router` |
| `tokenlab-async-media-polling.json` | Start an async video task and poll until it reaches a terminal status | Webhook `POST /webhook/tokenlab-async-media` |
| `tokenlab-openai-compatible-chat.json` | Forward an existing OpenAI-style chat request to TokenLab | Webhook `POST /webhook/tokenlab-chat` |

## Import

1. In n8n, choose **Workflows -> Import from File**.
2. Import one JSON file from `workflows/`.
3. Set `TOKENLAB_API_KEY` in the n8n runtime, or replace the header expression with a credential.
4. Activate the workflow and send the example payload shown below.

### Model and cost router

```json
{
  "prompt": "Summarize this incident in three bullet points.",
  "budget_tier": "balanced"
}
```

The workflow reads `https://api.tokenlab.sh/v1/models` first. It prefers a live model whose ID matches the requested tier and falls back to `gpt-5.5` only when the catalog does not include a matching preferred model.

### Async video polling

```json
{
  "prompt": "A slow camera move over a mountain lake at sunrise",
  "model": "veo3.1",
  "duration": 5,
  "aspect_ratio": "16:9"
}
```

Video creation is asynchronous. The workflow follows the returned `poll_url` when present, otherwise it uses `/v1/tasks/{id}`. It checks `status` and stops only at `succeeded`, `completed`, `failed`, `cancelled`, or `expired`; it does not treat an optional progress field as completion.

### OpenAI-compatible chat

```json
{
  "model": "gpt-5.5",
  "messages": [
    { "role": "user", "content": "Give me a concise release checklist." }
  ]
}
```

## Public contract

- API base: `https://api.tokenlab.sh`
- Models: `GET /v1/models`
- Chat: `POST /v1/chat/completions`
- Video tasks: `POST /v1/videos/generations` and `GET /v1/tasks/{id}`
- Full docs: https://docs.tokenlab.sh
- Canonical site: https://tokenlab.sh

The templates are examples, not a replacement for the current API contract. Check the live docs before adding new fields or media operations.
