---
test_case_id: TC-DEPLOY-002
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-DEPLOY-002: Deploy InferenceService with GPU runtime and HardwareProfile

**Objective**: Verify that deploying an InferenceService using the
`mlserver-cuda-runtime` with the `nvidia-gpu-a100` HardwareProfile
results in a running pod scheduled on an NVIDIA GPU node with correct
resource allocation and tolerations.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator 12.9+
- `mlserver-cuda-runtime` ClusterServingRuntime applied in
  `redhat-ods-applications` namespace (TC-DEPLOY-001)
- HardwareProfile `nvidia-gpu-a100` (API group
  `infrastructure.opendatahub.io/v1`) created
- At least one worker node with NVIDIA GPU hardware
- ResNet-50 ONNX model available in S3-compatible storage
- `namespace-admin` access in the test namespace

**Test Steps**:
1. Verify the HardwareProfile CR exists with correct API group:
   ```bash
   oc get hardwareprofile nvidia-gpu-a100 \
     -o jsonpath='{.apiVersion}'
   ```
2. Create an InferenceService referencing the `mlserver-cuda-runtime`
   and the GPU HardwareProfile:
   ```bash
   cat <<EOF | oc apply -f -
   apiVersion: serving.kserve.io/v1beta1
   kind: InferenceService
   metadata:
     name: resnet-gpu
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
3. Wait for the InferenceService to reach `Ready` state:
   ```bash
   oc wait inferenceservice/resnet-gpu --for=condition=Ready \
     --timeout=300s
   ```
4. Verify the predictor pod is running on a GPU node:
   ```bash
   NODE=$(oc get pod \
     -l serving.kserve.io/inferenceservice=resnet-gpu \
     -o jsonpath='{.items[0].spec.nodeName}')
   oc get node "$NODE" \
     -o jsonpath='{.status.allocatable.nvidia\.com/gpu}'
   ```
5. Confirm the pod has `nvidia.com/gpu` resource allocated:
   ```bash
   oc get pod -l serving.kserve.io/inferenceservice=resnet-gpu \
     -o jsonpath='{.items[0].spec.containers[?(@.name=="kserve-container")].resources}'
   ```
6. Verify the toleration for GPU-tainted nodes is applied:
   ```bash
   oc get pod -l serving.kserve.io/inferenceservice=resnet-gpu \
     -o jsonpath='{.items[0].spec.tolerations}' | jq .
   ```

**Expected Results**:
- HardwareProfile API version is `infrastructure.opendatahub.io/v1`
- InferenceService reaches `Ready` condition within 300 seconds
- Predictor pod is `Running` on a node with `nvidia.com/gpu`
  allocatable capacity greater than zero
- Pod resource requests include `nvidia.com/gpu: 1`
- Pod tolerations include `nvidia.com/gpu` operator `Exists`, effect
  `NoSchedule`
- No GPU-related scheduling warnings or errors in `oc describe pod`

**Notes**: To be filled later in the process.
