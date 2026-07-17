---
test_case_id: TC-UPGRADE-003
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-UPGRADE-003: Rollback removes GPU template without CPU impact

**Objective**: Verify that removing the `mlserver-cuda-runtime`
ClusterServingRuntime (simulating a rollback) does not impact existing
CPU-based InferenceServices using `mlserver-onnx`.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-cuda-runtime` ClusterServingRuntime applied in
  `redhat-ods-applications` namespace
- CPU InferenceService `resnet-cpu` deployed and serving inference
- No active GPU InferenceServices (or they have been deleted)

**Test Steps**:
1. Record baseline CPU inference response:
   ```bash
   URL=$(oc get inferenceservice resnet-cpu \
     -o jsonpath='{.status.url}')
   BASELINE=$(curl -s -X POST \
     "${URL}/v2/models/resnet-cpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json)
   ```
2. Delete the GPU ClusterServingRuntime:
   ```bash
   oc delete clusterservingruntime mlserver-cuda-runtime
   ```
3. Verify the GPU runtime is removed:
   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime 2>&1
   ```
4. Wait 30 seconds for reconciliation.
5. Verify the CPU InferenceService is still `Ready`:
   ```bash
   oc get inferenceservice resnet-cpu \
     -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'
   ```
6. Send an inference request to the CPU runtime and compare:
   ```bash
   AFTER=$(curl -s -X POST \
     "${URL}/v2/models/resnet-cpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json)
   ```

**Expected Results**:
- GPU ClusterServingRuntime returns "not found" after deletion
- CPU InferenceService remains in `Ready` state
- CPU inference response matches the baseline prediction
- No errors in odh-model-controller logs related to the CPU runtime

**Notes**: To be filled later in the process.
