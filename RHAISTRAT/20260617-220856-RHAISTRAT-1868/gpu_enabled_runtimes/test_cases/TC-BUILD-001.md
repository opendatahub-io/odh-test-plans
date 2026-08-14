---
test_case_id: TC-BUILD-001
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-BUILD-001: CSV relatedImages and template image digest cross-reference

**Objective**: Verify that the RHOAI operator CSV
(ClusterServiceVersion) `relatedImages` section includes both
CPU and GPU MLServer image references using digest format, and
that these digests match the images referenced in the
ServingRuntime templates.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- RHOAI operator CSV accessible in `redhat-ods-operator`
  namespace
- ServingRuntime templates for `mlserver-runtime` and
  `mlserver-cuda-runtime` present in
  `redhat-ods-applications` namespace

**Test Steps**:

1. Retrieve the RHOAI operator CSV from the
   `redhat-ods-operator` namespace using the
   `ocp-resources` library:

   ```python
   from ocp_resources.cluster_service_version import (
       ClusterServiceVersion,
   )

   csvs = list(
       ClusterServiceVersion.get(
           dyn_client=client,
           namespace="redhat-ods-operator",
           label_selector=(
               "operators.coreos.com/"
               "rhods-operator.redhat-ods-operator"
           ),
       )
   )
   csv = csvs[0]
   related_images = csv.instance.spec.relatedImages
   ```

2. Extract the CPU and GPU MLServer image entries from
   `relatedImages`. Names use underscores:
   `odh_mlserver_image` (CPU) and `odh_mlserver_cuda_image`
   (GPU):

   ```python
   gpu_entry = next(
       img for img in related_images
       if "mlserver_cuda_image" in img.name
   )
   cpu_entry = next(
       img for img in related_images
       if "mlserver_image" in img.name
       and "cuda" not in img.name
   )
   ```

3. Verify both entries use `@sha256:` digest references:

   ```python
   assert "@sha256:" in gpu_entry.image, (
       "GPU image must use digest format"
   )
   assert "@sha256:" in cpu_entry.image, (
       "CPU image must use digest format"
   )
   ```

4. Read the image references from the ServingRuntime templates
   in `redhat-ods-applications`:

   ```python
   from utilities.serving_runtime import (
       get_runtime_image_from_template,
   )

   gpu_template_image = get_runtime_image_from_template(
       client=client,
       template_name="mlserver-cuda-runtime",
       namespace="redhat-ods-applications",
   )
   cpu_template_image = get_runtime_image_from_template(
       client=client,
       template_name="mlserver-runtime",
       namespace="redhat-ods-applications",
   )
   ```

5. Cross-reference the template image digests with the CSV
   `relatedImages` digests:

   ```python
   assert gpu_template_image == gpu_entry.image, (
       "GPU template image must match CSV relatedImages digest"
   )
   assert cpu_template_image == cpu_entry.image, (
       "CPU template image must match CSV relatedImages digest"
   )
   ```

**Expected Results**:

- `relatedImages` contains entries named
  `odh_mlserver_cuda_image` (GPU) and `odh_mlserver_image`
  (CPU)
- Both image references use `@sha256:` digest format (not
  `:tag`)
- The image digest in each ServingRuntime template matches the
  corresponding `relatedImages` entry in the CSV
- The operator label used is
  `operators.coreos.com/rhods-operator.redhat-ods-operator`

**Notes**: This test absorbs the former TC-BUILD-004 (template
uses `$(mlserver-cuda-image)` variable). The CSV is read from
`redhat-ods-operator` namespace; templates are read from
`redhat-ods-applications`. Uses `get_runtime_image_from_template`
from `utilities/serving_runtime.py` for template image extraction.
