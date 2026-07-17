---
test_case_id: TC-INFER-001
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-INFER-001: GPU inference correctness via KServe V2 infer endpoint

**Objective**: Verify that the `mlserver-cuda-runtime` produces correct
inference results when sending a prediction request to the KServe V2
`/v2/models/{model}/infer` endpoint on port 8080.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator 12.9+
- InferenceService `resnet-gpu` deployed with `mlserver-cuda-runtime`
  and Ready (TC-DEPLOY-002)
- ResNet-50 ONNX model loaded from Red Hat-maintained S3 bucket with
  known expected outputs for a reference input image

**Test Steps**:
1. Obtain the InferenceService URL:
   ```bash
   URL=$(oc get inferenceservice resnet-gpu \
     -o jsonpath='{.status.url}')
   ```
2. Send an inference request with a known input:
   ```bash
   curl -s -X POST "${URL}/v2/models/resnet-gpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json
   ```
3. Parse the response and verify the predicted class matches the
   known expected output for the reference input.
4. Verify the response conforms to KServe V2 inference protocol
   structure.

**Expected Results**:
- Response HTTP status code is `200`
- Response body contains `"model_name": "resnet-gpu"` and an
  `"outputs"` array
- The top predicted class index matches the known expected output
  for the reference input (e.g., class 285 for a golden retriever
  image in ImageNet)
- Response includes `"id"` field matching the request ID

**Test Data**:
```json
{
  "id": "infer-001",
  "inputs": [
    {
      "name": "input",
      "shape": [1, 3, 224, 224],
      "datatype": "FP32",
      "data": "<flattened_pixel_values_for_golden_retriever_image>"
    }
  ]
}
```

**Expected Response**:
```json
{
  "id": "infer-001",
  "model_name": "resnet-gpu",
  "model_version": "1",
  "outputs": [
    {
      "name": "output",
      "shape": [1, 1000],
      "datatype": "FP32",
      "data": [0.0, 0.0, "...", 0.92, "..."]
    }
  ]
}
```

**Notes**: To be filled later in the process.
