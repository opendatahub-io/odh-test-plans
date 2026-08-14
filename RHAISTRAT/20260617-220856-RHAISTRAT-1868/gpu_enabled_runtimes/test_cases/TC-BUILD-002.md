---
test_case_id: TC-BUILD-002
source_key: RHAISTRAT-1868
priority: P1
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-BUILD-002: Two-image architecture separation

**Objective**: Verify that the GPU and CPU MLServer container
images have distinct contents -- the GPU image includes CUDA
runtime libraries and `onnxruntime-gpu`, while the CPU image
does not. All checks run via `pod.execute()` on live containers.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- `mlserver-runtime` and `mlserver-cuda-runtime` ServingRuntime
  templates present in `redhat-ods-applications`
- ResNet-50 ONNX model available in S3-compatible storage
- Valid AWS/S3 credentials configured for model access
- GPU node(s) available on the cluster for the GPU ISVC

**Test Steps**:

1. Retrieve image references from both templates:

   ```python
   pytestmark = pytest.mark.usefixtures("valid_aws_config")

   from utilities.serving_runtime import (
       get_runtime_image_from_template,
   )

   gpu_image = get_runtime_image_from_template(
       client=client,
       template_name="mlserver-cuda-runtime",
       namespace="redhat-ods-applications",
   )
   cpu_image = get_runtime_image_from_template(
       client=client,
       template_name="mlserver-runtime",
       namespace="redhat-ods-applications",
   )
   ```

2. Deploy both GPU and CPU InferenceServices and retrieve their
   pods:

   ```python
   isvc_gpu = create_isvc(
       model_service_account=mlserver_model_service_account,
       runtime=gpu_runtime,
       model_format="onnx",
       storage_uri="s3://models/resnet-50-onnx/",
       external_route=True,
   )
   isvc_cpu = create_isvc(
       model_service_account=mlserver_model_service_account,
       runtime=cpu_runtime,
       model_format="onnx",
       storage_uri="s3://models/resnet-50-onnx/",
       external_route=True,
   )
   gpu_pods = get_pods_by_isvc_label(client=client, isvc=isvc_gpu)
   cpu_pods = get_pods_by_isvc_label(client=client, isvc=isvc_cpu)
   gpu_pod = gpu_pods[0]
   cpu_pod = cpu_pods[0]
   ```

3. GPU pod -- verify CUDA runtime libraries are present. Check
   for `libcudart*` (CUDA runtime toolkit), not `libcuda*`
   (NVIDIA driver library, host-provided):

   ```python
   result = gpu_pod.execute(
       command=[
           "find", "/usr/local/cuda/lib64",
           "-name", "libcudart*",
       ],
   )
   assert "libcudart" in result, (
       "GPU image must contain CUDA runtime libraries"
   )
   ```

4. GPU pod -- verify `onnxruntime-gpu` is installed:

   ```python
   result = gpu_pod.execute(
       command=["pip", "show", "onnxruntime-gpu"],
   )
   assert "onnxruntime-gpu" in result
   ```

5. GPU pod -- verify ONNX Runtime available providers include
   `CUDAExecutionProvider`:

   ```python
   result = gpu_pod.execute(
       command=[
           "python3", "-c",
           "import onnxruntime; "
           "print(onnxruntime.get_available_providers())",
       ],
   )
   assert "CUDAExecutionProvider" in result
   ```

6. CPU pod -- verify the same checks show the absence of CUDA
   components:

   ```python
   # libcudart should NOT be found in the CPU image
   result = cpu_pod.execute(
       command=[
           "find", "/usr/local/cuda/lib64",
           "-name", "libcudart*",
       ],
   )
   assert "libcudart" not in result, (
       "CPU image must NOT contain CUDA runtime libraries"
   )

   # onnxruntime (not gpu) should be installed
   result = cpu_pod.execute(
       command=["pip", "show", "onnxruntime"],
   )
   assert "onnxruntime" in result
   result = cpu_pod.execute(
       command=["pip", "show", "onnxruntime-gpu"],
   )
   assert "onnxruntime-gpu" not in result or (
       "not found" in result.lower()
   )

   # CUDAExecutionProvider should NOT be available
   result = cpu_pod.execute(
       command=[
           "python3", "-c",
           "import onnxruntime; "
           "print(onnxruntime.get_available_providers())",
       ],
   )
   assert "CUDAExecutionProvider" not in result
   ```

7. Distribution-specific base image check (parameterize for
   RHOAI vs ODH):

   ```python
   result = gpu_pod.execute(
       command=["cat", "/etc/os-release"],
   )
   # Assert distribution-specific base image markers
   # For RHOAI: expect UBI-based or aipcc/cuda-based image
   # For ODH: expect upstream base image
   ```

**Expected Results**:

- GPU image contains `libcudart*` shared libraries in
  `/usr/local/cuda/lib64`
- GPU image has `onnxruntime-gpu` installed and reports
  `CUDAExecutionProvider` in available providers
- CPU image does NOT contain `libcudart*` libraries
- CPU image has `onnxruntime` (not `-gpu`) installed and does
  NOT report `CUDAExecutionProvider`
- Base image lineage is appropriate for the distribution
  (RHOAI vs ODH)
- All checks performed via `pod.execute()` on running
  containers -- no `podman` or `skopeo` required

**Notes**: This test absorbs the former TC-BUILD-003 (CPU image
lacks CUDA). Important distinction: `libcudart*` is the CUDA
runtime toolkit library (baked into the GPU container image),
while `libcuda*` is the NVIDIA driver library (provided by the
host via device plugin, NOT expected inside the container).
Step 7 should be parameterized based on the product distribution.
