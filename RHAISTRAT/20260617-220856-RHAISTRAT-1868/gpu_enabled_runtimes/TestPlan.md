---
feature: gpu_enabled_runtimes
source_key: RHAISTRAT-1868
source_type: strat
status: In Review
author: Model Runtimes
components:
- Model Runtimes
additional_docs:
- RHAISTRAT-1868-context.md
last_updated: '2026-07-22'
version: 1.3.1
reviewers: []
---
# GPU Enabled Runtimes Test Plan

**Model Runtimes – GPU-Accelerated Inference for Predictive ML**

**Strategy**: [RHAISTRAT-1868](https://issues.redhat.com/browse/RHAISTRAT-1868)

---

## 1. Executive Summary

### 1.1 Purpose

This test plan validates the enablement of NVIDIA GPU-accelerated inference
for predictive machine learning workloads in Red Hat OpenShift AI (RHOAI)
3.5 GA on OpenShift 4.20. The feature introduces a separate MLServer GPU
container image built on the `aipcc/cuda` base with `onnxruntime-gpu`,
paired with a new `mlserver-cuda-runtime` template. Testing ensures that
GPU inference works correctly, that the existing CPU runtime remains
unaffected, and that the two-image architecture prevents silent CPU
fallback while keeping CUDA payload off CPU-only and air-gapped clusters.

The feature targets performance-sensitive domains including computer vision
(multi-camera analytics, defect detection), speech and audio processing,
recommender systems, and scientific ML. GPU acceleration is provided via
the CUDA execution provider baked into the container image at build time.
Validation covers the full deployment lifecycle from GPU runtime template
installation through GPU inference under load.

### 1.2 Scope

#### In Scope (Model Runtimes Responsibilities)

- GPU MLServer container image build on `aipcc/cuda` base with
  `onnxruntime-gpu` and CUDA execution provider
- New `mlserver-cuda-runtime-template` in `redhat-ods-applications`
  namespace (creates `mlserver-cuda-runtime` ServingRuntime)
- HardwareProfile-based GPU allocation via `nvidia-gpu-a100`
  HardwareProfile CR (API group: `infrastructure.opendatahub.io/v1`)
- Dashboard auto-discovery of GPU runtime via `opendatahub.io/dashboard`
  label
- CUDA execution provider baked into container image at build time
  (overridable via `MLSERVER_MODEL_ONNX_PROVIDERS` env var or
  per-model `parameters.extra.providers` in `model-settings.json`)
- CSV `relatedImages` inclusion of both CPU and GPU image references
- Backwards compatibility of existing CPU MLServer image and template
- Silent CPU fallback detection (pod Running on CPU when GPU
  resources not requested, ORT falls back to CPUExecutionProvider)
- GPU inference correctness and numerical consistency with CPU results
- KServe V2 REST protocol only on port 8080 (no gRPC, no V1 protocol)
- Single-model serving only (multi-model: false)
- ONNX format version 1 support
- `kustomization.yaml` update with new GPU runtime template
- Air-gapped deployment validation for GPU runtime (mirrored GPU image
  - pre-staged model)
- Standard KServe RAW deployment on GPU nodes (standard Kubernetes
  scheduling)
- Security context: non-root, all capabilities dropped, privilege
  escalation disallowed
- Startup, readiness, and liveness probes per template spec

#### Out of Scope (Other Teams or Explicitly Excluded)

- OVMS NVIDIA CUDA support (deprecated as of RHOAI 2.25)
- Dashboard UI code changes (annotation-driven discovery is automatic)
- Platform/Operator team changes (HardwareProfile is GA and generic)
- vLLM or LLM-specific GPU runtime changes
- Multi-GPU allocation (model spanning 2+ GPUs) — explicitly out of
  scope
- MIG (Multi-Instance GPU) partitioning — explicitly out of scope
- GPU time-slicing — explicitly out of scope
- Dynamic batching configuration and testing — explicitly out of scope
- gRPC inference protocol — REST/HTTP V2 only
- Konflux/Tekton GPU build pipeline validation — pre-built images
  referenced directly, pipeline not required for current release
- V1 KServe protocol support — V2 only
- Multi-model serving — single-model only
- Custom CUDA execution provider configuration at deployment time —
  baked into image
- Training workloads (this strategy covers inference only)
- Intel GPU support via OVMS

### 1.3 Test Objectives

1. Verify the GPU MLServer container image starts successfully with the
   CUDA execution provider enabled, as evidenced by runtime logs and GPU
   utilization during inference
2. Validate that selecting the `nvidia-gpu-a100` HardwareProfile
   correctly propagates the GPU resource request (`nvidia.com/gpu: 1`)
   and toleration (`nvidia.com/gpu` operator Exists, effect NoSchedule)
   to the serving pod
3. Confirm that deploying the GPU runtime without a GPU HardwareProfile
   results in pod Pending status (no silent CPU fallback)
4. Ensure the existing CPU MLServer image and ServingRuntime
   template remain fully functional and unaffected by the GPU additions
5. Validate KServe V2 inference endpoints (`/v2/models/{model}/infer`,
   `/v2/models/{model}/ready`, `/v2/health/ready`) on port 8080 produce
   correct results using the ResNet-50 reference model
6. Verify air-gapped GPU runtime deployment succeeds when both the
   MLServer CUDA image is mirrored to an internal registry and the ONNX
   model is pre-staged to in-cluster storage
7. Confirm startup, readiness, and liveness probes function correctly per
   the template specification
8. Verify the `mlserver-cuda-runtime-template` appears in the Dashboard
   UI via annotation-driven discovery without any Dashboard code changes
9. Confirm `model-settings.json` configuration follows upstream MLServer
   settings schema (CUDA EP baked into image, not user-configurable)

---

## 2. Test Strategy

### 2.1 Test Levels

- **API Integration Testing** — Verify GPU ServingRuntime template
  deployment via odh-model-controller, KServe inference endpoint
  availability, and model serving API responses when using
  GPU-accelerated runtimes (MLServer with onnxruntime-gpu)
- **Functional Testing** — Validate end-to-end GPU inference workflows:
  model deployment using the `mlserver-cuda-runtime` ServingRuntime,
  HardwareProfile-based GPU allocation, CUDA execution provider
  initialization (baked into image), and Dashboard auto-discovery
  of GPU runtimes
- **Data Validation Testing** — Verify inference output correctness and
  numerical consistency between CPU and GPU execution providers for
  identical models and inputs; validate `model-settings.json` schema
  compliance with upstream MLServer specifications
- **UI Testing** — Confirm GPU ServingRuntime appears in Dashboard runtime
  selection via annotation-driven discovery, and that the
  HardwareProfile correctly links the runtime to NVIDIA GPU resources
- **Performance Testing** — Baseline establishment only until SLOs
  defined; measure inference latency, throughput, and scalability under
  GPU-accelerated workloads (dynamic batching explicitly out of scope)
- **Security Testing** — Validate container image provenance for the new
  GPU image built on `aipcc/cuda` base, verify non-root execution with
  all capabilities dropped and privilege escalation disallowed, and
  confirm RBAC enforcement on ServingRuntime resources

### 2.2 Test Types

- **Positive Testing** — Deploy GPU runtime with correct HardwareProfile,
  run inference with supported ONNX models, verify CUDA execution
  provider activation, confirm Dashboard discovery of GPU runtime
- **Negative Testing** — Deploy GPU runtime without GPU HardwareProfile
  (expect pod Pending, not silent CPU fallback), attempt inference with
  incompatible model formats, deploy GPU image on nodes without NVIDIA
  GPU resources
- **Boundary Testing** — High-concurrency inference requests, large batch
  sizes, maximum resolution inputs, multiple simultaneous model
  deployments on shared GPU resources
- **Regression Testing** — Verify existing CPU MLServer image and
  ServingRuntime template remain fully functional and unchanged;
  confirm no behavioral changes to existing model serving workflows

### 2.3 Test Priorities

- **P0 (Critical)** — GPU inference produces correct results; GPU runtime
  deploys successfully with HardwareProfile-based GPU allocation; CPU
  runtime remains unaffected (backwards compatibility); GPU template
  without GPU HardwareProfile results in pod Pending (no silent CPU
  fallback)
- **P1 (High)** — Dashboard auto-discovers GPU runtime via annotations;
  `model-settings.json` correctly configures CUDA execution provider
  (baked into image); CSV `relatedImages` includes both CPU and GPU
  images; air-gapped deployment of GPU runtime when image pre-mirrored
  and model pre-staged; startup, readiness, and liveness probes
  function per template spec
- **P2 (Medium)** — Performance benchmarks under sustained load (baseline
  establishment until SLOs defined); GPU runtime behavior under
  resource contention; resource consumption patterns measurement

---

## 3. Test Environment

### 3.1 Test Cluster Configuration

- **OpenShift version**: 4.20+ (minimum supported)
- **RHOAI version**: 3.5 GA
- **NVIDIA GPU Operator**: 12.9+ (minimum supported; no ClusterPolicy
  customizations beyond defaults required)
- **CUDA toolkit/cuDNN**: Provided inside the GPU container image
  (`aipcc/cuda` base); no cluster-level CUDA install required beyond
  the GPU Operator
- **KServe**: Required — ServingRuntime/InferenceService CRDs must be
  available
- **odh-model-controller**: Required — manages ServingRuntime templates
  from `config/runtimes/`
- **MLServer images**: Two container images required:
  - CPU image (existing, unchanged)
  - GPU image (`$(mlserver-cuda-image)` — auto-populated on RHOAI
    clusters, built on `aipcc/cuda` base with `onnxruntime-gpu`)
- **HardwareProfile**: `infrastructure.opendatahub.io/v1` API group;
  cluster must support HardwareProfile-based GPU allocation (GA as of
  RHOAI 3.0)
- **Node requirements**: At least one worker node with any modern NVIDIA
  GPU hardware (no restricted SKU list — any GPU compatible with GPU
  Operator 12.9+)
- **Feature gates**: None required (GPU runtime available by default once
  template is present)

### 3.2 Test Data Requirements

- **ServingRuntime templates**:
  - CPU: existing `mlserver-runtime` in
    `odh-model-controller/config/runtimes/`
  - GPU: `mlserver-cuda-runtime-template` in
    `redhat-ods-applications` namespace
- **ONNX model artifacts**: ResNet-50 (ONNX format) from
  onnx-community/resnet-50-ONNX, hosted on Red Hat-maintained S3
  bucket at path `mlserver/model_repository/onnx_resnet50/` (bucket
  name injected via `models_s3_bucket_name` pytest fixture, not
  hardcoded; not fetched from HuggingFace during tests; distinct from
  existing iris model at `mlserver/model_repository/onnx/`)
- **model-settings.json**: Configuration following upstream MLServer
  settings schema (ModelSettings, ModelParameters, Settings); CUDA EP
  baked into image, not user-configurable
- **InferenceService CR**: Sample InferenceService definitions
  referencing both CPU and GPU runtimes
- **HardwareProfile CR**: `nvidia-gpu-a100` in
  `redhat-ods-applications` namespace with label
  `app.opendatahub.io/hardwareprofile: "true"`, resource identifiers:
  - CPU: default 4, min 1, max 16
  - Memory: default 16Gi, min 4Gi, max 64Gi
  - `nvidia.com/gpu`: default 1, min 1, max 8
  - Toleration: `nvidia.com/gpu` operator Exists, effect NoSchedule
- **kustomization.yaml**: Updated kustomization including the new GPU
  runtime template
- **CSV relatedImages entries**: Both CPU and GPU image references for
  OLM/CSV validation
- **Inference request payloads**: Sample KServe V2 protocol payloads for
  `/v2/models/{model}/infer`

### 3.3 Test Users

- **cluster-admin**: For installing operators, creating
  ServingRuntimes, and managing cluster-scoped resources
- **namespace-admin**: For creating InferenceServices, HardwareProfiles,
  and ServingRuntimes within a project
- **data-scientist (unprivileged)**: For deploying models via Dashboard UI
  and verifying annotation-driven runtime discovery
- **Service account**: For InferenceService pods running under KServe with
  appropriate RBAC to pull images and access model storage

---

## 4. Components and Configuration Under Test

| Component / Config | Type | Purpose | Priority |
|--------------------|------|---------|----------|
| `mlserver-cuda-runtime-template` | OpenShift Template | Pre-installed in `redhat-ods-applications` namespace, creates `mlserver-cuda-runtime` ServingRuntime with ONNX v1 support, HTTP port 8080, single-model serving, V2 protocol | P0 |
| HardwareProfile CR (`nvidia-gpu-a100`) | Config | GPU resource allocation with `nvidia.com/gpu` (default: 1, min: 1, max: 8), CPU (default: 4, min: 1, max: 16), memory (default: 16Gi, min: 4Gi, max: 64Gi), toleration for GPU-tainted nodes (`nvidia.com/gpu` operator Exists, effect NoSchedule) | P0 |
| GPU MLServer container image | Image | `$(mlserver-cuda-image)` — auto-populated on RHOAI clusters, built on `aipcc/cuda` base + `onnxruntime-gpu` + MLServer, runtime version 1.7.1 | P0 |
| CPU MLServer container image (existing) | Image | Unchanged CPU image — backwards compatibility | P0 |
| KServe V2 `/v2/models/{model}/infer` (GPU) | REST | Inference endpoint on GPU runtime port 8080 — GPU-accelerated CUDA execution provider | P0 |
| KServe V2 `/v2/models/{model}/ready` (GPU) | REST | Model readiness endpoint on GPU runtime port 8080 — used by readiness and liveness probes | P0 |
| Readiness probe (HTTP) | Probe | HTTP GET `/v2/models/{model}/ready` port 8080 (5s initial delay, 5s period, 3 failure threshold, 5s timeout). _Timing values sourced from context document's ServingRuntime template specification (RHAISTRAT-1868-context.md)._ | P0 |
| Liveness probe (HTTP) | Probe | HTTP GET `/v2/models/{model}/ready` port 8080 (20s initial delay, 10s period, 10 failure threshold, 5s timeout). _Timing values sourced from context document's ServingRuntime template specification (RHAISTRAT-1868-context.md)._ | P0 |
| GPU pod scheduling behavior (no GPU HardwareProfile) | Behavior | Pod enters Pending — no silent CPU fallback | P0 |
| KServe V2 `/v2/models/{model}/infer` (CPU) | REST | Inference endpoint on CPU runtime for regression | P0 |
| `opendatahub.io/dashboard: "true"` label | Annotation | Dashboard auto-discovery for GPU runtime | P1 |
| `opendatahub.io/recommended-accelerators` annotation | Annotation | Links GPU runtime to NVIDIA GPU HardwareProfile (`nvidia.com/gpu`) | P1 |
| `model-settings.json` (GPU variant) | Config | Per-model config following upstream MLServer settings schema — CUDA EP baked into image, not user-configurable | P1 |
| KServe V2 `/v2/health/ready` (GPU) | REST | Server health endpoint on GPU runtime port 8080 | P1 |
| KServe V2 `/v2/models/{model}/ready` (CPU) | REST | Model readiness check on CPU runtime for regression | P1 |
| Startup probe (exec) | Probe | Checks `/mnt/models` non-empty (1s initial delay, 1s period, 1 failure threshold). _Configuration sourced from context document's ServingRuntime template specification (RHAISTRAT-1868-context.md)._ | P1 |
| CSV `relatedImages` (CPU + GPU) | Config | OLM image references for both image variants | P1 |
| `kustomization.yaml` (updated) | Config | Includes new GPU runtime template | P1 |
| Environment variables | Config | `MLSERVER_MODEL_IMPLEMENTATION: {{.Labels.modelClass}}` (template variable), `MLSERVER_HTTP_PORT: 8080`, `MLSERVER_MODELS_DIR: /mnt/models`, `MLSERVER_MODEL_ONNX_PROVIDERS` (CUDA image only, overridable) | P1 |
| Security context | Config | Non-root execution, all capabilities dropped, privilege escalation disallowed | P1 |
| CUDA execution provider initialization logs | Log | Verify CUDA execution provider loads in GPU pod (baked into image at build time) | P1 |

---

## 5. Test Cases

Test cases have been generated — 17 total (8 P0, 8 P1, 1 P2).
Consolidated from 41 original TCs after PR review feedback
(redundancy analysis, framework alignment, architectural
corrections).

**Test Cases Directory**: [test_cases/](test_cases/)
**Complete Test Case Index**: [test_cases/INDEX.md](test_cases/INDEX.md)

### 5.1 Test Case Organization

| Category | Test Cases | Priority Distribution |
|----------|------------|----------------------|
| TC-DEPLOY | 2 | 2 P0 |
| TC-INFER | 3 | 2 P0, 1 P1 |
| TC-COMPAT | 2 | 1 P0, 1 P1 |
| TC-FALLBACK | 2 | 1 P0, 1 P1 |
| TC-DISC | 1 | 1 P0 |
| TC-BUILD | 2 | 2 P1 |
| TC-PERF | 1 | 1 P2 |
| TC-RBAC | 1 | 1 P1 |
| TC-UPGRADE | 3 | 2 P0, 1 P1 |
| **Total** | **17** | **8 P0, 8 P1, 1 P2** |

### 5.2 Test Case Naming Convention

Test cases follow the naming pattern: `TC-<CATEGORY>-<NUMBER>`

- `TC-DEPLOY` — GPU runtime deployment and template validation
- `TC-INFER` — GPU inference correctness and execution provider tests
- `TC-COMPAT` — CPU backwards compatibility and regression tests
- `TC-FALLBACK` — Silent CPU fallback detection tests
- `TC-DISC` — Dashboard discovery (manual)
- `TC-BUILD` — CSV image references and architecture separation
- `TC-PERF` — Performance baseline tests
- `TC-RBAC` — RBAC and authorization tests
- `TC-UPGRADE` — Upgrade and migration tests

---

## 6. E2E Test Scenarios

Dedicated E2E test cases (TC-E2E-*) were removed during PR review
consolidation. The Dashboard-driven user journey (TC-E2E-001) required
browser automation incompatible with the `opendatahub-tests` pytest
framework. The remaining E2E scenarios were redundant with individual
TCs (TC-E2E-002 ≡ TC-FALLBACK-001, TC-E2E-003 ≡ TC-INFER-002).

E2E coverage is now provided by:

- **TC-DISC-001** (manual): Dashboard GPU runtime discovery
- **TC-DEPLOY-001 → TC-INFER-001**: Template → ServingRuntime → ISVC
  → inference chain
- **TC-INFER-002**: Cross-validates GPU vs CPU inference results
- **TC-COMPAT-002**: CPU runtime unaffected after GPU runtime added

### 6.1 E2E Coverage Matrix

| Endpoint (from Section 4) | Covering TCs |
|----------------------------|-------------|
| `mlserver-cuda-runtime-template` | TC-DEPLOY-001, TC-UPGRADE-001 |
| `opendatahub.io/dashboard: "true"` label | TC-DEPLOY-001, TC-DISC-001 |
| `opendatahub.io/recommended-accelerators` annotation | TC-DEPLOY-001, TC-DISC-001 |
| HardwareProfile CR for `nvidia.com/gpu` | TC-DISC-001 (manual) |
| CSV `relatedImages` (CPU + GPU) | TC-BUILD-001 |
| GPU MLServer container image | TC-BUILD-002, TC-DEPLOY-002 |
| CPU MLServer container image (existing) | TC-BUILD-002, TC-COMPAT-001 |
| KServe V2 `/v2/models/{model}/infer` (GPU) | TC-INFER-001, TC-INFER-002 |
| KServe V2 `/v2/models/{model}/ready` (GPU) | TC-INFER-001 |
| KServe V2 `/v2/health/ready` (GPU) | TC-INFER-001 |
| KServe V2 `/v2/models/{model}/infer` (CPU) | TC-COMPAT-001, TC-INFER-002 |
| KServe V2 `/v2/models/{model}/ready` (CPU) | TC-COMPAT-001 |
| GPU pod scheduling (silent CPU fallback) | TC-FALLBACK-001 |
| CUDA execution provider initialization logs | TC-DEPLOY-002, TC-FALLBACK-001 |

---

## 7. Non-Functional Requirements

Each category below must be explicitly addressed. If a category
does not apply to this feature, state **Not Applicable** with a
brief justification.

### 7.1 Disconnected/Air-Gapped

The two-image architecture ensures that CPU workloads do not require the
GPU image to be pulled, avoiding the ~500MB–1GB CUDA payload for
CPU-only deployments. Note: Both CPU and GPU MLServer images and their
ServingRuntime templates always co-exist on all clusters (including
air-gapped environments), pre-installed together by DevOps automation.
GPU runtime air-gapped deployment requires two conditions: (1) GPU image
mirrored to internal registry, and (2) ONNX model pre-staged to
in-cluster S3-compatible storage or PVC. Testing considerations:

- Verify that deployments using only the CPU runtime in air-gapped
  environments function without requiring the GPU image to be pulled
- Confirm the GPU image can be mirrored to a disconnected registry and
  deployed successfully from that registry, with ResNet-50 model
  pre-staged to S3-compatible in-cluster storage (not fetched from
  HuggingFace during tests)
- Validate that the ServingRuntime GPU template references images via
  digest (as required for CSV `relatedImages`) so air-gapped image
  mirroring works correctly
- Test that no external registry pulls occur at runtime for either CPU
  or GPU images in a disconnected environment
- Verify catalog source configuration includes the GPU runtime when GPU
  support is desired in disconnected clusters

### 7.2 Upgrade/Migration

No CRD schema changes, no API version changes, no persistent state
migration required. The strategy introduces new artifacts (GPU image, GPU
ServingRuntime template) but explicitly states the CPU image is unchanged
(backwards compatible). Testing considerations:

- Verify upgrade from a release without GPU runtime to a release with GPU
  runtime: new GPU ServingRuntime template appears, existing CPU
  deployments unaffected
- Confirm rollback scenario: removing the GPU runtime template does not
  impact existing CPU-based InferenceServices
- Validate CSV `relatedImages` is correctly updated to include both CPU
  and GPU images, and that OLM handles the addition during operator
  upgrade
- Test that existing models deployed on the CPU runtime continue to serve
  without interruption after the operator upgrade introduces the GPU
  template
- Verify `kustomization.yaml` changes in
  `odh-model-controller/config/runtimes/` are applied cleanly during
  upgrade

### 7.3 Performance/Scalability

Dynamic batching is out of scope, multi-GPU is out of scope, and
performance SLOs are undefined — baseline establishment only until SLOs
are defined. The strategy explicitly targets performance-sensitive
workloads but requires performance SLO definition before pass/fail
assertions can be made. Testing considerations:

- Baseline establishment: measure inference latency (p50/p95/p99) under
  increasing concurrency to establish performance characteristics
- Test resource consumption patterns to ensure no GPU memory leaks under
  sustained load
- Benchmark GPU memory utilization and throughput at different model
  sizes and resolutions
- Compare GPU vs CPU inference latency and throughput to validate the
  performance benefit justification
- Collect metrics for future SLO definition (no pass/fail assertions
  until SLOs defined)
- GPU image size (~500MB–1GB) impacts deployment times — measure pull
  times and validate registry requirements

### 7.4 RBAC/Authorization

No multi-tenant isolation concerns; single-model serving only. The
feature introduces new ServingRuntime resources and relies on existing
KServe/odh-model-controller RBAC patterns. Testing considerations:

- Verify that creation, modification, and deletion of the GPU
  ServingRuntime follows existing RBAC policies (cluster-admin vs
  namespace-scoped users)
- Confirm that deploying an InferenceService using the GPU runtime
  respects the same authorization boundaries as CPU runtimes
- Validate that the GPU HardwareProfile association does not bypass any
  existing authorization checks
- Test that unprivileged users cannot modify the
  `recommended-accelerators` annotation to redirect GPU allocation

---

## 8. Risks and Mitigation

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| GPU test environment availability — testing requires NVIDIA GPU-equipped nodes which may have limited availability in CI/test infrastructure; any modern NVIDIA GPU compatible with GPU Operator 12.9+ is supported (no restricted SKU list) | High | High | Secure dedicated GPU node pool for test execution; define minimum GPU hardware requirements for test environments; no specific GPU model required |
| CUDA execution provider baked into container image at build time — incorrect build-time configuration cannot be fixed at deployment time without image rebuild | High | Medium | Validate CUDA execution provider initialization early; verify GPU utilization and runtime logs for CUDA provider activation; establish compatibility matrix for CUDA toolkit, cuDNN, and onnxruntime-gpu version combinations |
| Performance SLOs undefined — TC-PERF-001, TC-PERF-002, TC-PERF-003 cannot assert pass/fail criteria until concrete latency targets, throughput benchmarks, and concurrency limits are established | High | High | Collect and report performance metrics for baseline establishment; defer pass/fail assertions until engineering defines concrete SLOs; re-baseline after SLO definition |
| ResNet-50 reference model must be manually uploaded to Red Hat-maintained S3 bucket — missing or incorrectly staged model blocks all inference validation tests | High | Medium | Document S3 bucket path in test setup procedures; automate model pre-staging verification before test execution; establish fallback process for model hosting issues |
| Silent CPU fallback prevention depends on correct HardwareProfile integration — if HardwareProfile matching fails, workloads may silently run on CPU | High | Low | Implement explicit test cases for the "GPU template without GPU HardwareProfile = pod Pending" scenario; add automated regression checks |
| GPU image size (~500MB–1GB additional CUDA payload) may impact deployment times and registry storage in resource-constrained environments | Medium | Medium | Measure and document GPU image pull times; test deployment timeouts with large image sizes; validate registry storage requirements |
| OVMS CUDA deprecation (RHOAI 2.25) may cause confusion if residual OVMS CUDA references remain in docs or tests | Medium | Medium | Audit all test suites and documentation for OVMS CUDA references; ensure test plan explicitly excludes OVMS CUDA scenarios |
| Dashboard annotation-driven discovery assumes specific label and annotation conventions — changes to Dashboard discovery logic could break GPU runtime visibility | Medium | Low | Test annotation-based discovery end-to-end; verify annotation behavior; add regression tests for annotation parsing |

---

## 9. Test Environment Requirements

### 9.1 Infrastructure

- GPU-enabled OpenShift 4.20+ cluster with at least one NVIDIA GPU node
  (any modern NVIDIA GPU compatible with GPU Operator 12.9+, no SKU
  restrictions)
- NVIDIA GPU Operator 12.9+ installed and configured (default
  ClusterPolicy sufficient)
- Nodes without GPU resources (for testing CPU runtime and silent CPU
  fallback scenarios)
- Air-gapped/disconnected environment (recommended) for validating that
  CPU workloads do not require the GPU image to be pulled, and for
  testing GPU deployments with mirrored images + pre-staged models
- S3-compatible object storage (Red Hat-maintained S3 bucket for
  ResNet-50 model; MinIO or AWS S3 for general testing)
- Container registry access for pulling both CPU and GPU MLServer
  images and `aipcc/cuda` base image

### 9.2 Configuration

- **OpenShift version**: 4.20+ (confirmed)
- **RHOAI version**: 3.5 GA (confirmed)
- **NVIDIA GPU Operator**: 12.9+ (confirmed)
- **Template**: `mlserver-cuda-runtime-template` in
  `redhat-ods-applications` namespace
- **Runtime name**: `mlserver-cuda-runtime`
- **HardwareProfile CRD**: `infrastructure.opendatahub.io/v1` API group
- **ServingRuntime templates**:
  - CPU: existing `mlserver-runtime` in
    `odh-model-controller/config/runtimes/`
  - GPU: `mlserver-cuda-runtime-template` in
    `redhat-ods-applications` namespace
- **Dashboard label**: `opendatahub.io/dashboard: "true"` on both
  ServingRuntimes
- **model-settings.json**: CUDA EP baked into image (not user-configurable
  at deployment time)
- **CSV `relatedImages`**: both CPU and GPU image digests
- **HardwareProfile configuration**: GPU resource requests via
  HardwareProfile (not hardcoded resource limits)
- **Feature gates**: None required (confirmed — available by default)
- **HTTP port**: 8080, protocol: V2 REST only (no gRPC, no V1)
- **Single-model serving only** (multi-model: false), ONNX format v1
- **Image reference**: `$(mlserver-cuda-image)` — auto-populated on
  RHOAI clusters
- **Security context**: non-root, drops all capabilities, disallows
  privilege escalation
- **Probes**:
  - Startup: exec, checks `/mnt/models` non-empty (1s initial delay,
    1s period, 1 failure threshold)
  - Readiness: HTTP GET `/v2/models/{model}/ready` port 8080 (5s
    initial delay, 5s period, 3 failure threshold, 5s timeout)
  - Liveness: HTTP GET `/v2/models/{model}/ready` port 8080 (20s
    initial delay, 10s period, failureThreshold: 10 GPU / 6 CPU,
    5s timeout)
- **Environment variables**: `MLSERVER_MODEL_IMPLEMENTATION:
  {{.Labels.modelClass}}` (template variable, resolved at
  instantiation), `MLSERVER_HTTP_PORT: 8080`,
  `MLSERVER_MODELS_DIR: /mnt/models`,
  `MLSERVER_MODEL_ONNX_PROVIDERS` (CUDA image only, overridable)
- **Tolerations**: `nvidia.com/gpu` operator Exists, effect NoSchedule
- **Container resource defaults**: GPU runtime: CPU 1-4, Memory
  4Gi-8Gi; CPU runtime: CPU 500m-2, Memory 1Gi-4Gi (values differ
  between runtimes)

### 9.3 Test Tools

- `ocp-resources` Python library — CRD management, pod inspection,
  resource allocation verification (preferred over raw `oc` commands)
- `opendatahub-tests` fixtures — `ServingRuntimeFromTemplate`,
  `create_isvc`, `get_exposed_isvc_url`, `send_rest_request`,
  `validate_deterministic_snapshot`, `unprivileged_client`
- `curl` / `httpie` — KServe V2 REST inference requests to
  `/v2/models/{model}/infer`, `/v2/models/{model}/ready`,
  `/v2/health/ready`
- `nvidia-smi` — GPU allocation and utilization verification (inside
  pod or on node)
- `oc logs` / `kubectl logs` — CUDA execution provider initialization
  verification in container logs
- `skopeo` / `podman` — Container image layer inspection (verify GPU
  image contains CUDA libraries, CPU image does not)
- `kustomize` — Validate `kustomization.yaml` correctness with new GPU
  runtime template
- Performance/load tools — Inference latency, throughput, and
  scalability testing; recommended options include Locust or k6 (to be
  confirmed during test execution planning)

---

## 10. Implementation Guidance

### 10.1 Framework Alignment

All test cases in this plan should be implemented using the
`opendatahub-tests` framework conventions:

- Use `ocp-resources` Python library, not shell `oc` commands
- Template-based ServingRuntime instantiation via
  `ServingRuntimeFromTemplate`, not direct YAML apply
- GPU tests use `external_route=True` and access inference via
  `get_exposed_isvc_url(isvc)` + `status.url` (same pattern as vLLM
  and AutoGluon runtimes)
- Image reference extraction via `get_runtime_image_from_template`
  from `utilities/serving_runtime.py`
- Pod operations via `get_pods_by_isvc_label`, `pod.execute()`,
  `pod.log()`

### 10.2 Required Framework Additions

Before GPU MLServer test implementation, the following must be added
to `opendatahub-tests`:

- `RuntimeTemplates.MLSERVER_CUDA` constant
- `ModelInferenceRuntime.MLSERVER_CUDA_RUNTIME` constant
- `RESNET50_REST_INPUT_QUERY` constant with shape `[1, 3, 224, 224]`
- ResNet-50 model staged at S3 path
  `mlserver/model_repository/onnx_resnet50/`
- Utility functions: `verify_runtime_labels`,
  `verify_ort_device`, `verify_cuda_ep_active`,
  `compare_top_k_predictions`
- Fixtures: `mlserver_cuda_serving_runtime`,
  `mlserver_cuda_inference_service`, `mlserver_cuda_pod`

### 10.3 Key Implementation Notes

- **Bucket name injection**: Never hardcode S3 bucket names; use
  `models_s3_bucket_name` pytest fixture
- **Resource allocation**: `opendatahub-tests` injects GPU resources
  directly via ISVC spec, not via HardwareProfile CRs (Dashboard-only
  flow)
- **Response validation**: `validate_deterministic_snapshot`
  validates response structure only, not specific class indices or
  numeric values
- **Distribution differences**: Base images differ between RHOAI
  (aipcc/cuda, aipcc/cpu) and ODH (ubi9/ubi-minimal); tests must be
  parameterized
- **RBAC tests**: Use `unprivileged_client` fixture and assert
  `ForbiddenError` exceptions, not shell `oc login` commands

---

## 11. Appendix

### 11.1 Test Case Summary

| Category | Total | P0 | P1 | P2 |
|----------|-------|----|----|-----|
| TC-DEPLOY | 2 | 2 | 0 | 0 |
| TC-INFER | 3 | 2 | 1 | 0 |
| TC-COMPAT | 2 | 1 | 1 | 0 |
| TC-FALLBACK | 2 | 1 | 1 | 0 |
| TC-DISC | 1 | 1 | 0 | 0 |
| TC-BUILD | 2 | 0 | 2 | 0 |
| TC-PERF | 1 | 0 | 0 | 1 |
| TC-RBAC | 1 | 0 | 1 | 0 |
| TC-UPGRADE | 3 | 2 | 1 | 0 |
| **Total** | **17** | **8** | **8** | **1** |

### 11.2 Component Coverage

| Component / Config | Test Cases | Coverage |
|--------------------|------------|----------|
| `mlserver-cuda-runtime-template` | TC-DEPLOY-001, TC-DEPLOY-002, TC-UPGRADE-001 | |
| `opendatahub.io/dashboard: "true"` label | TC-DEPLOY-001, TC-DISC-001 | |
| `opendatahub.io/recommended-accelerators` annotation | TC-DEPLOY-001, TC-DISC-001 | |
| HardwareProfile CR for `nvidia.com/gpu` | TC-DISC-001 (manual) | |
| CSV `relatedImages` (CPU + GPU) | TC-BUILD-001 | |
| GPU MLServer container image | TC-BUILD-001, TC-BUILD-002, TC-DEPLOY-002 | |
| CPU MLServer container image (existing) | TC-BUILD-002, TC-COMPAT-001 | |
| KServe V2 `/v2/models/{model}/infer` (GPU) | TC-INFER-001, TC-INFER-002, TC-INFER-003 | |
| KServe V2 `/v2/models/{model}/ready` (GPU) | TC-INFER-001 | |
| KServe V2 `/v2/health/ready` (GPU) | TC-INFER-001 | |
| KServe V2 `/v2/models/{model}/infer` (CPU) | TC-COMPAT-001, TC-INFER-002 | |
| KServe V2 `/v2/models/{model}/ready` (CPU) | TC-COMPAT-001 | |
| GPU pod scheduling (silent CPU fallback) | TC-FALLBACK-001 | |
| GPU pod scheduling (no GPU nodes) | TC-FALLBACK-002 | |
| Readiness probe (HTTP) | TC-INFER-001 | |
| Environment variables | TC-DEPLOY-001 | |
| Security context | TC-DEPLOY-001 | |
| CUDA EP initialization logs | TC-DEPLOY-002, TC-FALLBACK-001 | |
| RBAC (namespace boundary) | TC-RBAC-001 | |
| Upgrade resilience | TC-UPGRADE-001, TC-UPGRADE-002, TC-UPGRADE-003 | |

### 11.3 Document Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1.3.0 | 2026-07-21 | PR review feedback: consolidated 41→17 TCs, corrected ServingRuntime kind, CPU runtime name, CSV namespace, CUDA EP configurability, template variable, silent CPU fallback, opendatahub-tests framework alignment |
| 1.2.0 | 2026-07-17 | Updated with RHAISTRAT-1868-context.md; corrected runtime name to mlserver-cuda-runtime; added HardwareProfile specs, probe timings, security context; regenerated 41 test cases |
| 1.0.0 | 2026-06-17 | Initial test plan |

---

**End of Test Plan**
