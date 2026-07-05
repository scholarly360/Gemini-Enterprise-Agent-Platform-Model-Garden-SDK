# Deploy using Gemini Enterprise Agent Platform Model Garden SDK

## Prerequisites

- Python 3.x
- [Google Cloud CLI](https://docs.cloud.google.com/sdk/docs/install-sdk#windows)
- A GCP project with Gemini Enterprise Agent Platform (Vertex AI) API enabled
- A `.env` file in the repo root:
  ```
  GOOGLE_CLOUD_PROJECT_ID=your-project-id
  GOOGLE_CLOUD_REGION=us-central1
  ```

## Setup

### 1. Install Google Cloud CLI
[Install guide](https://docs.cloud.google.com/sdk/docs/install-sdk#windows)

### 2. Authenticate (Application Default Credentials)
```bash
gcloud auth application-default login
```
[ADC setup guide](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment)

### 3. Install Python dependencies
```bash
pip install google-cloud-aiplatform python-dotenv
```

## Usage

See `main.ipynb` for the full interactive walkthrough. Key steps:

### Initialize Vertex AI
```python
import os
import vertexai
from dotenv import load_dotenv
from vertexai import model_garden

load_dotenv()
PROJECT_ID = os.environ.get("GOOGLE_CLOUD_PROJECT_ID")
LOCATION = os.environ.get("GOOGLE_CLOUD_REGION", "us-central1")
vertexai.init(project=PROJECT_ID, location=LOCATION)
```

### List deployable models
```python
models = model_garden.list_deployable_models(model_filter="gemma", list_hf_models=True)
```

### Inspect deploy options
```python
model_id = "google/gemma3@gemma-3-1b-it"
gemma_model = model_garden.OpenModel(model_id)
deploy_options = gemma_model.list_deploy_options(concise=True)
print(deploy_options)
```

### Deploy a model
```python
gemma_endpoint = gemma_model.deploy(
    machine_type="g2-standard-4",
    accelerator_type="NVIDIA_L4",
    accelerator_count=1,
    min_replica_count=1,
    max_replica_count=1,
    endpoint_display_name="gemma_model_endpoint",
    model_display_name="gemma_model_self",
    deploy_request_timeout=3 * 60 * 60,
)
```

### Run a prediction
```python
prediction = gemma_endpoint.predict(
    instances=[{"prompt": "Tell me about Google Cloud", "temperature": 0.0001, "max_tokens": 150}]
)
print(prediction.predictions[0])
```

### Clean up
```python
gemma_endpoint.delete(force=True)
```
