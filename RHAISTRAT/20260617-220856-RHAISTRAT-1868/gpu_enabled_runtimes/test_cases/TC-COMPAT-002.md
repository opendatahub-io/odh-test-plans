---
test_case_id: TC-COMPAT-002
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-COMPAT-002: CPU model readiness via KServe V2 ready endpoint

**Objective**: Verify that the KServe V2 `/v2/models/{model}/ready`
endpoint returns a ready status for a model loaded on the existing
CPU `mlserver-onnx` runtime.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- CPU InferenceService `resnet-cpu` deployed and Ready
  (TC-COMPAT-001)

**Test Steps**:
1. Obtain the InferenceService URL:
   ```bash
   URL=$(oc get inferenceservice resnet-cpu \
     -o jsonpath='{.status.url}')
   ```
2. Query the model readiness endpoint:
   ```bash
   curl -s -o /dev/null -w "%{http_code}" \
     "${URL}/v2/models/resnet-cpu/ready"
   ```

**Expected Results**:
- Response HTTP status code is `200`
- Response body indicates the model is ready

**Notes**: To be filled later in the process.
