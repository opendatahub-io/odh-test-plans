---
test_case_id: TC-UPGRADE-004
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-UPGRADE-004: kustomization.yaml changes applied during upgrade

**Objective**: Verify that the `kustomization.yaml` changes in
`odh-model-controller/config/runtimes/` are applied cleanly during
the RHOAI 3.5 GA operator upgrade, resulting in both CPU and GPU
runtime templates being rendered.

**Preconditions**:

- OCP 4.20+ cluster
- RHOAI operator upgraded to 3.5 GA (TC-UPGRADE-001 passed)

**Test Steps**:

1. Verify the odh-model-controller deployment reflects the updated
   kustomization:

   ```bash
   oc get deployment odh-model-controller -n redhat-ods-applications \
     -o jsonpath='{.metadata.generation}'
   ```

2. Check that both ClusterServingRuntimes exist:

   ```bash
   oc get clusterservingruntime mlserver-onnx -o name
   oc get clusterservingruntime mlserver-cuda-runtime -o name
   ```

3. Verify no reconciliation errors in the odh-model-controller logs:

   ```bash
   oc logs deployment/odh-model-controller \
     -n redhat-ods-applications --tail=100 \
     | grep -iE "(error|fail)" | grep -i "runtime"
   ```

4. Verify the GPU runtime template content matches expectations:

   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime \
     -o jsonpath='{.spec.containers[?(@.name=="kserve-container")].image}'
   ```

**Expected Results**:

- Both `mlserver-onnx` and `mlserver-cuda-runtime`
  ClusterServingRuntimes exist after upgrade
- No runtime-related errors in odh-model-controller logs
- GPU runtime has the correct container image reference
  (`$(mlserver-cuda-image)` resolved)
- odh-model-controller deployment has been updated

**Notes**: To be filled later in the process.
