---
test_case_id: TC-INFER-003
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-INFER-003: GPU model readiness via KServe V2 ready endpoint

**Objective**: Verify that the KServe V2 `/v2/models/{model}/ready`
endpoint returns a ready status for a model loaded on the
`mlserver-cuda-runtime` GPU runtime on port 8080.

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

2. Query the model readiness endpoint:

   ```bash
   curl -s -o /dev/null -w "%{http_code}" \
     "${URL}/v2/models/resnet-gpu/ready"
   ```

3. Retrieve the response body:

   ```bash
   curl -s "${URL}/v2/models/resnet-gpu/ready"
   ```

**Expected Results**:

- Response HTTP status code is `200`
- Response body indicates the model is ready (e.g.,
  `{"ready": true}` or `{"name": "resnet-gpu", "ready": true}`)

**Notes**: To be filled later in the process.
