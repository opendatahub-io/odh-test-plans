---
test_case_id: TC-DISC-001
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-DISC-001: GPU runtime appears in Dashboard via dashboard label

**Objective**: Verify that the `mlserver-cuda-runtime`
ClusterServingRuntime appears in the RHOAI Dashboard runtime selection
list due to the `opendatahub.io/dashboard: "true"` label, without any
Dashboard code changes.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-cuda-runtime` ClusterServingRuntime applied in
  `redhat-ods-applications` namespace with
  `opendatahub.io/dashboard: "true"` label (TC-DEPLOY-001)
- RHOAI Dashboard accessible
- `data-scientist` user credentials for Dashboard login

**Test Steps**:

1. Log in to the RHOAI Dashboard as a `data-scientist` user.
2. Navigate to the model serving section (Data Science Projects >
   select project > Deploy model).
3. Open the runtime selection dropdown.
4. Verify the GPU runtime (`mlserver-cuda-runtime`) appears in the
   list.
5. Verify the runtime is distinguishable from the CPU variant
   (`mlserver-onnx`) by name.

**Expected Results**:

- The runtime selection dropdown includes an entry for
  `mlserver-cuda-runtime`
- The GPU runtime is listed alongside the existing CPU
  `mlserver-onnx` runtime
- No Dashboard errors or console warnings related to the GPU runtime
  discovery

**Notes**: To be filled later in the process.
