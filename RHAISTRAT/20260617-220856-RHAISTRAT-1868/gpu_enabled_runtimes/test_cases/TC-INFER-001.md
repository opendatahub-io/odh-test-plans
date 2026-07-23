---
test_case_id: TC-INFER-001
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-21"
upgrade_phase: post
---
# TC-INFER-001: GPU inference correctness via KServe V2 infer endpoint

**Objective**: Verify model readiness, server health, and correct
inference results when sending a prediction request through the
external route to the KServe V2 `/v2/models/{model}/infer`
endpoint on the `mlserver-cuda-runtime`.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator
  12.9+
- InferenceService deployed with `mlserver-cuda-runtime`,
  `nvidia.com/gpu: 1` in resource limits, and `external_route=True`
  (TC-DEPLOY-002)
- ResNet-50 ONNX model loaded from S3

**Markers**:

```python
pytestmark = pytest.mark.usefixtures("valid_aws_config")

@pytest.mark.model_server_gpu
@skip_if_no_supported_accelerator_type
```

**Test Steps**:

1. Verify model readiness via the V2 model ready endpoint.
   Absorbed from old TC-INFER-003:

   ```python
   from mlserver.probes.utils import exec_http_probe

   http_get = {
       "path": "/v2/models/{model}/ready",
       "port": 8080,
   }
   result = exec_http_probe(pod, http_get)
   assert result is True
   ```

2. Verify server health via the V2 health ready endpoint.
   Absorbed from old TC-INFER-004:

   ```python
   http_get = {
       "path": "/v2/health/ready",
       "port": 8080,
   }
   result = exec_http_probe(pod, http_get)
   assert result is True
   ```

3. Send an inference request using the external route URL and
   `RESNET50_REST_INPUT_QUERY` constant. Input tensor name must
   match the ONNX model's actual input node. Shape is
   `[1, 3, 224, 224]`:

   ```python
   url = get_exposed_isvc_url(isvc)
   response = send_rest_request(
       url=url,
       input_data=RESNET50_REST_INPUT_QUERY,
   )
   ```

4. Validate response structure using `validate_deterministic_snapshot`.
   Do NOT assert specific class indices (e.g., class 285):

   ```python
   validate_deterministic_snapshot(response)
   ```

**Expected Results**:

- Model readiness probe returns success at
  `/v2/models/{model}/ready`
- Server health probe returns success at `/v2/health/ready`
- Response is a non-empty dict with an `"outputs"` list
- `outputs[0]["data"]` is a non-empty list of int/float values
- `validate_deterministic_snapshot` passes for response structure

**Test Data**:

Use `RESNET50_REST_INPUT_QUERY` constant (from
`mlserver/constant.py`). The input tensor name must match the
ONNX model's actual input node. For ResNet-50 ONNX models the
input is typically named `data` or `input.1` with shape
`[1, 3, 224, 224]` (NCHW, ImageNet-normalized).

**Notes**: Absorbs old TC-INFER-003 (model readiness check) and
old TC-INFER-004 (server health check). No specific class index
assertion; structure-only validation via
`validate_deterministic_snapshot`.
