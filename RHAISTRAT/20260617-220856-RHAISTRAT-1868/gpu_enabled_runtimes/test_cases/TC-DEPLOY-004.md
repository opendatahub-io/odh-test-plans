---
test_case_id: TC-DEPLOY-004
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-DEPLOY-004: Verify kustomization.yaml includes GPU runtime template

**Objective**: Verify that the `kustomization.yaml` in
`odh-model-controller/config/runtimes/` includes the new
`mlserver-cuda-runtime` template as a resource and renders correctly.

**Preconditions**:

- Access to the `odh-model-controller` repository or deployed
  configuration

**Test Steps**:

1. Read the `kustomization.yaml` file:

   ```bash
   cat odh-model-controller/config/runtimes/kustomization.yaml
   ```

2. Verify the GPU runtime template file is listed in the `resources`
   section.
3. Run `kustomize build` to validate the kustomization renders
   without errors:

   ```bash
   kustomize build odh-model-controller/config/runtimes/
   ```

4. Confirm the rendered output includes the `mlserver-cuda-runtime`
   ClusterServingRuntime resource alongside the existing CPU runtime.

**Expected Results**:

- `kustomization.yaml` contains an entry for the GPU runtime template
  file in its `resources` list
- `kustomize build` completes without errors
- The rendered output includes a ClusterServingRuntime named
  `mlserver-cuda-runtime` alongside the existing CPU runtime
  (`mlserver-onnx`)

**Notes**: To be filled later in the process.
