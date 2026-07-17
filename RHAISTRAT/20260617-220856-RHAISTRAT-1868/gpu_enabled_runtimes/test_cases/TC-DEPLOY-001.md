---
test_case_id: TC-DEPLOY-001
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-DEPLOY-001: Apply mlserver-cuda-runtime ClusterServingRuntime template

**Objective**: Verify that the `mlserver-cuda-runtime-template` in the
`redhat-ods-applications` namespace creates the
`mlserver-cuda-runtime` ClusterServingRuntime successfully with correct
labels, annotations, and container configuration.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- KServe and odh-model-controller deployed
- `cluster-admin` access
- `mlserver-cuda-runtime-template` present in
  `redhat-ods-applications` namespace

**Test Steps**:

1. Verify the `mlserver-cuda-runtime` ClusterServingRuntime exists:

   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime -o yaml
   ```

2. Verify the `opendatahub.io/dashboard: "true"` label is present:

   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime \
     -o jsonpath='{.metadata.labels.opendatahub\.io/dashboard}'
   ```

3. Verify the `opendatahub.io/recommended-accelerators` annotation
   contains `nvidia.com/gpu`:

   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime \
     -o jsonpath='{.metadata.annotations.opendatahub\.io/recommended-accelerators}'
   ```

4. Verify the container image reference uses `$(mlserver-cuda-image)`:

   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime \
     -o jsonpath='{.spec.containers[?(@.name=="kserve-container")].image}'
   ```

5. Verify supported model format is ONNX v1 with single-model serving:

   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime \
     -o jsonpath='{.spec.supportedModelFormats}'
   ```

6. Verify the runtime uses HTTP port 8080 and V2 protocol:

   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime \
     -o jsonpath='{.spec.containers[?(@.name=="kserve-container")].ports}'
   ```

**Expected Results**:

- ClusterServingRuntime `mlserver-cuda-runtime` exists without error
  conditions
- Label `opendatahub.io/dashboard` equals `"true"`
- Annotation `opendatahub.io/recommended-accelerators` contains
  `["nvidia.com/gpu"]`
- Container image reference is `$(mlserver-cuda-image)`
- Supported model format is `onnx` version `"1"` with
  `multiModel: false`
- Container port is `8080` with protocol `v2`

**Notes**: To be filled later in the process.
