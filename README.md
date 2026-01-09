# Qdrant Llama Stack

This directory contains Kubernetes/OpenShift manifests for deploying a Llama Stack distribution configured to use Qdrant as the vector database backend.

## Files

### `qdrant-deployment.yaml`

This file contains four resources for deploying a Qdrant vector database:

1. **PersistentVolumeClaim (`qdrant-pvc`)**: Requests 10Gi of persistent storage for vector data
2. **Deployment (`qdrant`)**: Deploys a Qdrant container with:
   - Docker image: `qdrant/qdrant:v1.16` (official Qdrant image)
   - HTTP API on port 6333, gRPC on port 6334
   - Security context for OpenShift compatibility
   - API key authentication (from secret)
3. **Secret (`qdrant-secret`)**: Stores the Qdrant API key (default: `yourapikey` - should be changed)
4. **Service (`qdrant`)**: Exposes Qdrant on ports 6333 (HTTP) and 6334 (gRPC)

### `llama-stack-distribution-qdrant.yaml`

This file contains two resources:

1. **ConfigMap (`llama-stack-config`)**: Defines the Llama Stack runtime configuration including:
   - API endpoints (agents, datasetio, files, inference, safety, scoring, tool_runtime, vector_io)
   - Inference providers (vLLM, sentence-transformers)
   - Vector I/O provider (Qdrant)
   - Agents, scoring, and tool runtime providers
   - Model configurations for inference and embedding models
   - Server configuration (port 8321)

2. **LlamaStackDistribution Custom Resource (`llama-stack-distribution-qdrant`)**: Deploys the Llama Stack server with:
   - Resource requests/limits (250m CPU / 500Mi memory requests, 8 CPU / 12Gi memory limits)
   - Environment variables for inference model configuration (from secrets)
   - Qdrant connection settings pointing to the qdrant service
   - Installs qdrant-client library at startup
   - References to the ConfigMap for user configuration

## Deployment

```bash
# Apply the Qdrant database deployment first
oc apply -f qdrant-deployment.yaml

# Wait for the Qdrant pod to be ready
oc wait --for=condition=ready pod -l app=qdrant --timeout=300s

# Apply the Llama Stack distribution
oc apply -f llama-stack-distribution-qdrant.yaml
```

## Note

The Red Hat image of Llama Stack does not include the Qdrant client library by default. It is installed manually in this deployment.

## Configuration

### Qdrant Secret

Before deploying, update the `qdrant-secret` in `qdrant-deployment.yaml` with a secure API key (default: `yourapikey`).

### Inference Model Secret

The deployment requires the `llama-stack-inference-model-secret` with the following keys:
- `INFERENCE_MODEL`: The model identifier
- `VLLM_URL`: URL to your vLLM inference server
- `VLLM_TLS_VERIFY`: Whether to verify TLS (true/false)
- `VLLM_API_TOKEN`: API token for vLLM authentication
