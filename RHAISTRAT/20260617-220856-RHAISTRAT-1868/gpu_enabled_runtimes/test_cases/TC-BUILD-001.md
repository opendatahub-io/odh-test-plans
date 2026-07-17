---
test_case_id: TC-BUILD-001
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-BUILD-001: CSV relatedImages contains both CPU and GPU image digests

**Objective**: Verify that the RHOAI operator CSV (ClusterServiceVersion)
`relatedImages` section includes both the CPU and GPU MLServer
container image references using digest format, ensuring OLM can
mirror both images for disconnected deployments.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- RHOAI operator CSV accessible in `openshift-operators` namespace

**Test Steps**:

1. Retrieve the CSV for the RHOAI operator:

   ```bash
   CSV=$(oc get csv -n openshift-operators \
     -l operators.coreos.com/rhods-operator.openshift-operators \
     -o jsonpath='{.items[0].metadata.name}')
   oc get csv "$CSV" -n openshift-operators \
     -o jsonpath='{.spec.relatedImages}' | jq .
   ```

2. Verify a CPU MLServer image entry exists in `relatedImages`:

   ```bash
   oc get csv "$CSV" -n openshift-operators \
     -o jsonpath='{.spec.relatedImages}' \
     | jq '.[] | select(.name | contains("mlserver"))'
   ```

3. Verify a GPU MLServer CUDA image entry exists in `relatedImages`
   referencing `$(mlserver-cuda-image)`:

   ```bash
   oc get csv "$CSV" -n openshift-operators \
     -o jsonpath='{.spec.relatedImages}' \
     | jq '.[] | select(.name | contains("mlserver-cuda"))'
   ```

4. Confirm both entries use image digests (not tags):

   ```bash
   oc get csv "$CSV" -n openshift-operators \
     -o jsonpath='{.spec.relatedImages}' \
     | jq '.[] | select(.name | contains("mlserver"))' \
     | grep -c "@sha256:"
   ```

**Expected Results**:

- `relatedImages` contains at least two entries with names referencing
  MLServer — one for CPU, one for GPU (CUDA)
- Both image references use `@sha256:` digest format (not `:tag`)
- The GPU image reference corresponds to the `mlserver-cuda-image`
  built on the `aipcc/cuda` base
- The CPU image reference matches the existing unchanged CPU image

**Notes**: To be filled later in the process.
