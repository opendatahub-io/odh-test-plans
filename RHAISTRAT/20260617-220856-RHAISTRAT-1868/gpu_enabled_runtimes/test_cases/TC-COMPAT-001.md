---
test_case_id: TC-COMPAT-001
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-COMPAT-001: CPU MLServer deployment and inference baseline

**Objective**: Verify that the existing CPU MLServer image and
`mlserver-runtime` ServingRuntime deploy successfully, reach
readiness, and serve inference requests via KServe V2 protocol
using a ResNet-50 model, confirming backwards compatibility after
GPU runtime addition.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-runtime` ServingRuntime present on the cluster
- ResNet-50 ONNX model available in S3-compatible storage
- Valid AWS/S3 credentials configured for model access
- `namespace-admin` access

**Test Steps**:

1. Create a CPU ServingRuntime from the MLServer template:

   ```python
   pytestmark = pytest.mark.usefixtures("valid_aws_config")

   runtime = ServingRuntimeFromTemplate(
       template_name=RuntimeTemplates.MLSERVER,
   )
   ```

2. Deploy an InferenceService using the CPU runtime with an
   external route:

   ```python
   isvc = create_isvc(
       model_service_account=mlserver_model_service_account,
       runtime=runtime,
       model_format="onnx",
       storage_uri="s3://models/resnet-50-onnx/",
       external_route=True,
   )
   ```

3. Verify the InferenceService reaches `Ready` state and the
   predictor pod is running. Confirm readiness via health check:

   ```python
   pods = get_pods_by_isvc_label(client=client, isvc=isvc)
   assert pods, "Expected at least one predictor pod"
   assert pods[0].instance.status.phase == "Running"
   ```

4. Retrieve the exposed URL and send an inference request.
   Validate the response structure without asserting specific
   class indices:

   ```python
   url = get_exposed_isvc_url(isvc=isvc)
   response = send_rest_request(url=url, input_data=input_data)
   validate_deterministic_snapshot(response=response)
   ```

**Expected Results**:

- InferenceService reaches `Ready` condition within the default
  timeout
- Predictor pod is in `Running` state
- Inference response returns HTTP `200` with a valid KServe V2
  output structure (verified by `validate_deterministic_snapshot`)
- No specific class index is asserted -- only response structure
  is validated
- Pod does not request or consume `nvidia.com/gpu` resources

**Notes**: This test absorbs the former TC-COMPAT-002 (readiness
check) and TC-COMPAT-003 (baseline inference) into a single
end-to-end validation. Uses ResNet-50 model to avoid duplicating
coverage with the existing `TestMLServerModels` iris test.
