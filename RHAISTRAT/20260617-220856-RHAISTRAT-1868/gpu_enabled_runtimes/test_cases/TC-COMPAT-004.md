---
test_case_id: TC-COMPAT-004
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-COMPAT-004: CPU runtime unaffected after GPU template addition

**Objective**: Verify that adding the `mlserver-cuda-runtime` template
to the cluster does not affect the behavior or availability of
existing CPU-based InferenceServices using `mlserver-onnx`.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- CPU InferenceService `resnet-cpu` deployed, Ready, and serving
  inference (TC-COMPAT-001 passed)
- `mlserver-cuda-runtime` ClusterServingRuntime not yet applied (or
  test must record baseline before applying)

**Test Steps**:

1. Record baseline: send an inference request to the CPU runtime and
   save the response:

   ```bash
   URL=$(oc get inferenceservice resnet-cpu \
     -o jsonpath='{.status.url}')
   BASELINE=$(curl -s -X POST \
     "${URL}/v2/models/resnet-cpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json)
   ```

2. Apply the GPU ClusterServingRuntime template:

   ```bash
   oc apply -f mlserver-cuda-runtime-template.yaml
   ```

3. Wait 30 seconds for any reconciliation to complete.
4. Verify the CPU InferenceService is still `Ready`:

   ```bash
   oc get inferenceservice resnet-cpu \
     -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'
   ```

5. Send the same inference request and compare to the baseline:

   ```bash
   AFTER=$(curl -s -X POST \
     "${URL}/v2/models/resnet-cpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json)
   ```

6. Verify the CPU predictor pod was not restarted:

   ```bash
   oc get pod \
     -l serving.kserve.io/inferenceservice=resnet-cpu \
     -o jsonpath='{.items[0].status.containerStatuses[0].restartCount}'
   ```

**Expected Results**:

- CPU InferenceService remains in `Ready` state after GPU template
  addition
- CPU inference response returns HTTP `200` with identical prediction
  class as the baseline
- CPU predictor pod restart count has not increased
- No errors or warnings in odh-model-controller logs related to the
  CPU runtime

**Notes**: To be filled later in the process.
