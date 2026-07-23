---
test_case_id: TC-INFER-002
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-21"
upgrade_phase: post
---
# TC-INFER-002: GPU vs CPU numerical consistency

**Objective**: Verify that GPU inference via `mlserver-cuda-runtime`
and CPU inference via `mlserver-runtime` produce consistent top-K
predicted class indices for the same ResNet-50 model and identical
inputs. Uses top-K class index comparison instead of element-wise
numerical comparison because CUDA uses FMA operations with
different rounding than CPU BLAS.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator
  12.9+
- GPU InferenceService deployed with `mlserver-cuda-runtime`,
  `gpu_count=1`, and `external_route=True` (TC-DEPLOY-002)
- CPU InferenceService deployed with `mlserver-runtime` and
  `external_route=True`, using the same ResNet-50 ONNX model from
  `mlserver/model_repository/resnet-50-onnx`

**Markers**:

```python
pytestmark = pytest.mark.usefixtures("valid_aws_config")

@pytest.mark.model_server_gpu
@skip_if_no_supported_accelerator_type
```

**Test Steps**:

```python
# Both ISVCs are provided by class-scoped fixtures:
#   gpu_isvc — mlserver_cuda_inference_service
#              (nvidia.com/gpu: 1, external_route=True, resnet-50 ONNX)
#   cpu_isvc — mlserver_inference_service
#              (external_route=True, same resnet-50 ONNX model)
```

1. Obtain both InferenceService URLs via external routes:

   ```python
   gpu_url = get_exposed_isvc_url(gpu_isvc)
   cpu_url = get_exposed_isvc_url(cpu_isvc)
   ```

2. Send the same inference request to the GPU runtime:

   ```python
   gpu_response = send_rest_request(
       url=gpu_url,
       input_data=RESNET50_REST_INPUT_QUERY,
   )
   ```

3. Send the identical request to the CPU runtime:

   ```python
   cpu_response = send_rest_request(
       url=cpu_url,
       input_data=RESNET50_REST_INPUT_QUERY,
   )
   ```

4. Compare the top-5 predicted class indices using
   `compare_top_k_predictions` (new utility to be added to
   `mlserver/utils.py`):

   ```python
   from mlserver.utils import compare_top_k_predictions

   compare_top_k_predictions(
       response_a=gpu_response,
       response_b=cpu_response,
       k=5,
   )
   ```

**Expected Results**:

- Both responses return valid KServe V2 output with `"outputs"`
  list
- Output tensor shapes are identical between GPU and CPU
- Top-5 predicted class indices match between GPU and CPU
  responses
- No element-wise numerical tolerance assertion (CUDA FMA
  rounding differs from CPU BLAS)

**Notes**: Uses `compare_top_k_predictions` utility (does not
exist yet, to be added to `mlserver/utils.py`). Both ISVCs use
the same ResNet-50 model from
`mlserver/model_repository/resnet-50-onnx`. No `1e-4` absolute
tolerance assertion.
