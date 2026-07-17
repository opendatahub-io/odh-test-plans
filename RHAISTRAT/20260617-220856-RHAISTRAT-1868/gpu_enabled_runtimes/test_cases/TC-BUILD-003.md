---
test_case_id: TC-BUILD-003
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-BUILD-003: CPU image unchanged -- no CUDA libraries present

**Objective**: Verify that the existing CPU MLServer container image
does not contain CUDA libraries, confirming the two-image architecture
keeps the CUDA payload off CPU-only clusters.

**Preconditions**:

- Access to the CPU MLServer container image digest (from CSV
  `relatedImages` or `mlserver-onnx` template)
- `podman` available on the test machine

**Test Steps**:

1. Extract the CPU image reference from the CPU ClusterServingRuntime:

   ```bash
   CPU_IMAGE=$(oc get clusterservingruntime mlserver-onnx \
     -o jsonpath='{.spec.containers[?(@.name=="kserve-container")].image}')
   echo "$CPU_IMAGE"
   ```

2. Check the CPU image for CUDA shared libraries:

   ```bash
   podman run --rm --entrypoint="" "${CPU_IMAGE}" \
     find /usr -name "libcuda*" -o -name "libcudart*" \
     -o -name "libcudnn*" 2>/dev/null
   ```

3. Verify `onnxruntime-gpu` is NOT installed:

   ```bash
   podman run --rm --entrypoint="" "${CPU_IMAGE}" \
     pip show onnxruntime-gpu 2>&1
   ```

4. Verify standard `onnxruntime` (CPU) is installed:

   ```bash
   podman run --rm --entrypoint="" "${CPU_IMAGE}" \
     pip show onnxruntime
   ```

**Expected Results**:

- No CUDA shared libraries found in the CPU image (empty output from
  `find`)
- `pip show onnxruntime-gpu` returns "Package(s) not found" or
  equivalent error
- `pip show onnxruntime` (CPU variant) is installed and reports a
  version
- The CPU image has not been modified by the GPU runtime addition

**Notes**: To be filled later in the process.
