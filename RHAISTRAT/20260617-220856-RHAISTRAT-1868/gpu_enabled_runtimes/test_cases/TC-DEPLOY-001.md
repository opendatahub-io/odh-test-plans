---
test_case_id: TC-DEPLOY-001
source_key: RHAISTRAT-1868
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-07-21"
upgrade_phase: post
---
# TC-DEPLOY-001: Verify GPU runtime template and instantiated ServingRuntime

**Objective**: Verify that `ServingRuntimeFromTemplate` using
`RuntimeTemplates.MLSERVER_CUDA` creates a ServingRuntime with
correct labels, annotations, container configuration, and
environment variables. Also verify that the CPU `MLSERVER` template
does NOT include GPU-specific accelerator annotations.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA installed
- KServe and odh-model-controller deployed
- `mlserver-cuda-runtime-template` present in
  `redhat-ods-applications` namespace

**Markers**:

```python
pytestmark = pytest.mark.usefixtures("valid_aws_config")

@pytest.mark.model_server_gpu
@skip_if_no_supported_accelerator_type
```

**Test Steps**:

1. Create the GPU ServingRuntime from the template (no
   InferenceService needed):

   ```python
   gpu_runtime = ServingRuntimeFromTemplate(
       template_name=RuntimeTemplates.MLSERVER_CUDA
   )
   ```

2. Verify the `opendatahub.io/dashboard: "true"` label is present:

   ```python
   labels = gpu_runtime.instance.metadata.labels
   assert labels["opendatahub.io/dashboard"] == "true"
   ```

3. Verify the `opendatahub.io/recommended-accelerators` annotation
   contains `nvidia.com/gpu`:

   ```python
   annotations = gpu_runtime.instance.metadata.annotations
   accel = annotations["opendatahub.io/recommended-accelerators"]
   assert "nvidia.com/gpu" in accel
   ```

4. Verify the container image reference uses the CUDA image
   variable:

   ```python
   image = (
       gpu_runtime.instance.spec.containers[0].image
   )
   assert "mlserver-cuda-image" in image or "cuda" in image.lower()
   ```

5. Verify supported model format is ONNX v1 with single-model
   serving:

   ```python
   formats = gpu_runtime.instance.spec.supportedModelFormats
   onnx_fmt = [f for f in formats if f.name == "onnx"][0]
   assert onnx_fmt.version == "1"
   assert onnx_fmt.multiModel is False
   ```

6. Verify the runtime uses HTTP port 8080:

   ```python
   ports = gpu_runtime.instance.spec.containers[0].ports
   assert any(p.containerPort == 8080 for p in ports)
   ```

7. Verify `MLSERVER_MODEL_IMPLEMENTATION` env var uses a template
   variable:

   ```python
   envs = gpu_runtime.instance.spec.containers[0].env
   impl_env = [
       e for e in envs
       if e.name == "MLSERVER_MODEL_IMPLEMENTATION"
   ][0]
   assert "{{.Labels.modelClass}}" in impl_env.value
   ```

8. Verify `MLSERVER_MODEL_ONNX_PROVIDERS` env var is set on the
   CUDA image only:

   ```python
   provider_envs = [
       e for e in envs
       if e.name == "MLSERVER_MODEL_ONNX_PROVIDERS"
   ]
   assert len(provider_envs) == 1
   ```

9. Instantiate a CPU ServingRuntime and verify it does NOT have the
   `recommended-accelerators` annotation (absorbed from old
   TC-DEPLOY-005):

   ```python
   cpu_runtime = ServingRuntimeFromTemplate(
       template_name=RuntimeTemplates.MLSERVER
   )
   cpu_annotations = cpu_runtime.instance.metadata.annotations or {}
   assert (
       "opendatahub.io/recommended-accelerators"
       not in cpu_annotations
   )
   ```

**Expected Results**:

- GPU ServingRuntime is created without error conditions
- Label `opendatahub.io/dashboard` equals `"true"`
- Annotation `opendatahub.io/recommended-accelerators` contains
  `["nvidia.com/gpu"]`
- Container image reference includes `mlserver-cuda-image` or
  `cuda`
- Supported model format is `onnx` version `"1"` with
  `multiModel: false`
- Container port is `8080`
- `MLSERVER_MODEL_IMPLEMENTATION` uses `{{.Labels.modelClass}}`
  template variable
- `MLSERVER_MODEL_ONNX_PROVIDERS` env var is present on the CUDA
  runtime
- CPU `MLSERVER` runtime does NOT have
  `opendatahub.io/recommended-accelerators` annotation

**Notes**: Absorbs old TC-DEPLOY-005 (GPU annotation check + CPU
negative assertion).
