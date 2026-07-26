---
name: pioneer-synthetic-data
description: Generate synthetic NER, classification, or decoder training data using Pioneer's Felix engine, then use the dataset name to start a fine-tuning job.
---

# Pioneer Synthetic Data Generation

Generate labeled training examples using Pioneer's Felix synthesis engine.

## Authentication

Pioneer API keys start with `pio_sk_`. Both header formats are accepted:

```
X-API-Key: YOUR_API_KEY
```
```
Authorization: Bearer YOUR_API_KEY
```

## Start a Generation Job

Returns HTTP 202 immediately. Required fields: `task_type`, `dataset_name`, `num_examples`.

```http
POST https://api.pioneer.ai/generate
X-API-Key: YOUR_API_KEY
Content-Type: application/json

{
  "task_type": "ner",
  "dataset_name": "my-ner-dataset",
  "labels": ["person", "company", "product"],
  "num_examples": 100,
  "domain_description": "Tech industry news articles"
}
```

**`task_type` options:**
- `ner` — requires `labels` (list of entity type strings)
- `classification` — requires `labels`
- `decoder` — requires `domain_description`
- `custom` — free-form prompt-based; requires `prompt` (natural language description of the task)

**Other useful fields:**
- `domain_description` — guides the style and topic of generated text
- `classified_examples` — seed examples to steer generation
- `prompt` — natural language task description (for `custom` type)

## Poll Job Status

`POST /generate` returns `job_id` — use this to poll.

```http
GET https://api.pioneer.ai/generate/jobs/JOB_ID
X-API-Key: YOUR_API_KEY
```

| Status | Meaning |
|--------|---------|
| `queued` | Waiting to start |
| `generating` | In progress |
| `ready` | Complete — response includes `data` rows and `count` |
| `failed` | Check `error` field in response |

## Auto-Label Existing Text

Have unlabeled text and want Pioneer to annotate it instead of generating from scratch:

```http
POST https://api.pioneer.ai/generate/ner/label-existing
X-API-Key: YOUR_API_KEY
Content-Type: application/json

{
  "labels": ["person", "organization", "location"],
  "inputs": [
    "Apple CEO Tim Cook spoke in Cupertino.",
    "Google hired 500 engineers in London."
  ]
}
```

For classification, use the equivalent endpoint (same request/response shape):

```http
POST https://api.pioneer.ai/generate/classification/label-existing
X-API-Key: YOUR_API_KEY
Content-Type: application/json

{
  "labels": ["positive", "negative", "neutral"],
  "inputs": [
    "The product exceeded my expectations.",
    "Delivery was three days late."
  ]
}
```

Both endpoints accept 1–1000 strings per request.

## Use the Dataset for Training

Pass the `dataset_name` you chose directly to `POST /felix/training-jobs`:

```json
{
  "datasets": [{"name": "my-ner-dataset"}]
}
```

See the `pioneer-fine-tune` skill for the full training workflow.

## Error Codes

| Code | Meaning |
|------|---------|
| 401 | Invalid or missing API key |
| 402 | Insufficient credits |
| 404 | Resource not found |
| 422 | Validation error — check request body fields |
| 500 | Server error — safe to retry |
