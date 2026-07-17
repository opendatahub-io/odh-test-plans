---
test_case_id: TC-FALLBACK-002
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-FALLBACK-002: GPU runtime on node without NVIDIA GPU results in pod Pending

**Objective**: Verify that deploying the `mlserver-cuda-runtime` on a
cluster where no nodes have NVIDIA GPU resources results in the pod
remaining in Pending state.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA
- `mlserver-cuda-runtime` ClusterServingRuntime applied in
  `redhat-ods-applications` namespace
- HardwareProfile `nvidia-gpu-a100` (API group
  `infrastructure.opendatahub.io/v1`) created
- Test executed on a cluster or node pool where no nodes have
  `nvidia.com/gpu` allocatable resources (or all GPU nodes are
  cordoned)

**Test Steps**:
1. Confirm no nodes have allocatable GPU resources (or cordon GPU
   nodes):
   ```bash
   oc get nodes -o custom-columns=\
     "NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"
   ```
2. Deploy InferenceService with GPU runtime and HardwareProfile:
   ```bash
   cat <<EOF | oc apply -f -
   apiVersion: serving.kserve.io/v1beta1
   kind: InferenceService
   metadata:
     name: resnet-gpu-nogpu
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
3. Wait 60 seconds and check pod status:
   ```bash
   oc get pod \
     -l serving.kserve.io/inferenceservice=resnet-gpu-nogpu \
     -o jsonpath='{.items[0].status.phase}'
   ```
4. Inspect scheduling events:
   ```bash
   oc get events --field-selector \
     reason=FailedScheduling --sort-by='.lastTimestamp'
   ```

**Expected Results**:
- Pod status is `Pending`
- Events contain a `FailedScheduling` event referencing insufficient
  `nvidia.com/gpu` resources
- The pod does not start on a CPU-only node — no silent fallback
  occurs

**Notes**: To be filled later in the process.
