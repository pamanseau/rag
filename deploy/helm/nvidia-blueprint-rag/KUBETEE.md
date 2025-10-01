# RAG Blueprint Deployment for KubeTEE AI


## Pre-requis

### Building Custom Containers

If you need to build custom containers for rag-server and ingestor-server (e.g., with code modifications), use the following commands from the `rag` directory root:

#### Set Environment Variables

```bash
export REGISTRY=<your-registry>
export TAG=<your-tag>
```

#### Build RAG-Server Container

```bash
docker buildx build --platform linux/amd64 \
  -f src/nvidia_rag/rag_server/Dockerfile \
  -t ${REGISTRY}/rag-server:${TAG} \
  --push \
  .
```

#### Build Ingestor-Server Container

```bash
docker buildx build --platform linux/amd64 \
  -f src/nvidia_rag/ingestor_server/Dockerfile \
  -t ${REGISTRY}/ingestor-server:${TAG} \
  --push \
  .
```

**Note:** The build context is the repository root (`.`), so ensure you run these commands from the `rag` directory. After building, update the image references in `values-staging.yaml` to use your custom images.

### KubeTEE AI Provided Services

The following are the default components included in this blueprint:

* NVIDIA NIM Microservices
   * Response Generation (Inference)
      * [NIM of nvidia/llama-3.3-nemotron-super-49b-v1](https://build.nvidia.com/nvidia/llama-3_3-nemotron-super-49b-v1_5)
    * Retriever Models
      * [NIM of nvidia/llama-3_2-nv-embedqa-1b-v2]( https://build.nvidia.com/nvidia/llama-3_2-nv-embedqa-1b-v2)
      * [NIM of nvidia/llama-3_2-nv-rerankqa-1b-v2](https://build.nvidia.com/nvidia/llama-3_2-nv-rerankqa-1b-v2)
      * [NeMo Retriever Page Elements NIM](https://build.nvidia.com/nvidia/nemoretriever-page-elements-v2)
      * [NeMo Retriever Table Structure NIM](https://build.nvidia.com/nvidia/nemoretriever-table-structure-v1)
      * [NeMo Retriever Graphic Elements NIM](https://build.nvidia.com/nvidia/nemoretriever-graphic-elements-v1)

    * [NeMo Retriever OCR NIM](https://build.nvidia.com/nvidia/nemoretriever-ocr)
    * [Llama 3.1 NemoGuard 8B Content Safety NIM](https://build.nvidia.com/nvidia/llama-3_1-nemoguard-8b-content-safety)
    * [Llama 3.1 NemoGuard 8B Topic Control NIM](https://build.nvidia.com/nvidia/llama-3_1-nemoguard-8b-topic-control)
    * [Llama-3.1 Nemotron-nano-vl-8b-v1 NIM](https://build.nvidia.com/nvidia/llama-3.1-nemotron-nano-vl-8b-v1)
    * [NeMo Retriever Parse NIM](https://build.nvidia.com/nvidia/nemoretriever-parse)
    * [llama-3.2-nemoretriever-1b-vlm-embed-v1](https://build.nvidia.com/nvidia/llama-3_2-nemoretriever-1b-vlm-embed-v1)


## Helm Deployment

Deploy from the repository
```sh
helm upgrade --install rag ./rag/deploy/helm/nvidia-blueprint-rag -n aiq -f ./rag/deploy/helm/nvidia-blueprint-rag/values-staging.yaml
```
