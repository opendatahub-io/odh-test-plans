---
test_case_id: TC-BUILD-004
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-BUILD-004: GPU runtime template references image via variable

**Objective**: Verify that the `mlserver-cuda-runtime` ClusterServingRuntime
template references the GPU container image via `$(mlserver-cuda-image)`
variable (auto-populated on RHOAI clusters), which resolves to a digest
for air-gapped image mirroring compatibility.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-cuda-runtime` ClusterServingRuntime applied in
  `redhat-ods-applications` namespace

**Test Steps**:

1. Inspect the ClusterServingRuntime template for image references:

   ```bash
   oc get clusterservingruntime mlserver-cuda-runtime -o yaml \
     | grep "image:"
   ```

2. Verify the resolved image reference uses `@sha256:` digest format
   on the deployed cluster.
3. Cross-reference the digest with the CSV `relatedImages` entry to
   confirm they match:

   ```bash
   CSV=$(oc get csv -n openshift-operators \
     -l operators.coreos.com/rhods-operator.openshift-operators \
     -o jsonpath='{.items[0].metadata.name}')
   oc get csv "$CSV" -n openshift-operators \
     -o jsonpath='{.spec.relatedImages}' \
     | jq '.[] | select(.name | contains("mlserver-cuda"))'
   ```

**Expected Results**:

- The `image:` field in the ClusterServingRuntime resolves to a digest
  reference (`@sha256:` format) on deployed clusters
- The digest matches the corresponding entry in the CSV
  `relatedImages` section
- The template source uses `$(mlserver-cuda-image)` variable

**Notes**: To be filled later in the process.
