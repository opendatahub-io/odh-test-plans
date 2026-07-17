---
test_case_id: TC-BUILD-002
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-BUILD-002: GPU image contains CUDA libraries

**Objective**: Verify that the pre-built GPU MLServer container image
(`$(mlserver-cuda-image)`) is built on the `aipcc/cuda` base and
contains the required CUDA and `onnxruntime-gpu` libraries for CUDA
execution provider support (baked in at build time).

**Preconditions**:

- Access to the pre-built GPU MLServer container image digest (from
  CSV `relatedImages` or `mlserver-cuda-runtime` template)
- `skopeo` and `podman` available on the test machine

**Test Steps**:

1. Extract the GPU image reference from the ClusterServingRuntime:

   ```bash
   GPU_IMAGE=$(oc get clusterservingruntime mlserver-cuda-runtime \
     -o jsonpath='{.spec.containers[?(@.name=="kserve-container")].image}')
   echo "$GPU_IMAGE"
   ```

2. Inspect the GPU container image metadata:

   ```bash
   skopeo inspect --config "docker://${GPU_IMAGE}" | jq .
   ```

3. Verify the image contains CUDA shared libraries:

   ```bash
   podman run --rm --entrypoint="" "${GPU_IMAGE}" \
     find /usr -name "libcuda*" -o -name "libcudart*" \
     -o -name "libcudnn*"
   ```

4. Verify `onnxruntime-gpu` Python package is installed:

   ```bash
   podman run --rm --entrypoint="" "${GPU_IMAGE}" \
     pip show onnxruntime-gpu
   ```

5. Check the base image lineage includes `aipcc/cuda`:

   ```bash
   skopeo inspect "docker://${GPU_IMAGE}" | jq '.Labels'
   ```

**Expected Results**:

- Image contains CUDA shared libraries (`libcudart.so`,
  `libcudnn.so` or equivalent)
- `onnxruntime-gpu` package is installed and reports a version
- Image labels or layer history reference the `aipcc/cuda` base image
- The CUDA EP is baked into the image (not user-configurable)

**Notes**: To be filled later in the process.
