---
test_case_id: TC-DEPLOY-006
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-DEPLOY-006: Verify HardwareProfile-based GPU allocation

**Objective**: Verify that the `nvidia-gpu-a100` HardwareProfile (API
group `infrastructure.opendatahub.io/v1`) correctly injects
`nvidia.com/gpu` resource requests and GPU node tolerations into the
inference pod, without hardcoded resource limits in the
ServingRuntime template.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator 12.9+
- `mlserver-cuda-runtime` ClusterServingRuntime applied in
  `redhat-ods-applications` namespace (TC-DEPLOY-001)
- HardwareProfile `nvidia-gpu-a100` (API group
  `infrastructure.opendatahub.io/v1`) created with:
  - CPU: default 4, min 1, max 16
  - Memory: default 16Gi, min 4Gi, max 64Gi
  - `nvidia.com/gpu`: default 1, min 1, max 8
  - Toleration: `nvidia.com/gpu` operator Exists, effect NoSchedule
- GPU node available in the cluster

**Test Steps**:
1. Verify the `mlserver-cuda-runtime` ClusterServingRuntime does not
   contain hardcoded `nvidia.com/gpu` resource limits:
   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime -o yaml \
     | grep -A5 "resources"
   ```
2. Deploy an InferenceService with the GPU HardwareProfile:
   ```bash
   cat <<EOF | oc apply -f -
   apiVersion: serving.kserve.io/v1beta1
   kind: InferenceService
   metadata:
     name: resnet-gpu-hp
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
3. Verify the resulting pod has GPU resources injected by the
   HardwareProfile:
   ```bash
   oc get pod -l serving.kserve.io/inferenceservice=resnet-gpu-hp \
     -o jsonpath='{.items[0].spec.containers[?(@.name=="kserve-container")].resources}'
   ```
4. Confirm the pod is scheduled on a node with `nvidia.com/gpu`
   allocatable capacity:
   ```bash
   NODE=$(oc get pod \
     -l serving.kserve.io/inferenceservice=resnet-gpu-hp \
     -o jsonpath='{.items[0].spec.nodeName}')
   oc get node "$NODE" \
     -o jsonpath='{.status.allocatable.nvidia\.com/gpu}'
   ```

**Expected Results**:
- ClusterServingRuntime YAML does not contain hardcoded
  `nvidia.com/gpu` resource limits in the container spec
- Pod resource limits include `nvidia.com/gpu: 1` injected by the
  HardwareProfile (not from the template)
- Pod is scheduled on a node where `allocatable["nvidia.com/gpu"]` is
  greater than zero
- `oc describe pod` shows successful scheduling with GPU resource
  binding

**Notes**: To be filled later in the process.
