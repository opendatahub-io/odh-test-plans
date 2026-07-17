---
test_case_id: TC-DISC-003
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-DISC-003: CPU runtime remains visible in Dashboard after GPU runtime addition

**Objective**: Verify that the existing CPU `mlserver-onnx` runtime
remains visible and selectable in the Dashboard after the
`mlserver-cuda-runtime` is added to the cluster.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-onnx` ClusterServingRuntime present on the cluster
- `mlserver-cuda-runtime` ClusterServingRuntime applied in
  `redhat-ods-applications` namespace
- RHOAI Dashboard accessible

**Test Steps**:

1. Log in to the RHOAI Dashboard.
2. Navigate to the model serving runtime selection.
3. Verify `mlserver-onnx` (CPU) runtime is listed.
4. Verify both `mlserver-onnx` and `mlserver-cuda-runtime` are listed
   without conflicts.
5. Select the CPU runtime and verify the deployment flow proceeds
   without errors.

**Expected Results**:

- The CPU runtime `mlserver-onnx` is listed in the runtime
  selection dropdown
- Both runtimes appear as separate entries with distinct names
- Selecting the CPU runtime does not show GPU-related
  HardwareProfile recommendations

**Notes**: To be filled later in the process.
