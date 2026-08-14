---
test_case_id: TC-FALLBACK-001
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-FALLBACK-001: Silent CPU fallback detection

**Objective**: Verify that deploying the `mlserver-cuda-runtime`
without requesting GPU resources results in a silent CPU fallback
-- the pod schedules and runs on a CPU node, CUDA initialization
fails, and inference falls back to the CPU execution provider.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator
- `mlserver-cuda-runtime` ServingRuntime template available
- ResNet-50 ONNX model available in S3-compatible storage
- Valid AWS/S3 credentials configured for model access
- Cluster has GPU nodes available (to confirm the pod schedules
  on a CPU node rather than pending for GPU resources)

**Markers**:

```python
pytestmark = [
    pytest.mark.usefixtures("valid_aws_config"),
    pytest.mark.model_server_gpu,
]
```

Skip condition: `skip_if_no_supported_accelerator_type`

**Test Steps**:

1. Deploy an InferenceService using the GPU runtime but without
   requesting any GPU resources (`gpu_count=0`). The pod uses
   the GPU runtime image but schedules on a CPU node:

   ```python
   isvc = create_isvc(
       model_service_account=mlserver_model_service_account,
       runtime=ModelInferenceRuntime.MLSERVER_CUDA_RUNTIME,
       gpu_count=0,
       model_format="onnx",
       storage_uri="s3://models/resnet-50-onnx/",
       external_route=True,
   )
   ```

2. Verify the pod is in `Running` state (not `Pending` -- this
   is a silent fallback, not a scheduling failure):

   ```python
   pods = get_pods_by_isvc_label(client=client, isvc=isvc)
   assert pods, "Expected at least one predictor pod"
   assert pods[0].instance.status.phase == "Running"
   ```

3. Check the container logs for CUDA initialization failure
   messages indicating fallback to CPU:

   ```python
   logs = pods[0].log(
       container=Containers.KSERVE_CONTAINER_NAME,
   )
   assert "Failed to create CUDAExecutionProvider" in logs
   ```

4. Execute a command inside the container to confirm the
   ORT device is CPU:

   ```python
   result = pods[0].execute(
       command=[
           "python3", "-c",
           "import onnxruntime; print(onnxruntime.get_device())",
       ],
   )
   assert "CPU" in result
   ```

5. Confirm inference still works despite running on CPU
   silently:

   ```python
   url = get_exposed_isvc_url(isvc=isvc)
   response = send_rest_request(url=url, input_data=input_data)
   validate_deterministic_snapshot(response=response)
   ```

**Expected Results**:

- Pod status is `Running` (not `Pending`) -- the pod schedules
  successfully on a CPU node
- Container logs contain a CUDA initialization failure message
  (e.g., `"Failed to create CUDAExecutionProvider"`)
- `onnxruntime.get_device()` returns `"CPU"` inside the
  container
- Inference response returns HTTP `200` with a valid KServe V2
  output structure (verified by `validate_deterministic_snapshot`)
- The fallback to CPU is silent -- no pod crash, no restart

**Notes**: This test detects the silent fallback behavior where
the GPU runtime image runs on a CPU node without GPU resources.
The pod starts successfully but CUDA is unavailable, so
ONNX Runtime falls back to the CPU execution provider.
