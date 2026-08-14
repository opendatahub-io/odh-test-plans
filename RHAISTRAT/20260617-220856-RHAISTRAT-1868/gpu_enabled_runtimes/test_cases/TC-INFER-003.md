---
test_case_id: TC-INFER-003
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-21"
upgrade_phase: post
---
# TC-INFER-003: Invalid input returns error on GPU runtime

**Objective**: Verify that the `mlserver-cuda-runtime` returns
appropriate HTTP 400 error responses when receiving malformed or
invalid inference requests via the KServe V2 protocol, and that
the server remains stable after processing invalid inputs.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator
  12.9+
- InferenceService deployed with `mlserver-cuda-runtime`,
  `nvidia.com/gpu: 1` in resource limits, and `external_route=True`
  (TC-DEPLOY-002)

**Markers**:

```python
pytestmark = pytest.mark.usefixtures("valid_aws_config")

@pytest.mark.model_server_gpu
@skip_if_no_supported_accelerator_type
```

**Test Steps**:

1. Obtain the InferenceService URL via external route:

   ```python
   url = get_exposed_isvc_url(isvc)
   infer_url = f"{url}/v2/models/{model_name}/infer"
   ```

2. Send a request with wrong tensor name and assert HTTP 400.
   Uses `requests.post` directly (not `send_rest_request`, which
   may swallow errors):

   ```python
   import requests

   wrong_name_payload = {
       "inputs": [{
           "name": "WRONG_NAME",
           "shape": [1, 3, 224, 224],
           "datatype": "FP32",
           "data": [0.0] * 150528,
       }]
   }
   resp = requests.post(
       infer_url,
       json=wrong_name_payload,
   )
   assert resp.status_code == 400
   ```

3. Send a request with wrong shape and assert HTTP 400:

   ```python
   wrong_shape_payload = {
       "inputs": [{
           "name": "<correct_input_name>",
           "shape": [1, 1],
           "datatype": "FP32",
           "data": [1.0],
       }]
   }
   resp = requests.post(
       infer_url,
       json=wrong_shape_payload,
   )
   assert resp.status_code == 400
   ```

4. Send a request with wrong datatype and assert HTTP 400:

   ```python
   wrong_dtype_payload = {
       "inputs": [{
           "name": "<correct_input_name>",
           "shape": [1, 3, 224, 224],
           "datatype": "INVALID",
           "data": [],
       }]
   }
   resp = requests.post(
       infer_url,
       json=wrong_dtype_payload,
   )
   assert resp.status_code == 400
   ```

5. Send a request with missing required field (`inputs`) and
   assert HTTP 400:

   ```python
   missing_field_payload = {}
   resp = requests.post(
       infer_url,
       json=missing_field_payload,
   )
   assert resp.status_code == 400
   ```

6. After all invalid requests, send a valid request to confirm
   server stability:

   ```python
   valid_resp = send_rest_request(
       url=url,
       input_data=RESNET50_REST_INPUT_QUERY,
   )
   assert "outputs" in valid_resp
   ```

**Expected Results**:

- Wrong tensor name request returns HTTP `400`
- Wrong shape request returns HTTP `400`
- Wrong datatype request returns HTTP `400`
- Missing required field request returns HTTP `400`
- Each error response body contains a message describing the
  validation failure
- The server does not crash or become unresponsive after
  processing invalid requests; subsequent valid request returns
  HTTP `200` with correct output

**Notes**: Renamed from old TC-INFER-006. Uses `requests.post`
directly for 400 status assertions instead of `send_rest_request`
which may swallow errors.
