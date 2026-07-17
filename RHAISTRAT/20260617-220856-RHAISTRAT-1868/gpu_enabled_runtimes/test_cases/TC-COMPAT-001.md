---
test_case_id: TC-COMPAT-001
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-COMPAT-001: CPU MLServer deploys and serves inference

**Objective**: Verify that the existing CPU MLServer image and
`mlserver-onnx` ClusterServingRuntime deploy successfully and serve
inference requests via KServe V2 protocol, confirming backwards
compatibility after GPU runtime addition.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-onnx` ClusterServingRuntime present on the cluster
- ResNet-50 ONNX model available in S3-compatible storage
- `namespace-admin` access

**Test Steps**:

1. Deploy an InferenceService using the CPU runtime:

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

2. Wait for the InferenceService to reach `Ready`:

   ```bash
   oc wait inferenceservice/resnet-cpu --for=condition=Ready \
     --timeout=300s
   ```

3. Send an inference request to the CPU endpoint:

   ```bash
   URL=$(oc get inferenceservice resnet-cpu \
     -o jsonpath='{.status.url}')
   curl -s -X POST "${URL}/v2/models/resnet-cpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json
   ```

4. Verify the response contains a valid prediction.

**Expected Results**:

- InferenceService reaches `Ready` condition within 300 seconds
- Predictor pod is in `Running` state
- Inference response returns HTTP `200` with a valid KServe V2
  output containing the expected class prediction
- Pod does not request or consume `nvidia.com/gpu` resources

**Notes**: To be filled later in the process.
