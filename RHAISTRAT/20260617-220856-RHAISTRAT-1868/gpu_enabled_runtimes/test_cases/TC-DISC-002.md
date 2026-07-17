---
test_case_id: TC-DISC-002
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-DISC-002: recommended-accelerators annotation links GPU runtime to HardwareProfile

**Objective**: Verify that the
`opendatahub.io/recommended-accelerators` annotation on the
`mlserver-cuda-runtime` correctly associates it with the
`nvidia-gpu-a100` HardwareProfile in the Dashboard, so users see
GPU acceleration options when selecting this runtime.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-cuda-runtime` ClusterServingRuntime applied in
  `redhat-ods-applications` namespace with the
  `recommended-accelerators` annotation (TC-DEPLOY-001)
- HardwareProfile `nvidia-gpu-a100` (API group
  `infrastructure.opendatahub.io/v1`) exists on the cluster
- RHOAI Dashboard accessible

**Test Steps**:
1. Log in to the RHOAI Dashboard.
2. Navigate to model deployment and select the
   `mlserver-cuda-runtime` runtime.
3. Verify that the HardwareProfile selection shows NVIDIA GPU options
   linked via the `recommended-accelerators` annotation.
4. Verify that selecting the CPU runtime (`mlserver-onnx`) does NOT
   show GPU HardwareProfile recommendations.
5. Verify the annotation value via CLI:
   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime \
     -o jsonpath='{.metadata.annotations.opendatahub\.io/recommended-accelerators}'
   ```

**Expected Results**:
- When the GPU runtime is selected, the Dashboard shows
  `nvidia-gpu-a100` HardwareProfile as a recommended accelerator
- When the CPU runtime is selected, no GPU HardwareProfiles are
  listed as recommended
- The annotation value contains `nvidia.com/gpu`
- The association is driven by the annotation value, not by
  Dashboard-side code

**Notes**: To be filled later in the process.
