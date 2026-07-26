---
name: pioneer-api
description: Interact with the Pioneer API to manage datasets, training jobs, evaluations, and run inference. Use when the user wants to call the Pioneer API, manage ML models, start training runs, run inference, or integrate Pioneer into their workflow.
---

# Pioneer API

Base URL: https://api.pioneer.ai
Auth header: X-API-Key: YOUR_API_KEY

Get your API key: pioneer.ai → Settings → API Keys.

## Inference (Pioneer format)

POST /inference — run predictions against a fine-tuned or base model
GET  /base-models — list available models (supports ?supports_inference=true&task_type=decoder filters)

```bash
curl -X POST https://api.pioneer.ai/inference \
 -H "X-API-Key: YOUR_API_KEY" \
 -H "Content-Type: application/json" \
 -d '{
 "model_id": "job_abc123",
 "text": "Apple announced the MacBook Pro at WWDC in Cupertino.",
 "schema": {
 "entities": ["organization", "product", "event", "location"]
 },
 "threshold": 0.5
 }'
```

The schema is a dict with optional keys:
- entities        — list of entity type strings (NER)
- classifications — list of {task, labels} objects (text classification)
- structures      — dict of structure definitions (JSON extraction)
- relations       — list of relation definitions

For decoder models, use "task": "generate" with a "messages" array instead of "text" and "schema": 
{                                                                                                                     
    "model_id": "job_abc123",                                                                                           
    "task": "generate",                                                                                                 
    "messages": [{"role": "user", "content": "Summarize this: ..."}]                                                    
  }

model_id is the job_id returned from POST /felix/training-jobs,
or a base model ID like "fastino/gliner2-base-v1".

## Inference (OpenAI-compatible)

POST /v1/chat/completions — OpenAI-compatible chat completions
POST /v1/completions      — OpenAI-compatible text completions

```bash
curl -X POST https://api.pioneer.ai/v1/chat/completions \
 -H "X-API-Key: YOUR_API_KEY" \
 -H "Content-Type: application/json" \
 -d '{
 "model": "job_abc123",
 "messages": [{"role": "user", "content": "Extract entities from: Apple launched the iPhone."}],
 "schema": {"entities": ["organization", "product"]}
 }'
```

## Inference (Anthropic-compatible)

POST /v1/messages — Anthropic-compatible messages

```bash
curl -X POST https://api.pioneer.ai/v1/messages \
 -H "X-API-Key: YOUR_API_KEY" \
 -H "Content-Type: application/json" \
 -d '{
 "model": "job_abc123",
 "max_tokens": 1024,
 "messages": [{"role": "user", "content": "Extract entities from: Apple launched the iPhone."}],
 "schema": {"entities": ["organization", "product"]}
 }'
```

## Inference History

GET  /inferences              — list past inferences
GET  /inferences/:id          — get a specific inference result
POST /inferences/:id/feedback — submit feedback on a result

## Datasets

GET    /felix/datasets           — list all datasets
GET    /felix/datasets/:name     — get dataset details
DELETE /felix/datasets/:name     — delete a dataset

```bash
curl https://api.pioneer.ai/felix/datasets \
 -H "X-API-Key: YOUR_API_KEY"
```

## Training Jobs

POST   /felix/training-jobs          — start a training job
GET    /felix/training-jobs          — list training jobs
GET    /felix/training-jobs/:id      — get job status and metrics
POST   /felix/training-jobs/:id/stop — stop a running job

```bash
curl -X POST https://api.pioneer.ai/felix/training-jobs \
 -H "X-API-Key: YOUR_API_KEY" \
 -H "Content-Type: application/json" \
 -d '{
 "model_name": "my-ner-model",
 "base_model": "fastino/gliner2-base-v1",
 "datasets": [{"name": "my-dataset"}],
 "training_type": "lora",
 "nr_epochs": 5,
 "learning_rate": 5e-5
 }'
```

base_model is required. Valid values: a supported model ID returned by GET /base-models
(e.g. fastino/gliner2-base-v1, Qwen/Qwen3-8B, meta-llama/Llama-3.1-8B-Instruct)
or a checkpoint UUID from a previous training job.

Response: { "id": "uuid-of-training-job", "status": "requested" }

Job status values: requested | running | complete | failed | stopped
Metrics (on COMPLETED): { "f1": 0.94, "precision": 0.96, "recall": 0.92 }

## Evaluations

POST   /felix/evaluations      — run an evaluation
GET    /felix/evaluations      — list evaluations
GET    /felix/evaluations/:id  — get evaluation results

```bash
curl -X POST https://api.pioneer.ai/felix/evaluations \
 -H "X-API-Key: YOUR_API_KEY" \
 -H "Content-Type: application/json" \
 -d '{
 "base_model": "job_abc123",
 "dataset_name": "my-eval-dataset"
 }'
```

Results include: f1, precision, recall, per_entity breakdown

## Errors

401 — invalid or missing API key
402 — insufficient credits
404 — resource not found
422 — validation error (check request body)
500 — server error
