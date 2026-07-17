---
test_case_id: TC-INFER-006
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-INFER-006: Inference with invalid input returns error on GPU runtime

**Objective**: Verify that the `mlserver-cuda-runtime` returns
appropriate error responses when receiving malformed or invalid
inference requests via the KServe V2 protocol on port 8080.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator 12.9+
- InferenceService `resnet-gpu` deployed with `mlserver-cuda-runtime`
  and Ready (TC-DEPLOY-002)

**Test Steps**:
1. Obtain the InferenceService URL:
   ```bash
   URL=$(oc get inferenceservice resnet-gpu \
     -o jsonpath='{.status.url}')
   ```
2. Send a request with mismatched input shape:
   ```bash
   curl -s -w "\n%{http_code}" -X POST \
     "${URL}/v2/models/resnet-gpu/infer" \
     -H "Content-Type: application/json" \
     -d '{"inputs":[{"name":"input","shape":[1,1],"datatype":"FP32","data":[1.0]}]}'
   ```
3. Send a request with invalid datatype:
   ```bash
   curl -s -w "\n%{http_code}" -X POST \
     "${URL}/v2/models/resnet-gpu/infer" \
     -H "Content-Type: application/json" \
     -d '{"inputs":[{"name":"input","shape":[1,3,224,224],"datatype":"INVALID","data":[]}]}'
   ```
4. Send a request with empty body:
   ```bash
   curl -s -w "\n%{http_code}" -X POST \
     "${URL}/v2/models/resnet-gpu/infer" \
     -H "Content-Type: application/json" -d '{}'
   ```
5. After all invalid requests, send a valid request to confirm server
   stability.

**Expected Results**:
- Each malformed request returns HTTP `400` (Bad Request)
- Response body contains an error message describing the validation
  failure (e.g., shape mismatch, invalid datatype)
- The server does not crash or become unresponsive after processing
  invalid requests — subsequent valid requests still return HTTP `200`

**Notes**: To be filled later in the process.
