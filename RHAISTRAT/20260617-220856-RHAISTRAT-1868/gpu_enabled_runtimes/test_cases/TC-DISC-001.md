---
test_case_id: TC-DISC-001
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: "N/A"
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-DISC-001: Dashboard GPU runtime discovery

**Objective**: Verify that the `mlserver-cuda-runtime` ServingRuntime
appears in the RHOAI Dashboard runtime selection list with GPU
accelerator recommendation, alongside the existing CPU
`mlserver-runtime`, due to the `opendatahub.io/dashboard: "true"`
label.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-cuda-runtime` ServingRuntime applied in
  `redhat-ods-applications` namespace with
  `opendatahub.io/dashboard: "true"` label (TC-DEPLOY-001)
- RHOAI Dashboard accessible
- `data-scientist` user credentials for Dashboard login
- A Data Science Project exists and is available for model serving

**Test Steps**:

1. Log in to the RHOAI Dashboard as a `data-scientist` user.
2. Navigate to **Model Serving** in the left navigation panel.
3. Select a Data Science Project from the project list.
4. Click **Add model server** to open the server creation dialog.
5. Open the **Serving runtime** dropdown.
6. **Assertion 1 -- GPU runtime visible with accelerator
   recommendation**: Verify the dropdown includes
   `mlserver-cuda-runtime` and that it shows a GPU accelerator
   recommendation (driven by the
   `opendatahub.io/recommended-accelerators` annotation).
   Take a screenshot of the dropdown showing the GPU runtime entry.
7. **Assertion 2 -- HardwareProfile recommendation visible**: Verify
   the GPU runtime entry displays a HardwareProfile recommendation
   in the UI (e.g., an indicator that a GPU hardware profile is
   suggested for this runtime). Take a screenshot.
8. **Assertion 3 -- CPU runtime still visible**: Verify the CPU
   `mlserver-runtime` is also listed in the same dropdown alongside
   the GPU runtime. Both runtimes must be selectable. Take a
   screenshot showing both entries.

**Expected Results**:

- The runtime selection dropdown includes an entry for
  `mlserver-cuda-runtime` with a GPU accelerator recommendation
- The GPU runtime displays a HardwareProfile recommendation in
  the Dashboard UI
- The CPU `mlserver-runtime` remains visible and selectable in the
  same dropdown
- No Dashboard errors or console warnings related to GPU runtime
  discovery

**Notes**: This is a manual test case. Dashboard UI interaction is
incompatible with pytest automation. Screenshots should be captured
at each assertion step for evidence.
