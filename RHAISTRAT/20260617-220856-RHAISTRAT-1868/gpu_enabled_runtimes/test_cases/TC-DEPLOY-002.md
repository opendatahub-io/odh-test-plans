---
test_case_id: TC-DEPLOY-002
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-21"
upgrade_phase: post
---
# TC-DEPLOY-002: Deploy GPU InferenceService and verify pod initialization

**Objective**: Verify that deploying an InferenceService using
`create_isvc` with `gpu_count=1` results in a running pod
scheduled on an NVIDIA GPU node with correct resource allocation,
CUDA execution provider initialization, and GPU device
accessibility.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator
  12.9+
- `mlserver-cuda-runtime` ServingRuntime applied
  (TC-DEPLOY-001)
- At least one worker node with NVIDIA GPU hardware
- ResNet-50 ONNX model available in S3-compatible storage
- Must run as `namespace-admin` (not cluster-admin)

**Markers**:

```python
pytestmark = pytest.mark.usefixtures("valid_aws_config")

@pytest.mark.model_server_gpu
@skip_if_no_supported_accelerator_type
```

**Test Steps**:

1. Create an InferenceService with GPU resources. Note that
   `create_isvc` automatically sets `nvidia.com/gpu` in resources
   and attaches volumes (`shared-memory`/`/dev/shm`,
   `tmp`/`/tmp`, `home`/`/home/mlserver`):

   ```python
   isvc = create_isvc(
       runtime=ModelInferenceRuntime.MLSERVER_CUDA_RUNTIME,
       gpu_count=1,
       external_route=True,
   )
   # The fixture handles waiting for Ready
   ```

2. Retrieve the predictor pod and verify it is running on a GPU
   node:

   ```python
   pods = get_pods_by_isvc_label(client, isvc)
   pod = pods[0]
   node_name = pod.instance.spec.nodeName
   assert node_name is not None
   ```

3. Verify the pod has `nvidia.com/gpu` resource limit of `"1"`:

   ```python
   resources = pod.instance.spec.containers[0].resources
   gpu_limit = resources.limits["nvidia.com/gpu"]
   assert gpu_limit == "1"
   ```

4. Verify `CUDAExecutionProvider` appears in container logs and
   is listed first (before `CPUExecutionProvider` fallback).
   Absorbed from old TC-DEPLOY-003:

   ```python
   logs = pod.log(container=Containers.KSERVE_CONTAINER_NAME)
   cuda_pos = logs.index("CUDAExecutionProvider")
   cpu_pos = logs.index("CPUExecutionProvider")
   assert cuda_pos < cpu_pos
   ```

5. Execute a command inside the container to confirm GPU device
   is accessible. Absorbed from old TC-INFER-005:

   ```python
   result = pod.execute(
       command=[
           "python3", "-c",
           "import onnxruntime; print(onnxruntime.get_device())"
       ]
   )
   assert result.strip() == "GPU"
   ```

**Expected Results**:

- InferenceService reaches `Ready` condition (handled by
  `create_isvc` fixture)
- Predictor pod is `Running` on a node with GPU capacity
- Pod resource limits include `nvidia.com/gpu: "1"`
- Container logs show `CUDAExecutionProvider` appearing before
  `CPUExecutionProvider`
- `onnxruntime.get_device()` inside the container returns `"GPU"`

**Notes**: Absorbs old TC-DEPLOY-003 (pod log/device checks) and
old TC-INFER-005 (CUDA EP check). HardwareProfile verification
removed; GPU resources injected directly via `create_isvc`
`gpu_count` parameter.
