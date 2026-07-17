---
test_case_id: TC-INFER-005
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-INFER-005: CUDA execution provider initialization in pod logs

**Objective**: Verify that the GPU MLServer pod logs show successful
initialization of the CUDA execution provider (baked into the
`$(mlserver-cuda-image)` at build time) via `onnxruntime-gpu`, with
no fallback to CPU provider.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator 12.9+
- InferenceService `resnet-gpu` deployed with `mlserver-cuda-runtime`
  and Ready (TC-DEPLOY-002)

**Test Steps**:
1. Identify the predictor pod:
   ```bash
   POD=$(oc get pod \
     -l serving.kserve.io/inferenceservice=resnet-gpu \
     -o jsonpath='{.items[0].metadata.name}')
   ```
2. Search pod logs for execution provider registration:
   ```bash
   oc logs "$POD" -c kserve-container \
     | grep -iE "(CUDAExecutionProvider|TensorrtExecutionProvider|execution.*provider)"
   ```
3. Search pod logs for CUDA initialization messages:
   ```bash
   oc logs "$POD" -c kserve-container | grep -i "cuda"
   ```
4. Verify no fallback-to-CPU warnings appear in logs:
   ```bash
   oc logs "$POD" -c kserve-container \
     | grep -iE "(fallback.*cpu|CPUExecutionProvider.*default)"
   ```

**Expected Results**:
- Logs contain at least one reference to `CUDAExecutionProvider`
  being registered or activated
- No log messages indicate fallback to `CPUExecutionProvider` as
  the primary provider
- No CUDA initialization errors (e.g., "CUDA driver version is
  insufficient" or "no CUDA-capable device")

**Notes**: To be filled later in the process.
