---
test_case_id: TC-INFER-002
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-INFER-002: Numerical consistency between GPU and CPU inference

**Objective**: Verify that GPU inference via `mlserver-cuda-runtime`
produces numerically consistent results compared to CPU inference via
`mlserver-onnx` for the same ResNet-50 model and identical inputs,
within an acceptable floating-point tolerance.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator 12.9+
- GPU InferenceService `resnet-gpu` deployed with
  `mlserver-cuda-runtime` and Ready (TC-DEPLOY-002)
- CPU InferenceService `resnet-cpu` deployed with the existing
  `mlserver-onnx` runtime and Ready, using the same ResNet-50 ONNX
  model artifact

**Test Steps**:

1. Obtain both InferenceService URLs:

   ```bash
   GPU_URL=$(oc get inferenceservice resnet-gpu \
     -o jsonpath='{.status.url}')
   CPU_URL=$(oc get inferenceservice resnet-cpu \
     -o jsonpath='{.status.url}')
   ```

2. Send the same inference request to the GPU runtime:

   ```bash
   GPU_RESP=$(curl -s -X POST \
     "${GPU_URL}/v2/models/resnet-gpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json)
   ```

3. Send the identical request to the CPU runtime:

   ```bash
   CPU_RESP=$(curl -s -X POST \
     "${CPU_URL}/v2/models/resnet-cpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json)
   ```

4. Compare the output tensors element-wise. Calculate the maximum
   absolute difference between corresponding output values.
5. Verify the top-K predicted classes are identical between GPU and
   CPU outputs.

**Expected Results**:

- Both responses return HTTP `200` with valid KServe V2 output
- The top-1 predicted class index is identical between GPU and CPU
- The maximum absolute difference between corresponding output
  values is less than `1e-4` (FP32 tolerance for ONNX Runtime
  CPU vs CUDA provider)
- Output tensor shapes are identical

**Notes**: To be filled later in the process.
