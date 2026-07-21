---
test_case_id: TC-FALLBACK-002
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-FALLBACK-002: Pod Pending when no GPU nodes available

**Objective**: Verify that deploying an InferenceService
requesting GPU resources on a cluster with no available GPU
nodes results in the pod remaining in `Pending` state and the
inference endpoint being inaccessible.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA
- `mlserver-cuda-runtime` ServingRuntime template available
- ResNet-50 ONNX model available in S3-compatible storage
- Valid AWS/S3 credentials configured for model access
- **Special environment requirement**: Cluster has no GPU
  resources allocatable, or all GPU nodes are cordoned. Use
  the `gpu_count_on_cluster` fixture to verify this precondition.

**Test Steps**:

1. Verify the cluster has no allocatable GPU resources (or all
   GPU nodes are cordoned):

   ```python
   pytestmark = pytest.mark.usefixtures("valid_aws_config")

   assert gpu_count_on_cluster == 0, (
       "This test requires a cluster with no available GPU "
       "resources"
   )
   ```

2. Deploy an InferenceService requesting 1 GPU. Use `wait=False`
   and `wait_for_predictor_pods=False` to avoid blocking until
   Ready (which would timeout):

   ```python
   isvc = create_isvc(
       model_service_account=mlserver_model_service_account,
       runtime=ModelInferenceRuntime.MLSERVER_CUDA_RUNTIME,
       gpu_count=1,
       model_format="onnx",
       storage_uri="s3://models/resnet-50-onnx/",
       wait=False,
       wait_for_predictor_pods=False,
   )
   ```

3. Poll for the predictor pod and verify it is in `Pending`
   state:

   ```python
   pods = get_pods_by_isvc_label(client=client, isvc=isvc)
   assert pods, "Expected at least one predictor pod"
   assert pods[0].instance.status.phase == "Pending"
   ```

4. Check the InferenceService status conditions to confirm it
   is not Ready:

   ```python
   conditions = isvc.instance.status.conditions
   ready_condition = next(
       (c for c in conditions if c.type == "Ready"), None
   )
   assert ready_condition is None or (
       ready_condition.status != "True"
   )
   ```

5. Verify the inference endpoint URL is empty or None,
   confirming the model is not accessible:

   ```python
   assert not isvc.instance.status.url, (
       "Expected no URL while pod is Pending"
   )
   ```

**Expected Results**:

- Pod status is `Pending` (not `Running`)
- InferenceService `Ready` condition is `False` or absent
- `isvc.instance.status.url` is empty or `None` -- no
  inference endpoint is available
- No silent fallback to CPU occurs -- the pod does not start
  on a CPU-only node

**Notes**: This test absorbs the former TC-FALLBACK-003
(inference inaccessible while pending). Requires a no-GPU
cluster or cordoned GPU nodes -- document as a special
environment requirement. The `gpu_count_on_cluster` fixture
validates this precondition at test start.
