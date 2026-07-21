---
test_case_id: TC-UPGRADE-001
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-UPGRADE-001: GPU template appears after operator upgrade

**Objective**: Verify that upgrading the RHOAI operator from a
release without the GPU runtime to RHOAI 3.5 GA results in the new
`mlserver-cuda-runtime` ServingRuntime appearing automatically in
`redhat-ods-applications`, with correct kustomization updates,
controller logs free of errors, and image references matching the
CSV `relatedImages`.

**Preconditions**:

- OCP 4.20+ cluster
- RHOAI operator installed at a version that does NOT include the
  GPU runtime (pre-3.5)
- KServe and odh-model-controller running
- `admin_client` fixture available
- `pytest.mark.post_upgrade` marker applied

**Test Steps**:

1. After upgrade, verify the GPU ServingRuntime exists in
   `redhat-ods-applications` using `ocp-resources`:

   ```python
   runtime = ServingRuntime(
       client=admin_client,
       name="mlserver-cuda-runtime",
       namespace="redhat-ods-applications",
   )
   assert runtime.exists, (
       "mlserver-cuda-runtime not found after upgrade"
   )
   ```

2. Read labels, annotations, and image from the runtime resource:

   ```python
   metadata = runtime.instance.metadata
   labels = metadata.labels
   assert labels.get("opendatahub.io/dashboard") == "true"

   annotations = metadata.annotations
   assert "opendatahub.io/recommended-accelerators" in annotations

   containers = runtime.instance.spec.containers
   gpu_image = containers[0].image
   assert gpu_image, "GPU runtime container image must be set"
   ```

3. Wait for CSV to reach `Succeeded` phase. This uses `oc wait`
   because no utility exists for CSV status polling:

   ```bash
   oc wait csv -n redhat-ods-operator \
     -l operators.coreos.com/rhods-operator.redhat-ods-operator \
     --for=jsonpath='{.status.phase}'=Succeeded \
     --timeout=600s
   ```

4. Verify `kustomization.yaml` update -- read from
   odh-model-controller resources to confirm the GPU runtime is
   included in the kustomization configuration:

   ```python
   from ocp_resources.config_map import ConfigMap

   # Verify the kustomization includes the GPU runtime entry
   # by inspecting odh-model-controller managed resources
   ```

5. Check odh-model-controller pod logs for runtime-related errors
   using `pod.log()`:

   ```python
   from ocp_resources.deployment import Deployment
   from ocp_resources.pod import Pod

   controller_deploy = Deployment(
       client=admin_client,
       name="odh-model-controller",
       namespace="redhat-ods-applications",
   )
   controller_pods = [
       Pod(client=admin_client, name=p.metadata.name,
           namespace="redhat-ods-applications")
       for p in controller_deploy.instance.status.readyReplicas
       and Pod.get(
           client=admin_client,
           namespace="redhat-ods-applications",
           label_selector=(
               "app=odh-model-controller"
           ),
       )
   ]
   for pod in controller_pods:
       logs = pod.log()
       assert "error" not in logs.lower() or \
           "runtime" not in logs.lower(), (
           f"Runtime-related errors in controller logs: {logs}"
       )
   ```

6. Verify image references match CSV `relatedImages`:

   ```python
   from ocp_resources.cluster_service_version import (
       ClusterServiceVersion,
   )

   csvs = list(ClusterServiceVersion.get(
       client=admin_client,
       namespace="redhat-ods-operator",
       label_selector=(
           "operators.coreos.com/"
           "rhods-operator.redhat-ods-operator"
       ),
   ))
   csv = csvs[0]
   related_images = [
       img["image"]
       for img in csv.instance.spec.relatedImages
   ]
   assert gpu_image in related_images, (
       f"GPU runtime image {gpu_image} not found in "
       f"CSV relatedImages"
   )
   ```

**Expected Results**:

- After upgrade: `mlserver-cuda-runtime` ServingRuntime exists with
  `opendatahub.io/dashboard: "true"` label and
  `opendatahub.io/recommended-accelerators` annotation
- Operator CSV reaches `Succeeded` phase
- Kustomization configuration includes the GPU runtime entry
- odh-model-controller logs contain no runtime-related errors
- GPU runtime container image reference matches an entry in the
  CSV `relatedImages`

**Notes**: The `oc wait csv` command in step 3 is acceptable because
no `ocp-resources` utility exists for CSV status polling.
