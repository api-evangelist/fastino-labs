---
name: pioneer-inference
description: Run NER, classification, or generation inference on a Pioneer model using the native endpoint or the OpenAI/Anthropic-compatible endpoints.
---

# Pioneer Inference

Run inference against a trained GLiNER or decoder model on Pioneer.

## Authentication

All requests require your Pioneer API key (starts with `pio_sk_`).
Both header formats are accepted:

```
X-API-Key: YOUR_API_KEY
```
```
Authorization: Bearer YOUR_API_KEY
```

## Native Inference Endpoint

Best for direct NER, classification, and structured extraction.

```http
POST https://api.pioneer.ai/inference
X-API-Key: YOUR_API_KEY
Content-Type: application/json

{
  "model_id": "YOUR_TRAINING_JOB_ID",
  "text": "Apple announced the MacBook Pro.",
  "schema": {
    "entities": [{"name": "organization"}, {"name": "product"}]
  }
}
```

**Recommended schema shape (unified):**

- Entity extraction: `{"entities": [{"name": "organization"}, {"name": "product"}]}`
- Classification: `{"classifications": [{"task": "category", "labels": ["spam", "ham"], "multi_label": false}]}`
- Structured extraction: `{"structures": {"Person": {"fields": [{"name": "name", "dtype": "str"}]}}}`
- Decoder generation: omit `schema`, send `messages` with `"task": "generate"`.

**Legacy request shape (deprecated):**

The flat `task` + `list[str]`/`{"categories": [...]}` form is still
accepted, but every legacy submission returns the unified payload along
with `Deprecation: true` and a `Sunset: <RFC 7231 date>` response
header. Migrate clients to the unified schema dict above before the
sunset date to avoid breaking changes.

## OpenAI-Compatible Endpoint

Drop-in replacement for the OpenAI SDK. Set `base_url` to `https://api.pioneer.ai/v1`.
Pass `schema` via `extra_body` for NER/classification tasks.

```http
POST https://api.pioneer.ai/v1/chat/completions
X-API-Key: YOUR_API_KEY
Content-Type: application/json

{
  "model": "YOUR_TRAINING_JOB_ID",
  "messages": [{"role": "user", "content": "Extract entities from: Apple launched the iPhone."}],
  "schema": {"entities": [{"name": "organization"}, {"name": "product"}]}
}
```

`task_type` and list-form `schema` are still accepted on the
OpenAI/Anthropic/Responses compat routes too, but trigger the same
`Deprecation` + `Sunset` response headers.

## OpenAI Responses API Endpoint

For agents using the OpenAI Responses API format (`/v1/responses`):

```http
POST https://api.pioneer.ai/v1/responses
X-API-Key: YOUR_API_KEY
Content-Type: application/json

{
  "model": "YOUR_TRAINING_JOB_ID",
  "input": "Extract entities from: Apple launched the iPhone."
}
```

Also available without the `/v1` prefix at `POST https://api.pioneer.ai/responses`.

## Anthropic-Compatible Endpoint

Drop-in replacement for the Anthropic SDK. Set `base_url` to `https://api.pioneer.ai/v1`.

```http
POST https://api.pioneer.ai/v1/messages
X-API-Key: YOUR_API_KEY
Content-Type: application/json

{
  "model": "YOUR_TRAINING_JOB_ID",
  "max_tokens": 1024,
  "messages": [{"role": "user", "content": "Extract entities from: Apple launched the iPhone."}],
  "schema": ["organization", "product"]
}
```

## List Available Models

```http
GET https://api.pioneer.ai/base-models
X-API-Key: YOUR_API_KEY
```

Filterable by `training`, `inference`, and `task_type`. Use a model ID from this
response — or a completed training job ID — as `model_id` / `model`.

## Error Codes

| Code | Meaning |
|------|---------|
| 401 | Invalid or missing API key |
| 402 | Insufficient credits |
| 404 | Resource not found |
| 422 | Validation error — check request body fields |
| 500 | Server error — safe to retry |
