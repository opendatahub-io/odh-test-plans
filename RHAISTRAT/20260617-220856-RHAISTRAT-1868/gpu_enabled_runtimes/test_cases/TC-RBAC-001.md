---
test_case_id: TC-RBAC-001
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-RBAC-001: Unprivileged user cannot create ClusterServingRuntime

**Objective**: Verify that a user with `data-scientist` (unprivileged)
role cannot create or modify ClusterServingRuntime resources, ensuring
cluster-scoped RBAC enforcement applies to the `mlserver-cuda-runtime`.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `data-scientist` user configured with namespace-scoped permissions
  only (no cluster-admin)
- `mlserver-cuda-runtime` ClusterServingRuntime present

**Test Steps**:
1. Authenticate as the `data-scientist` user:
   ```bash
   oc login -u data-scientist -p <password>
   ```
2. Attempt to create a ClusterServingRuntime:
   ```bash
   oc apply -f mlserver-cuda-runtime-template.yaml 2>&1
   ```
3. Attempt to list ClusterServingRuntimes:
   ```bash
   oc get clusterservingruntime 2>&1
   ```
4. Attempt to delete the GPU ClusterServingRuntime:
   ```bash
   oc delete clusterservingruntime mlserver-cuda-runtime 2>&1
   ```

**Expected Results**:
- Create attempt returns `Forbidden` error (HTTP 403)
- List attempt either returns `Forbidden` or returns results
  depending on RBAC read policy — but creation must be denied
- Delete attempt returns `Forbidden` error
- Error messages reference the user's insufficient permissions for
  cluster-scoped resources

**Notes**: To be filled later in the process.
