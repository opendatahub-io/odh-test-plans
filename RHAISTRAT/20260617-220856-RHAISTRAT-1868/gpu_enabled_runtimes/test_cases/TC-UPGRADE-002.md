---
test_case_id: TC-UPGRADE-002
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-UPGRADE-002: CPU InferenceService recovers after operator upgrade

**Objective**: Verify that an existing CPU-based InferenceService
using `mlserver-runtime` is functional before an operator upgrade to
RHOAI 3.5 GA, and recovers to a Ready state with correct inference
output within 5 minutes after the upgrade completes.

**Preconditions**:

- OCP 4.20+ cluster
- RHOAI operator installed at a pre-3.5 version ready to upgrade
- KServe and odh-model-controller running
- `admin_client` fixture available
- `pytest.mark.pre_upgrade` and `pytest.mark.post_upgrade` markers
  applied

**Test Steps (pre-upgrade)**:

1. Deploy a CPU InferenceService with an external route:

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
   ```

2. Record baseline inference response using `send_rest_request` and
   validate the response structure with
   `validate_deterministic_snapshot`:

   ```python
   baseline = send_rest_request(
       url=cpu_url,
       data=inference_payload,
   )
   validate_deterministic_snapshot(response=baseline)
   ```

3. Record restart counts for the predictor pod:

   ```python
   from tests.model_serving.model_runtime.utils import (
       get_restart_counts,
   )

   predictor_pod = get_predictor_pod(cpu_isvc)
   pre_upgrade_restarts = get_restart_counts(predictor_pod)
   ```

**Test Steps (post-upgrade)**:

1. Verify the ISVC reaches Ready condition within 5 minutes after
   upgrade completion:

   ```python
   conditions = cpu_isvc.instance.status.conditions
   ready_condition = next(
       (c for c in conditions if c.type == "Ready"), None
   )
   assert ready_condition is not None, (
       "Ready condition not found on ISVC"
   )
   assert ready_condition.status == "True", (
       f"ISVC not Ready after upgrade: {ready_condition.message}"
   )
   ```

2. Repeat inference and validate the response structure matches
   expectations:

   ```python
   cpu_url = get_exposed_isvc_url(cpu_isvc)
   post_upgrade = send_rest_request(
       url=cpu_url,
       data=inference_payload,
   )
   validate_deterministic_snapshot(response=post_upgrade)
   ```

3. Compare restart counts -- there should be zero additional
   restarts on the predictor pod:

   ```python
   post_upgrade_restarts = get_restart_counts(predictor_pod)
   for container, count in post_upgrade_restarts.items():
       pre_count = pre_upgrade_restarts.get(container, 0)
       assert count == pre_count, (
           f"Container {container} restarted during upgrade: "
           f"pre={pre_count}, post={count}"
       )
   ```

**Expected Results**:

- Pre-upgrade: CPU ISVC is deployed, serving inference, and
  `validate_deterministic_snapshot` passes on the baseline response
- Post-upgrade: ISVC returns to `Ready` state within 5 minutes
- Post-upgrade inference response passes
  `validate_deterministic_snapshot` (structure-only validation,
  no specific class index assertions)
- Predictor pod restart count is unchanged (zero additional
  restarts)

**Notes**: OLM upgrades may restart the odh-model-controller, so
zero-downtime is not expected. The acceptance criterion is recovery
within 5 minutes post-upgrade, not continuous availability during
the upgrade. No `oc get inferenceservice -o jsonpath` commands are
used -- all status checks use `ocp-resources` Python objects.
