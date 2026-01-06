<!--
  SPDX-FileCopyrightText: Copyright (c) 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
  SPDX-License-Identifier: Apache-2.0
-->
- Start the VLM embedding microservice
- Configure ingestion to embed content as text or images using env vars
- Point the ingestor to the VLM embedding service and model

Requirements: An NVIDIA GPU and a valid `NGC_API_KEY`.

:::{note}
**Early Access**: Currently, `nvidia/llama-3.2-nemoretriever-1b-vlm-embed-v1` is in early access preview.
:::

:::{note}
**PDF Support Only**: The VLM embedding feature is currently only supported for PDF documents. Other document formats (Word, PowerPoint, etc.) are not supported with VLM embedding.
:::

## Limitations

- The VLM embedding feature is experimental and responses may not be accurate.
- Summary generation doesn't work when this feature is enabled.

## 1. Start the VLM Embedding NIM locally

We provide a dedicated compose profile that starts only the VLM embedding service so the text embedding service does not start.
You can skip this step if you are interested in using cloud hosted endpoints.

```bash
export USERID=$(id -u)
export NGC_API_KEY=<your_ngc_api_key>
# Optionally select a GPU for the VLM embed service
export VLM_EMBEDDING_MS_GPU_ID=<gpu_id_or_leave_default>

# Start only the VLM embedding microservice
docker compose -f deploy/compose/nims.yaml --profile vlm-embed up -d

# Verify the service is healthy
docker ps --filter "name=nemoretriever-vlm-embedding-ms" --format "table {{.Names}}\t{{.Status}}"
```

Service details (from `deploy/compose/nims.yaml`):
- Service name: `nemoretriever-vlm-embedding-ms`
- Default port mapping: `9081:8000` (internal NIM port `8000`)

export APP_EMBEDDINGS_MODELNAME="nvidia/llama-3.2-nemoretriever-1b-vlm-embed-v1"

# Launch or restart the ingestor server so the new env vars take effect
docker compose -f deploy/compose/docker-compose-ingestor-server.yaml up -d
```


You can control what gets embedded as text or as images using these env vars:
- `APP_NVINGEST_STRUCTURED_ELEMENTS_MODALITY`: set to `image` to embed extracted tables/charts as images (keep text as text)
- `APP_NVINGEST_IMAGE_ELEMENTS_MODALITY`: set to `image` to embed page images as images
- `APP_NVINGEST_EXTRACTPAGEASIMAGE`: set to `True` to treat each page as a single image (experimental)

Below are common configurations.


Extractor collects text, tables, and charts as textual content; embedder treats all content as text.

```bash
export APP_NVINGEST_EXTRACTTEXT="true"
export APP_NVINGEST_EXTRACTTABLES="true"
export APP_NVINGEST_EXTRACTCHARTS="true"
export APP_NVINGEST_EXTRACTIMAGES="false"
# Do not set structured/image modalities (or set them empty) so everything embeds as text
export APP_NVINGEST_STRUCTURED_ELEMENTS_MODALITY=""
export APP_NVINGEST_IMAGE_ELEMENTS_MODALITY=""
export APP_NVINGEST_EXTRACTPAGEASIMAGE="false"

# Apply by restarting ingestor-server
docker compose -f deploy/compose/docker-compose-ingestor-server.yaml up -d
```


Extractor collects text, tables, and charts; embedder treats standard text as text while embedding tables and charts as images via `APP_NVINGEST_STRUCTURED_ELEMENTS_MODALITY="image"`.

```bash
export APP_NVINGEST_EXTRACTTEXT="true"
export APP_NVINGEST_EXTRACTTABLES="true"
export APP_NVINGEST_EXTRACTCHARTS="true"
export APP_NVINGEST_EXTRACTIMAGES="false"
# Use the VLM model to capture spatial/structural info for tables and charts
export APP_NVINGEST_STRUCTURED_ELEMENTS_MODALITY="image"
export APP_NVINGEST_IMAGE_ELEMENTS_MODALITY=""
export APP_NVINGEST_EXTRACTPAGEASIMAGE="false"

