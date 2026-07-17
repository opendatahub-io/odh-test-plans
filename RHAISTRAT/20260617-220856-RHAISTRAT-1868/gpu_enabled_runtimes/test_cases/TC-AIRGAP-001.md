---
test_case_id: TC-AIRGAP-001
source_key: RHAISTRAT-1868
priority: P2
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-AIRGAP-001: CPU-only air-gapped deployment works without GPU image

**Objective**: Verify that a CPU-only deployment in an air-gapped
(disconnected) environment functions without any reference to or
dependency on the GPU container image (`$(mlserver-cuda-image)`),
confirming the two-image architecture keeps the CUDA payload off
CPU-only clusters.

**Preconditions**:
- Disconnected OCP 4.20+ cluster with mirrored CPU MLServer image
- GPU MLServer CUDA image NOT mirrored to the disconnected registry
- `mlserver-onnx` ClusterServingRuntime present
- ResNet-50 ONNX model pre-staged to in-cluster S3-compatible storage

**Test Steps**:
1. Verify the disconnected registry contains only the CPU MLServer
   image and NOT the CUDA image:
   ```bash
   oc image info \
     <disconnected-registry>/mlserver-onnx:<tag> 2>&1
   oc image info \
     <disconnected-registry>/mlserver-cuda:<tag> 2>&1
   ```
2. Deploy an InferenceService using the CPU runtime:
   ```bash
   cat <<EOF | oc apply -f -
   apiVersion: serving.kserve.io/v1beta1
   kind: InferenceService
   metadata:
     name: resnet-cpu
     annotations:
       serving.kserve.io/deploymentMode: RawDeployment
   spec:
     predictor:
       model:
         modelFormat:
           name: onnx
           version: "1"
         runtime: mlserver-onnx
         storageUri: s3://models/resnet-50-onnx/
   EOF
   ```
3. Wait for the InferenceService to reach `Ready`:
   ```bash
   oc wait inferenceservice/resnet-cpu --for=condition=Ready \
     --timeout=300s
   ```
4. Verify no external registry pulls occurred:
   ```bash
   oc get events --field-selector reason=Pulling \
     | grep -v "<disconnected-registry>"
   ```
5. Send a V2 inference request to confirm the model serves correctly:
   ```bash
   URL=$(oc get inferenceservice resnet-cpu \
     -o jsonpath='{.status.url}')
   curl -s -o /dev/null -w "%{http_code}" \
     -X POST "${URL}/v2/models/resnet-cpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json
   ```

**Expected Results**:
- CPU InferenceService reaches `Ready` without requiring the CUDA
  GPU image
- No image pull attempts target external (non-mirrored) registries
- Inference request returns HTTP `200` with correct predictions via
  V2 REST protocol on port 8080
- No pod warnings or errors reference the CUDA image or
  `mlserver-cuda-runtime`

**Notes**: To be filled later in the process.
