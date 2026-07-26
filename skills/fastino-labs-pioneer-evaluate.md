---
name: pioneer-evaluate
description: Evaluate a trained Pioneer model against a dataset to measure performance, then compare results against baseline LLM models.
---

# Pioneer Evaluate

Run an evaluation to measure how well a trained model performs on a labeled dataset.

## Authentication

Pioneer API keys start with `pio_sk_`. Both header formats are accepted:

```
X-API-Key: YOUR_API_KEY
```
```
Authorization: Bearer YOUR_API_KEY
```

## Recommended Order

1. Complete a training job (status `deployed`)
2. Confirm your dataset exists (`GET /felix/datasets/:name`)
3. Run an evaluation with the training job ID and dataset name
4. Poll until results are ready

## Run an Evaluation

`base_model` accepts a training job ID (unlike training, which requires a HuggingFace model ID).

```http
POST https://api.pioneer.ai/felix/evaluations
X-API-Key: YOUR_API_KEY
Content-Type: application/json

{
  "base_model": "YOUR_TRAINING_JOB_ID",
  "dataset_name": "YOUR_DATASET_NAME"
}
```

## Get Evaluation Results

```http
GET https://api.pioneer.ai/felix/evaluations/YOUR_EVALUATION_ID
X-API-Key: YOUR_API_KEY
```

## List All Evaluations

```http
GET https://api.pioneer.ai/felix/evaluations
X-API-Key: YOUR_API_KEY
```

Optional query filter: `project_id`.

## Compare Against Baseline Models

List the baseline LLMs available for comparison:

```http
GET https://api.pioneer.ai/felix/baseline-models
X-API-Key: YOUR_API_KEY
```

## Delete an Evaluation

```http
DELETE https://api.pioneer.ai/felix/evaluations/YOUR_EVALUATION_ID
X-API-Key: YOUR_API_KEY
```

## Next Step

After evaluating, run inference with the same training job ID as `model_id`.
See the `pioneer-inference` skill.

## Error Codes

| Code | Meaning |
|------|---------|
| 401 | Invalid or missing API key |
| 402 | Insufficient credits |
| 404 | Resource not found |
| 422 | Validation error — check request body fields |
| 500 | Server error — safe to retry |