docker compose -f deploy/compose/docker-compose-ingestor-server.yaml up -d
```

### Embed entire pages as images (experimental)

Extractor captures each page as a single image (`APP_NVINGEST_EXTRACTPAGEASIMAGE="True"`); embedder processes page images via `APP_NVINGEST_IMAGE_ELEMENTS_MODALITY="image"`. Other extraction types are disabled to avoid duplicating content.

:::{note}
Citations don't work in the `generate` and `search` APIs of the RAG server with this configuration.
:::

```bash
# Treat each page as a single image (turn off other extractors)
export APP_NVINGEST_EXTRACTTEXT="false"
export APP_NVINGEST_EXTRACTTABLES="false"
export APP_NVINGEST_EXTRACTCHARTS="false"
export APP_NVINGEST_EXTRACTIMAGES="false"
export APP_NVINGEST_EXTRACTPAGEASIMAGE="true"
# Ensure page images are embedded as images
export APP_NVINGEST_IMAGE_ELEMENTS_MODALITY="image"
export APP_NVINGEST_STRUCTURED_ELEMENTS_MODALITY=""

docker compose -f deploy/compose/docker-compose-ingestor-server.yaml up -d
```

- **Start only VLM embedding service**: `docker compose -f deploy/compose/nims.yaml --profile vlm-embed up -d`
- **Point ingestor to VLM embedding**:
  - `APP_EMBEDDINGS_SERVERURL=nemoretriever-vlm-embedding-ms:8000`
  - `APP_EMBEDDINGS_MODELNAME=nvidia/llama-3.2-nemoretriever-1b-vlm-embed-v1`
- **Modality env vars**:
  - `APP_NVINGEST_STRUCTURED_ELEMENTS_MODALITY`: `image` or empty
  - `APP_NVINGEST_IMAGE_ELEMENTS_MODALITY`: `image` or empty
  - `APP_NVINGEST_EXTRACTPAGEASIMAGE`: `True` or `False`

If you use a `.env` file, add the variables there instead of exporting them, then rerun the compose commands.

ingestor-server:
  envVars:
    APP_EMBEDDINGS_SERVERURL: "nemoretriever-vlm-embedding-ms:8000"
    APP_EMBEDDINGS_MODELNAME: "nvidia/llama-3.2-nemoretriever-1b-vlm-embed-v1"

nv-ingest:
  envVars:
    EMBEDDING_NIM_ENDPOINT: "http://nemoretriever-vlm-embedding-ms:8000/v1"
    EMBEDDING_NIM_MODEL_NAME: "nvidia/llama-3.2-nemoretriever-1b-vlm-embed-v1"
```

  --username '$oauthtoken' \
  --password "${NGC_API_KEY}" \
  --set imagePullSecret.password=$NGC_API_KEY \
  --set ngcApiSecret.password=$NGC_API_KEY \
  -f deploy/helm/nvidia-blueprint-rag/values.yaml
```

ingestor-server:
  envVars:
    # Extraction toggles
    APP_NVINGEST_EXTRACTTEXT: "true"
    APP_NVINGEST_EXTRACTTABLES: "true"
    APP_NVINGEST_EXTRACTCHARTS: "true"
    APP_NVINGEST_EXTRACTIMAGES: "false"
    APP_NVINGEST_EXTRACTPAGEASIMAGE: "false"
    # Embedding modality controls
    APP_NVINGEST_STRUCTURED_ELEMENTS_MODALITY: ""   # set to "image" to embed tables/charts as images
    APP_NVINGEST_IMAGE_ELEMENTS_MODALITY: ""        # set to "image" to embed page images as images
    # Ingestor-side embedding target
    APP_EMBEDDINGS_SERVERURL: "nemoretriever-vlm-embedding-ms:8000"
    APP_EMBEDDINGS_MODELNAME: "nvidia/llama-3.2-nemoretriever-1b-vlm-embed-v1"

nv-ingest:
  envVars:
    # NV-Ingest runtime embedding target
    EMBEDDING_NIM_ENDPOINT: "http://nemoretriever-vlm-embedding-ms:8000/v1"
    EMBEDDING_NIM_MODEL_NAME: "nvidia/llama-3.2-nemoretriever-1b-vlm-embed-v1"
