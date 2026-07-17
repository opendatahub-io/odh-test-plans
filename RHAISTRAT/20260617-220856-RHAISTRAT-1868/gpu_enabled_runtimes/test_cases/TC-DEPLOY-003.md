---
test_case_id: TC-DEPLOY-003
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-DEPLOY-003: Verify GPU container initializes CUDA execution provider

**Objective**: Verify that the GPU MLServer container image
(`$(mlserver-cuda-image)`) starts successfully, initializes the CUDA
execution provider (baked into image at build time), and reports GPU
device availability.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator 12.9+
- InferenceService deployed with `mlserver-cuda-runtime` and GPU
  HardwareProfile (TC-DEPLOY-002 passed)
- Predictor pod is in `Running` state on a GPU node

**Test Steps**:

1. Identify the predictor pod:

   ```bash
   POD=$(oc get pod \
     -l serving.kserve.io/inferenceservice=resnet-gpu \
     -o jsonpath='{.items[0].metadata.name}')
   ```

2. Check pod logs for CUDA execution provider initialization:

   ```bash
   oc logs "$POD" -c kserve-container | grep -i "cuda"
   ```

3. Check pod logs for onnxruntime execution provider loading:

   ```bash
   oc logs "$POD" -c kserve-container \
     | grep -i "execution.*provider"
   ```

4. Verify `nvidia-smi` is accessible inside the container:

   ```bash
   oc exec "$POD" -c kserve-container -- nvidia-smi
   ```

5. Verify the security context is enforced (non-root, capabilities
   dropped):

   ```bash
   oc get pod "$POD" \
     -o jsonpath='{.spec.containers[?(@.name=="kserve-container")].securityContext}'
   ```

**Expected Results**:

- Pod logs contain messages indicating CUDA execution provider
  initialization (e.g., `CUDAExecutionProvider` in provider list)
- Pod logs show `onnxruntime` loading GPU execution providers
- `nvidia-smi` returns GPU device information including driver version,
  GPU model, and memory usage
- No CUDA-related error messages in pod logs
- Security context shows `runAsNonRoot: true`,
  `allowPrivilegeEscalation: false`, and `drop: ["ALL"]` capabilities

**Notes**: To be filled later in the process.
