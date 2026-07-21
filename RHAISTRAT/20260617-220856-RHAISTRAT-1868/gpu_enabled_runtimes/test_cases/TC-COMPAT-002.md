---
test_case_id: TC-COMPAT-002
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: both
---
# TC-COMPAT-002: CPU runtime unaffected after GPU runtime added

**Objective**: Verify that instantiating a GPU
`mlserver-cuda-runtime` ServingRuntime in the same namespace as
an existing CPU-based InferenceService does not affect the
behavior or availability of the CPU InferenceService. This tests
namespace-level isolation between CPU and GPU runtimes.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-runtime` and `mlserver-cuda-runtime` templates
  pre-installed on the cluster
- ResNet-50 ONNX model available in S3-compatible storage
- Valid AWS/S3 credentials configured for model access
- `namespace-admin` access

**Test Steps**:

1. Deploy a CPU InferenceService and establish a baseline
   inference response:

   ```python
   pytestmark = pytest.mark.usefixtures("valid_aws_config")

   isvc_cpu = create_isvc(
       model_service_account=mlserver_model_service_account,
       runtime=cpu_runtime,
       model_format="onnx",
       storage_uri="s3://models/resnet-50-onnx/",
       external_route=True,
   )
   url = get_exposed_isvc_url(isvc=isvc_cpu)
   baseline = send_rest_request(url=url, input_data=input_data)
   validate_deterministic_snapshot(response=baseline)
   ```

2. Instantiate a GPU ServingRuntime from template in the same
   namespace:

   ```python
   gpu_runtime = ServingRuntimeFromTemplate(
       template_name=RuntimeTemplates.MLSERVER_CUDA,
   )
   ```

3. Verify the CPU InferenceService is still `Ready` and repeat
   the inference request. Confirm the response structure is
   unchanged:

   ```python
   assert isvc_cpu.instance.status.conditions
   ready_condition = next(
       c for c in isvc_cpu.instance.status.conditions
       if c.type == "Ready"
   )
   assert ready_condition.status == "True"

   after = send_rest_request(url=url, input_data=input_data)
   validate_deterministic_snapshot(response=after)
   ```

**Expected Results**:

- CPU InferenceService remains in `Ready` state after GPU
  runtime instantiation in the same namespace
- CPU inference response returns HTTP `200` with a valid
  KServe V2 output structure matching the baseline (verified
  by `validate_deterministic_snapshot`)
- No specific class index is asserted -- only response
  structure is validated
- CPU predictor pod was not restarted by the GPU runtime
  addition
- No errors in odh-model-controller logs related to the CPU
  runtime

**Notes**: The GPU template is always pre-installed by RHOAI.
This test validates namespace-level isolation rather than
template-level addition. Replaces former TC-COMPAT-004.
