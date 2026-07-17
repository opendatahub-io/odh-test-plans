---
test_case_id: TC-INFER-004
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-INFER-004: GPU server readiness via KServe V2 health endpoint

**Objective**: Verify that the KServe V2 `/v2/health/ready` endpoint
returns a healthy status for the `mlserver-cuda-runtime` server on
port 8080.

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

2. Query the server health endpoint:

   ```bash
   curl -s -o /dev/null -w "%{http_code}" \
     "${URL}/v2/health/ready"
   ```

3. Retrieve the response body:

   ```bash
   curl -s "${URL}/v2/health/ready"
   ```

**Expected Results**:

- Response HTTP status code is `200`
- Response body indicates the server is ready (e.g.,
  `{"ready": true}`)

**Notes**: To be filled later in the process.
