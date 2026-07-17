---
test_case_id: TC-AIRGAP-002
source_key: RHAISTRAT-1868
priority: P2
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-AIRGAP-002: GPU runtime in air-gapped env with mirrored image and pre-staged model

**Objective**: Verify that the GPU MLServer CUDA container image can
be mirrored to a disconnected registry and a GPU-accelerated
InferenceService deployed successfully from that registry with the
ONNX model pre-staged to in-cluster storage, confirming full
air-gapped GPU support.

**Preconditions**:
- Disconnected OCP 4.20+ cluster with GPU nodes (NVIDIA GPU
  Operator 12.9+)
- MLServer CUDA image (`$(mlserver-cuda-image)`) mirrored to
  disconnected registry via `oc mirror` or `skopeo copy`
- ResNet-50 ONNX model pre-staged to in-cluster S3-compatible object
  store or PVC (no external network access required at inference time)
- `mlserver-cuda-runtime-template` in `redhat-ods-applications`
  namespace, referencing the mirrored image
- HardwareProfile `nvidia-gpu-a100` (API group
  `infrastructure.opendatahub.io/v1`) applied

**Test Steps**:
1. Mirror the CUDA image to the disconnected registry:
   ```bash
   skopeo copy \
     docker://<source-registry>/mlserver-cuda@sha256:<digest> \
     docker://<disconnected-registry>/mlserver-cuda@sha256:<digest>
   ```
2. Verify the mirrored image is accessible:
   ```bash
   oc image info \
     <disconnected-registry>/mlserver-cuda@sha256:<digest>
   ```
3. Verify the ONNX model is pre-staged to in-cluster storage.
4. Deploy an InferenceService using the GPU runtime:
   ```bash
   cat <<EOF | oc apply -f -
   apiVersion: serving.kserve.io/v1beta1
   kind: InferenceService
   metadata:
     name: resnet-gpu
     annotations:
       serving.kserve.io/deploymentMode: RawDeployment
   spec:
     predictor:
       model:
         modelFormat:
           name: onnx
           version: "1"
         runtime: mlserver-cuda-runtime
         storageUri: s3://models/resnet-50-onnx/
   EOF
   ```
5. Wait for the InferenceService to reach `Ready`:
   ```bash
   oc wait inferenceservice/resnet-gpu --for=condition=Ready \
     --timeout=300s
   ```
6. Verify the image was pulled from the disconnected registry:
   ```bash
   oc get pod \
     -l serving.kserve.io/inferenceservice=resnet-gpu \
     -o jsonpath='{.items[0].status.containerStatuses[?(@.name=="kserve-container")].imageID}'
   ```
7. Verify no external registry pull attempts occurred:
   ```bash
   oc get events --field-selector reason=Pulling \
     | grep -v "<disconnected-registry>"
   ```
8. Send a V2 inference request:
   ```bash
   URL=$(oc get inferenceservice resnet-gpu \
     -o jsonpath='{.status.url}')
   curl -s -X POST "${URL}/v2/models/resnet-gpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json
   ```

**Expected Results**:
- GPU InferenceService reaches `Ready` state in the disconnected
  environment
- Pod `kserve-container` imageID references the disconnected registry
- No external registry pull attempts occur
- GPU inference returns HTTP `200` with correct predictions via V2
  REST protocol on port 8080
- Pod is scheduled on a GPU node with `nvidia.com/gpu` resource
  allocated

**Notes**: To be filled later in the process.
