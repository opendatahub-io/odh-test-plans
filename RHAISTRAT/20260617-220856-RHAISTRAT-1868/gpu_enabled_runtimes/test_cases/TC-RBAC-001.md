---
test_case_id: TC-RBAC-001
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-RBAC-001: Unprivileged user cannot modify ServingRuntime in redhat-ods-applications

**Objective**: Verify that a user with `data-scientist` (unprivileged)
role cannot create, patch, or delete namespace-scoped ServingRuntime
resources in the `redhat-ods-applications` namespace, while
confirming that the same user CAN create a ServingRuntime in their
own project namespace.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `data-scientist` user configured with namespace-scoped permissions
  only (no cluster-admin)
- `mlserver-cuda-runtime` ServingRuntime present in
  `redhat-ods-applications`
- `unprivileged_client` fixture available from `tests/conftest.py`
- `admin_client` fixture available for verification steps
- A user-owned Data Science Project namespace exists

**Test Steps**:

1. Obtain the unprivileged client via the `unprivileged_client`
   fixture (NOT `oc login`).

2. Attempt to create a ServingRuntime in
   `redhat-ods-applications` using `unprivileged_client`. Assert
   `ForbiddenError` (HTTP 403):

   ```python
   from ocp_resources.exceptions import ForbiddenError

   with pytest.raises(ForbiddenError):
       ServingRuntimeFromTemplate(
           client=unprivileged_client,
           name=ModelInferenceRuntime.MLSERVER_CUDA_RUNTIME,
           namespace="redhat-ods-applications",
           template_name=RuntimeTemplates.MLSERVER_CUDA,
       ).__enter__()
   ```

3. Attempt to patch the existing GPU ServingRuntime in
   `redhat-ods-applications` using `unprivileged_client`. Assert
   `ForbiddenError`:

   ```python
   runtime = ServingRuntime(
       client=unprivileged_client,
       name="mlserver-cuda-runtime",
       namespace="redhat-ods-applications",
   )
   with pytest.raises(ForbiddenError):
       runtime.patch(
           body={"metadata": {"labels": {"test": "forbidden"}}},
       )
   ```

4. Attempt to delete the GPU ServingRuntime in
   `redhat-ods-applications` using `unprivileged_client`. Assert
   `ForbiddenError`:

   ```python
   with pytest.raises(ForbiddenError):
       runtime.delete()
   ```

5. Attempt to patch an annotation on the existing GPU runtime in
   `redhat-ods-applications` using `unprivileged_client`. Assert
   `ForbiddenError`:

   ```python
   with pytest.raises(ForbiddenError):
       runtime.patch(
           body={
               "metadata": {
                   "annotations": {
                       "test-annotation": "forbidden",
                   },
               },
           },
       )
   ```

6. Positive case -- verify the data-scientist CAN create a
   ServingRuntime in their OWN project namespace:

   ```python
   with ServingRuntimeFromTemplate(
       client=unprivileged_client,
       name="user-test-runtime",
       namespace=user_project.name,
       template_name=RuntimeTemplates.MLSERVER_CUDA,
   ) as user_runtime:
       assert user_runtime.exists
   ```

7. Verify the GPU runtime in `redhat-ods-applications` is unchanged
   using `admin_client`:

   ```python
   admin_runtime = ServingRuntime(
       client=admin_client,
       name="mlserver-cuda-runtime",
       namespace="redhat-ods-applications",
   )
   assert admin_runtime.exists
   labels = admin_runtime.instance.metadata.labels
   assert labels.get("opendatahub.io/dashboard") == "true"
   assert "test" not in labels
   annotations = admin_runtime.instance.metadata.annotations
   assert "test-annotation" not in annotations
   ```

**Expected Results**:

- Create attempt in `redhat-ods-applications` raises
  `ForbiddenError` (HTTP 403)
- Patch (labels) attempt raises `ForbiddenError`
- Delete attempt raises `ForbiddenError`
- Patch (annotations) attempt raises `ForbiddenError`
- User CAN successfully create a ServingRuntime in their own
  project namespace
- The GPU runtime in `redhat-ods-applications` remains unchanged
  after all denied operations

**Notes**: The ServingRuntime is namespace-scoped, not
cluster-scoped. The RBAC concern is namespace boundary enforcement:
unprivileged users must not be able to modify resources in the
`redhat-ods-applications` namespace.
