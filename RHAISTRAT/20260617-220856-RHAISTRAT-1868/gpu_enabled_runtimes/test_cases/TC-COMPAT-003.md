---
test_case_id: TC-COMPAT-003
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-COMPAT-003: CPU inference correctness via KServe V2 infer endpoint

**Objective**: Verify that the existing CPU `mlserver-onnx` runtime
produces correct inference results for the ResNet-50 ONNX model,
serving as the baseline for GPU numerical consistency comparison
(TC-INFER-002).

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- CPU InferenceService `resnet-cpu` deployed and Ready
  (TC-COMPAT-001)
- Same ONNX model and reference input used in TC-INFER-001

**Test Steps**:
1. Obtain the InferenceService URL:
   ```bash
   URL=$(oc get inferenceservice resnet-cpu \
     -o jsonpath='{.status.url}')
   ```
2. Send the reference inference request:
   ```bash
   curl -s -X POST "${URL}/v2/models/resnet-cpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json
   ```
3. Verify the predicted class matches the known expected output.

**Expected Results**:
- Response HTTP status code is `200`
- Response body contains `"model_name": "resnet-cpu"` and an
  `"outputs"` array
- Top-1 predicted class index matches the known expected output
  (e.g., class 285 for a golden retriever image in ImageNet)
- Output tensor shape is `[1, 1000]`

**Notes**: To be filled later in the process.
