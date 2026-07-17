---
test_case_id: TC-FALLBACK-001
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-FALLBACK-001: GPU runtime without GPU HardwareProfile results in pod Pending

**Objective**: Verify that deploying the `mlserver-cuda-runtime`
without associating a GPU HardwareProfile causes the pod to remain in
Pending state rather than silently falling back to CPU execution.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator 12.9+
- `mlserver-cuda-runtime` ClusterServingRuntime applied in
  `redhat-ods-applications` namespace (TC-DEPLOY-001)
- No GPU HardwareProfile associated with the InferenceService
- Cluster has GPU nodes available (to confirm the issue is missing
  HardwareProfile, not missing hardware)

**Test Steps**:
1. Create an InferenceService using the GPU runtime without a GPU
   HardwareProfile:
   ```bash
   cat <<EOF | oc apply -f -
   apiVersion: serving.kserve.io/v1beta1
   kind: InferenceService
   metadata:
     name: resnet-gpu-nohw
     annotations:
       serving.kserve.io/deploymentMode: RawDeployment
   spec:
     predictor:
       model:
         modelFormat:
           name: onnx
           version: "1"
         runtime: mlserver-cuda-runtime
         storageUri: s3://models/resnet-50-onnx/
   EOF
   ```
2. Wait 60 seconds and check the pod status:
   ```bash
   oc get pod \
     -l serving.kserve.io/inferenceservice=resnet-gpu-nohw \
     -o jsonpath='{.items[0].status.phase}'
   ```
3. Check the pod events for scheduling failure reasons:
   ```bash
   oc describe pod \
     -l serving.kserve.io/inferenceservice=resnet-gpu-nohw \
     | grep -A3 "Events:"
   ```
4. Verify no inference endpoint is reachable:
   ```bash
   oc get inferenceservice resnet-gpu-nohw \
     -o jsonpath='{.status.conditions[?(@.type=="Ready")].status}'
   ```

**Expected Results**:
- Pod status is `Pending` (not `Running`)
- Pod events indicate a scheduling failure related to insufficient
  GPU resources (e.g., "Insufficient nvidia.com/gpu")
- InferenceService `Ready` condition is `False`
- No inference response is served — the model is not accessible via
  any endpoint

**Notes**: To be filled later in the process.
