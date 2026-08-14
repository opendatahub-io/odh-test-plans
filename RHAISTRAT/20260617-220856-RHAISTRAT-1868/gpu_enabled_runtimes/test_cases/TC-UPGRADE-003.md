---
test_case_id: TC-UPGRADE-003
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-UPGRADE-003: Rollback removes GPU template without CPU impact

**Objective**: Verify that removing the `mlserver-cuda-runtime`
ServingRuntime (simulating a rollback) does not impact existing
CPU-based InferenceServices using `mlserver-runtime`.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-cuda-runtime` ServingRuntime applied in
  `redhat-ods-applications` namespace
- No active GPU InferenceServices (or they have been deleted)
- `admin_client` fixture available

**Test Steps**:

1. Deploy a CPU InferenceService with an external route and verify
   baseline inference:

   ```python
   cpu_isvc = create_isvc(
       client=admin_client,
       name="resnet-cpu",
       namespace=project.name,
       runtime=ModelInferenceRuntime.MLSERVER_RUNTIME,
       model_format="sklearn",
       external_route=True,
   )
   cpu_url = get_exposed_isvc_url(cpu_isvc)
   baseline = send_rest_request(
       url=cpu_url,
       data=inference_payload,
   )
   validate_deterministic_snapshot(response=baseline)
   ```

2. Delete the GPU ServingRuntime using `ocp-resources`:

   ```python
   gpu_runtime = ServingRuntime(
       client=admin_client,
       name="mlserver-cuda-runtime",
       namespace="redhat-ods-applications",
   )
   gpu_runtime.delete()
   gpu_runtime.wait_deleted()
   ```

3. Verify the CPU ISVC remains Ready:

   ```python
   cpu_isvc.wait_for_condition(
       condition="Ready",
       status="True",
       timeout=120,
   )
   ```

4. Repeat CPU inference via the external route and verify the
   response is unchanged:

   ```python
   cpu_url = get_exposed_isvc_url(cpu_isvc)
   post_rollback = send_rest_request(
       url=cpu_url,
       data=inference_payload,
   )
   validate_deterministic_snapshot(response=post_rollback)
   ```

5. Verify the GPU runtime no longer exists:

   ```python
   removed_runtime = ServingRuntime(
       client=admin_client,
       name="mlserver-cuda-runtime",
       namespace="redhat-ods-applications",
   )
   assert not removed_runtime.exists, (
       "GPU runtime still exists after deletion"
   )
   ```

**Expected Results**:

- GPU ServingRuntime is successfully deleted and confirmed absent
- CPU InferenceService remains in `Ready` state after GPU runtime
  removal
- CPU inference response passes `validate_deterministic_snapshot`
  (structure-only validation, no specific class index assertions)
- No errors in odh-model-controller logs related to the CPU
  runtime

**Notes**: This test uses `runtime.delete()` via `ocp-resources`
instead of `oc delete`. No port-forwarding is used -- all inference
requests go through the external route via
`get_exposed_isvc_url(isvc)`.
