---
name: pioneer-fine-tune
description: Fine-tune a GLiNER or decoder model on Pioneer from a labeled dataset, then monitor training until the model is ready for inference.
---

# Pioneer Fine-Tune

Fine-tune a custom model on Pioneer's GPU infrastructure.

## Authentication

Pioneer API keys start with `pio_sk_`. Both header formats are accepted:

```
X-API-Key: YOUR_API_KEY
```
```
Authorization: Bearer YOUR_API_KEY
```

## Recommended Order

1. Create or confirm your dataset exists (`GET /felix/datasets/:name`)
2. Start training with that exact dataset name
3. Poll until status is `deployed`
4. Use the `id` field from the training job response as `model_id` for inference

## Start a Training Job

`base_model` is required. Use a model ID from `GET /base-models` or a checkpoint UUID.
`datasets` is an array — supports multi-dataset training.

```http
POST https://api.pioneer.ai/felix/training-jobs
X-API-Key: YOUR_API_KEY
Content-Type: application/json

{
  "model_name": "my-ner-model",
  "base_model": "fastino/gliner2-base-v1",
  "datasets": [{"name": "YOUR_DATASET_NAME"}],
  "training_type": "lora",
  "nr_epochs": 5,
  "learning_rate": 5e-5
}
```

## Key Response Fields

`POST /felix/training-jobs` returns:
- `id` — the training job ID; this is your `model_id` for inference and `base_model` for evaluations
- `status` — current job status (see values below)

## Poll Training Status

```http
GET https://api.pioneer.ai/felix/training-jobs/YOUR_TRAINING_JOB_ID
X-API-Key: YOUR_API_KEY
```

| Status | Meaning |
|--------|---------|
| `requested` | Queued, waiting for GPU |
| `running` | GPU training in progress |
| `complete` | GPU training finished, post-processing starting |
| `normalizing` | Artifacts being normalized and deployed to inference provider |
| `deployed` | Ready for inference — use the job `id` as `model_id` |
| `errored` | Training failed — check logs |
| `stopped` | Manually stopped |
| `terminated` | Force-terminated |

Poll until `deployed` before running inference.

## Stream Training Logs

```http
GET https://api.pioneer.ai/felix/training-jobs/YOUR_TRAINING_JOB_ID/logs
X-API-Key: YOUR_API_KEY
```

## Other Useful Endpoints

```http
# List all jobs
GET https://api.pioneer.ai/felix/training-jobs

# List checkpoints
GET https://api.pioneer.ai/felix/training-jobs/YOUR_TRAINING_JOB_ID/checkpoints

# Download trained model
GET https://api.pioneer.ai/felix/training-jobs/YOUR_TRAINING_JOB_ID/download

# List all trained models
GET https://api.pioneer.ai/felix/trained-models
```

## Stopping vs Terminating a Job

These are two distinct operations with different consequences:

| Action | Endpoint | Effect | Reversible? |
|--------|----------|--------|-------------|
| Stop | `POST /felix/training-jobs/{id}/stop` | Graceful stop; all checkpoints preserved; status → `stopped` | Yes — checkpoints remain usable |
| Terminate | `POST /felix/training-jobs/{id}/terminate` | Force-stops the job **and deletes all checkpoints and artifacts**; status → `terminated` | **No — irreversible** |

```http
# Stop gracefully (checkpoints preserved)
POST https://api.pioneer.ai/felix/training-jobs/YOUR_TRAINING_JOB_ID/stop
X-API-Key: YOUR_API_KEY

# Terminate and delete all artifacts (irreversible)
POST https://api.pioneer.ai/felix/training-jobs/YOUR_TRAINING_JOB_ID/terminate
X-API-Key: YOUR_API_KEY
```

Prefer `stop` unless you explicitly want to destroy all artifacts.

## After Training

Once `deployed`, use the `id` from the response as `model_id` in `POST /inference`
or as `model` in `/v1/chat/completions`. See the `pioneer-inference` skill.

## Error Codes

| Code | Meaning |
|------|---------|
| 401 | Invalid or missing API key |
| 402 | Insufficient credits |
| 404 | Resource not found |
| 422 | Validation error — check request body fields |
| 500 | Server error — safe to retry |
