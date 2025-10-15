<!--
  SPDX-FileCopyrightText: Copyright (c) 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
  SPDX-License-Identifier: Apache-2.0
-->
   ```bash
   export OCR_GRPC_ENDPOINT=nemoretriever-ocr:8001
   export OCR_HTTP_ENDPOINT=http://nemoretriever-ocr:8000/v1/infer
   export OCR_INFER_PROTOCOL=grpc
   export OCR_MODEL_NAME=scene_text_ensemble
   ```

   ```bash
   USERID=$(id -u) docker compose -f deploy/compose/nims.yaml --profile nemoretriever-ocr up -d
   ```

   ```bash
   watch -n 2 'docker ps --format "table {{.Names}}\t{{.Status}}"'
   ```

   ```bash
   docker compose -f deploy/compose/docker-compose-ingestor-server.yaml up -d
   ```

   ```bash
   export OCR_HTTP_ENDPOINT=https://ai.api.nvidia.com/v1/cv/nvidia/nemoretriever-ocr
   export OCR_INFER_PROTOCOL=http
   export OCR_MODEL_NAME=scene_text_ensemble
   ```

  --username '$oauthtoken' \
  --password "${NGC_API_KEY}" \
  --set nv-ingest.paddleocr-nim.deployed=false \
  --set nv-ingest.nemoretriever-ocr.deployed=true \
  --set nv-ingest.envVars.OCR_MODEL_NAME="scene_text_ensemble" \
  --set imagePullSecret.password=$NGC_API_KEY \
  --set ngcApiSecret.password=$NGC_API_KEY
```


- **NeMo Retriever OCR**: Enhanced text extraction optimized for document processing
- **Paddle OCR**: Default OCR service for general text extraction

