---
feature: gpu_enabled_runtimes
source_key: RHAISTRAT-1868
status: Open
gap_count: 8
last_updated: '2026-07-22'
---
# Gaps — GPU Enabled Runtimes for Predictive Machine Learning

## Resolved Gaps

The following gaps were resolved by the context document
(RHAISTRAT-1868-context.md) incorporated on 2026-07-17:

### Scope & Endpoints (5 of 7 resolved)

- ~~model-settings.json GPU configuration schema not defined~~ —
  **Resolved**: follows upstream MLServer settings schema (ModelSettings,
  ModelParameters, Settings from opendatahub-io/MLServer repo)
- ~~Multi-GPU and GPU sharing support unclear~~ — **Resolved**: explicitly
  out of scope for this release (multi-GPU, MIG, time-slicing)
- ~~KServe protocol version unconfirmed~~ — **Resolved**: V2 only
  confirmed; all endpoints follow `/v2/...` path convention
- ~~Dynamic batching configuration and limits not specified~~ —
  **Resolved**: explicitly out of scope for this release
- ~~Multi-stage pipeline architecture not detailed~~ — **Resolved**:
  standard KServe RAW deployment on GPU nodes, no custom CRD or chaining

### Test Strategy & Risks (3 of 3 resolved)

- ~~No ADR provided for detailed technical design~~ — **Resolved**:
  context document provides CUDA EP configuration details (baked into
  image at build time), supported GPU hardware info, and full template
  specification
- ~~Supported GPU hardware and driver versions not specified~~ —
  **Resolved**: all modern NVIDIA GPUs supported, minimum driver
  requirements governed by NVIDIA GPU Operator 12.9
- ~~GPU ClusterServingRuntime template YAML not provided~~ —
  **Resolved**: complete template specification provided
  (`mlserver-cuda-runtime-template`, port 8080, V2 protocol,
  single-model, ONNX v1, probes, security context, resource defaults)

### Environment & Infrastructure (8 of 8 resolved)

- ~~OpenShift version requirement not specified~~ — **Resolved**: minimum
  4.20
- ~~RHOAI exact target version not confirmed~~ — **Resolved**: 3.5 GA
- ~~NVIDIA GPU Operator version requirement not specified~~ —
  **Resolved**: minimum 12.9
- ~~HardwareProfile CRD specification and sample YAML not provided~~ —
  **Resolved**: `infrastructure.opendatahub.io/v1` API group,
  `nvidia-gpu-a100` sample with full resource identifiers and tolerations
- ~~Feature gate or feature flag status unknown~~ — **Resolved**: not
  gated behind any feature flag, available by default
- ~~Specific ONNX model artifacts for test data not identified~~ —
  **Resolved**: ResNet-50 from onnx-community/resnet-50-ONNX, hosted on
  Red Hat-maintained S3 bucket
- ~~Konflux/Tekton GPU pipeline definition and triggers not provided~~ —
  **Resolved**: not required for current release; pre-built images
  referenced directly
- ~~gRPC support status for MLServer GPU image unclear~~ — **Resolved**:
  not in scope; REST/HTTP V2 only on port 8080

### Test Case Coverage (2 of 3 resolved)

- ~~No test cases for dynamic batching behavior~~ — **Resolved**: dynamic
  batching out of scope for this release
- ~~No test cases for gRPC inference~~ — **Resolved**: gRPC out of scope
  for this release

## Open Gaps

### Test Case Coverage

- **E2E coverage for P1 config/build components** — Dedicated E2E test
  cases were removed during PR consolidation (41→17 TCs). P1
  config/build components (CSV `relatedImages`, image architecture
  separation, template processing) are now covered by TC-BUILD-001 and
  TC-BUILD-002, which provide functional validation but not full E2E
  user journey coverage. No promotion criteria defined for elevating
  these to P0.
  Would be resolved by: defining concrete promotion criteria or
  accepting current P1 coverage as sufficient

### Scope & Endpoints

- **Performance acceptance criteria not quantified** — the strategy
  describes qualitative goals (predictable low-latency, strong
  scalability) but provides no concrete SLOs, latency targets,
  throughput benchmarks, or concurrency limits. Context document
  explicitly marks this as "TODO — needs follow-up."
  Would be resolved by: engineering design doc or ADR specifying
  concrete p50/p95/p99 latency targets in ms, throughput benchmarks
  in inferences/sec, and maximum concurrency limits

- **Air-gapped environment acceptance criteria incomplete** — the context
  document provides two high-level conditions (GPU image mirroring +
  model pre-staging) but does not define specific test scenarios or
  detailed acceptance criteria for air-gapped GPU deployment.
  Would be resolved by: feature refinement with detailed air-gapped
  test scenarios

### Test Environment

- **S3 bucket path for ResNet-50 reference model not specified** — Red
  Hat-maintained S3 bucket confirmed as hosting location but specific
  bucket name and path not documented.
  Would be resolved by: test data configuration doc or environment
  setup guide

- **Load testing tool selection unconfirmed** — Locust and k6 are
  recommended but final selection not confirmed for test execution.
  Would be resolved by: test infrastructure decision

### Test Strategy

- **ONNX Runtime version/opset compatibility matrix not defined** —
  specific ONNX opset version support and TensorRT optimization flags
  are determined by the bundled ONNX Runtime version, but no
  compatibility matrix exists.
  Would be resolved by: build specification or ADR documenting CUDA
  toolkit, cuDNN, ONNX Runtime versions and supported opset versions

- **ClusterPolicy customizations beyond GPU Operator 12.9 defaults not
  documented** — context document states no customizations required
  beyond defaults, but this has not been verified against test
  environment requirements.
  Would be resolved by: infrastructure doc or design clarification

### Test Case Coverage

- **Performance test thresholds not defined** — TC-PERF-001,
  TC-PERF-002, and TC-PERF-003 measure performance but cannot
  assert pass/fail against specific SLOs because acceptance criteria
  are not quantified.
  Would be resolved by: feature refinement providing concrete
  latency targets and throughput benchmarks
