---
test_case_id: TC-UPGRADE-001
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-UPGRADE-001: GPU template appears after operator upgrade

**Objective**: Verify that upgrading the RHOAI operator from a release
without the GPU runtime to RHOAI 3.5 GA results in the new
`mlserver-cuda-runtime` ClusterServingRuntime template appearing
automatically in `redhat-ods-applications` namespace.

**Preconditions**:
- OCP 4.20+ cluster
- RHOAI operator installed at a version that does NOT include the
  GPU runtime (pre-3.5)
- KServe and odh-model-controller running

**Test Steps**:
1. Verify the GPU runtime does not exist before upgrade:
   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime 2>&1
   ```
2. Upgrade the RHOAI operator to 3.5 GA.
3. Wait for the operator upgrade to complete:
   ```bash
   oc wait csv -n openshift-operators \
     -l operators.coreos.com/rhods-operator.openshift-operators \
     --for=jsonpath='{.status.phase}'=Succeeded --timeout=600s
   ```
4. Verify the GPU ClusterServingRuntime now exists:
   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime -o yaml
   ```
5. Verify the existing CPU ClusterServingRuntime is unchanged:
   ```bash
   oc get clusterservingruntime mlserver-onnx -o yaml
   ```

**Expected Results**:
- Before upgrade: `mlserver-cuda-runtime` returns "not found"
- After upgrade: `mlserver-cuda-runtime` ClusterServingRuntime exists
  with `opendatahub.io/dashboard: "true"` label and
  `opendatahub.io/recommended-accelerators` annotation
- `mlserver-onnx` (CPU) ClusterServingRuntime remains present and
  unchanged
- Operator CSV reaches `Succeeded` phase

**Notes**: To be filled later in the process.
