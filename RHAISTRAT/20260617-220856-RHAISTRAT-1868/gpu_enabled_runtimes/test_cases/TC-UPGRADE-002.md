---
test_case_id: TC-UPGRADE-002
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-UPGRADE-002: CPU InferenceServices unaffected by operator upgrade

**Objective**: Verify that existing CPU-based InferenceServices using
`mlserver-onnx` continue serving inference without interruption during
and after an operator upgrade to RHOAI 3.5 GA that introduces the
`mlserver-cuda-runtime`.

**Preconditions**:
- OCP 4.20+ cluster
- CPU InferenceService `resnet-cpu` deployed with `mlserver-onnx` and
  serving inference on the pre-upgrade version
- RHOAI operator ready to upgrade to 3.5 GA

**Test Steps**:
1. Record baseline: send an inference request and note the response:
   ```bash
   URL=$(oc get inferenceservice resnet-cpu \
     -o jsonpath='{.status.url}')
   BASELINE=$(curl -s -X POST \
     "${URL}/v2/models/resnet-cpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json)
   ```
2. Record the CPU predictor pod name and restart count:
   ```bash
   oc get pod -l serving.kserve.io/inferenceservice=resnet-cpu \
     -o jsonpath='{.items[0].metadata.name} restarts={.items[0].status.containerStatuses[0].restartCount}'
   ```
3. Initiate the operator upgrade to RHOAI 3.5 GA.
4. During the upgrade, periodically send inference requests (every
   30 seconds) and record success/failure:
   ```bash
   curl -s -o /dev/null -w "%{http_code}" \
     -X POST "${URL}/v2/models/resnet-cpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json
   ```
5. After the upgrade completes, verify the InferenceService is still
   `Ready` and inference results match the baseline.

**Expected Results**:
- CPU InferenceService remains `Ready` throughout and after the
  upgrade
- All periodic inference requests during upgrade return HTTP `200`
  (zero downtime)
- Post-upgrade inference response matches the baseline prediction
  class
- CPU predictor pod was not restarted by the upgrade process

**Notes**: To be filled later in the process.
