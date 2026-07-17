---
test_case_id: TC-DEPLOY-005
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-DEPLOY-005: Verify recommended-accelerators annotation on GPU runtime

**Objective**: Verify that the `mlserver-cuda-runtime`
ClusterServingRuntime has the
`opendatahub.io/recommended-accelerators` annotation correctly set
to reference NVIDIA GPU HardwareProfiles, and that the CPU runtime
does not have GPU accelerator recommendations.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-cuda-runtime` ClusterServingRuntime applied in
  `redhat-ods-applications` namespace (TC-DEPLOY-001)
- CPU runtime (`mlserver-onnx`) also present

**Test Steps**:
1. Retrieve the annotation value from the GPU runtime:
   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime \
     -o jsonpath='{.metadata.annotations.opendatahub\.io/recommended-accelerators}'
   ```
2. Parse the annotation value as JSON and verify it contains
   `nvidia.com/gpu`.
3. Verify the existing CPU runtime does NOT have the
   `recommended-accelerators` annotation set to GPU resources:
   ```bash
   oc get clusterservingruntime mlserver-onnx \
     -o jsonpath='{.metadata.annotations.opendatahub\.io/recommended-accelerators}'
   ```

**Expected Results**:
- GPU runtime annotation value is `'["nvidia.com/gpu"]'` (valid JSON
  array containing the NVIDIA GPU resource identifier)
- CPU runtime either has no `recommended-accelerators` annotation or
  its annotation does not include `nvidia.com/gpu`

**Notes**: To be filled later in the process.
