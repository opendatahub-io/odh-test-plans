---
test_case_id: TC-RBAC-003
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-RBAC-003: Unprivileged user cannot modify recommended-accelerators annotation

**Objective**: Verify that an unprivileged (`data-scientist`) user
cannot modify the `opendatahub.io/recommended-accelerators` annotation
on the `mlserver-cuda-runtime` ClusterServingRuntime, preventing
unauthorized redirection of GPU allocation.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-cuda-runtime` ClusterServingRuntime applied in
  `redhat-ods-applications` namespace (TC-DEPLOY-001)
- `data-scientist` user configured with namespace-scoped permissions

**Test Steps**:

1. Authenticate as the `data-scientist` user:

   ```bash
   oc login -u data-scientist -p <password>
   ```

2. Attempt to patch the `recommended-accelerators` annotation:

   ```bash
   oc annotate clusterservingruntime mlserver-cuda-runtime \
     opendatahub.io/recommended-accelerators='["amd.com/gpu"]' \
     --overwrite 2>&1
   ```

3. Attempt to remove the annotation:

   ```bash
   oc annotate clusterservingruntime mlserver-cuda-runtime \
     opendatahub.io/recommended-accelerators- 2>&1
   ```

**Expected Results**:

- Both patch and remove attempts return `Forbidden` error (HTTP 403)
- The annotation remains unchanged at `'["nvidia.com/gpu"]'`
- Error messages reference insufficient permissions for
  cluster-scoped resources

**Validation**:

- Log back in as `cluster-admin` and verify the annotation is
  unchanged:

  ```bash
  oc get clusterservingruntime mlserver-cuda-runtime \
    -o jsonpath='{.metadata.annotations.opendatahub\.io/recommended-accelerators}'
  ```

**Notes**: To be filled later in the process.
