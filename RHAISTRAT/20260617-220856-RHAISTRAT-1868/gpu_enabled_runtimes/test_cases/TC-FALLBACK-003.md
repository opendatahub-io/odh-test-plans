---
test_case_id: TC-FALLBACK-003
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-FALLBACK-003: No inference accessible during Pending state

**Objective**: Verify that when a `mlserver-cuda-runtime` pod is in
Pending state (due to missing GPU resources), no inference endpoint
becomes accessible — confirming there is no silent CPU fallback path.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA
- GPU InferenceService deployed without GPU HardwareProfile, pod in
  Pending state (TC-FALLBACK-001)

**Test Steps**:

1. Confirm the pod is still in Pending state:

   ```bash
   oc get pod \
     -l serving.kserve.io/inferenceservice=resnet-gpu-nohw \
     -o jsonpath='{.items[0].status.phase}'
   ```

2. Attempt to retrieve the InferenceService URL:

   ```bash
   URL=$(oc get inferenceservice resnet-gpu-nohw \
     -o jsonpath='{.status.url}')
   ```

3. If a URL is returned, attempt an inference request:

   ```bash
   curl -s -o /dev/null -w "%{http_code}" -m 10 \
     "${URL}/v2/models/resnet-gpu-nohw/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json
   ```

4. Verify the InferenceService status conditions:

   ```bash
   oc get inferenceservice resnet-gpu-nohw \
     -o jsonpath='{.status.conditions}'
   ```

**Expected Results**:

- InferenceService `Ready` condition is `False`
- Either no URL is assigned, or any request to the URL returns a
  non-`200` status (e.g., `503` Service Unavailable or connection
  refused)
- No `200` response with inference output is returned — confirming
  the model is not silently running on CPU

**Notes**: To be filled later in the process.
