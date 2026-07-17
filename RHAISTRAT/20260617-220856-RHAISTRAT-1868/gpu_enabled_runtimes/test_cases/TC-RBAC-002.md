---
test_case_id: TC-RBAC-002
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-RBAC-002: Namespace admin deploys InferenceService with GPU runtime

**Objective**: Verify that a `namespace-admin` user can create an
InferenceService referencing the `mlserver-cuda-runtime`
ClusterServingRuntime within their project namespace, following the
same authorization boundaries as CPU runtimes.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator 12.9+
- `namespace-admin` user with admin role in the test namespace
- `mlserver-cuda-runtime` ClusterServingRuntime applied in
  `redhat-ods-applications` namespace by cluster-admin (TC-DEPLOY-001)
- HardwareProfile `nvidia-gpu-a100` (API group
  `infrastructure.opendatahub.io/v1`) available

**Test Steps**:
1. Authenticate as the `namespace-admin` user:
   ```bash
   oc login -u namespace-admin -p <password>
   oc project test-gpu-serving
   ```
2. Create an InferenceService using the GPU runtime:
   ```bash
   cat <<EOF | oc apply -f -
   apiVersion: serving.kserve.io/v1beta1
   kind: InferenceService
   metadata:
     name: resnet-gpu-rbac
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
3. Verify the InferenceService was created:
   ```bash
   oc get inferenceservice resnet-gpu-rbac
   ```
4. Wait for the InferenceService to reach `Ready`:
   ```bash
   oc wait inferenceservice/resnet-gpu-rbac --for=condition=Ready \
     --timeout=300s
   ```

**Expected Results**:
- InferenceService is created without authorization errors
- The InferenceService reaches `Ready` state
- The namespace-admin has the same level of access to GPU runtime
  InferenceServices as they have for CPU runtime InferenceServices

**Notes**: To be filled later in the process.
