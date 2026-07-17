# Test Case Index — GPU Enabled Runtimes

**Parent Test Plan**: [TestPlan.md](../TestPlan.md)

## Quick Stats

| Metric | Count |
|--------|-------|
| Total Test Cases | 41 |
| P0 (Critical) | 19 |
| P1 (High) | 17 |
| P2 (Medium) | 5 |

---

## Deployment (TC-DEPLOY)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-DEPLOY-001](TC-DEPLOY-001.md) | Apply mlserver-cuda-runtime ClusterServingRuntime template | P0 |
| [TC-DEPLOY-002](TC-DEPLOY-002.md) | Deploy InferenceService with GPU runtime and HardwareProfile | P0 |
| [TC-DEPLOY-003](TC-DEPLOY-003.md) | Verify GPU container initializes CUDA execution provider | P0 |
| [TC-DEPLOY-004](TC-DEPLOY-004.md) | Verify kustomization.yaml includes GPU runtime template | P1 |
| [TC-DEPLOY-005](TC-DEPLOY-005.md) | Verify recommended-accelerators annotation on GPU runtime | P0 |
| [TC-DEPLOY-006](TC-DEPLOY-006.md) | Verify HardwareProfile-based GPU allocation | P0 |

## Inference (TC-INFER)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-INFER-001](TC-INFER-001.md) | GPU inference correctness via KServe V2 infer endpoint | P0 |
| [TC-INFER-002](TC-INFER-002.md) | Numerical consistency between GPU and CPU inference | P0 |
| [TC-INFER-003](TC-INFER-003.md) | GPU model readiness via KServe V2 ready endpoint | P0 |
| [TC-INFER-004](TC-INFER-004.md) | GPU server readiness via KServe V2 health endpoint | P1 |
| [TC-INFER-005](TC-INFER-005.md) | CUDA execution provider initialization in pod logs | P1 |
| [TC-INFER-006](TC-INFER-006.md) | Inference with invalid input returns error on GPU runtime | P1 |

## Compatibility (TC-COMPAT)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-COMPAT-001](TC-COMPAT-001.md) | CPU MLServer deploys and serves inference | P0 |
| [TC-COMPAT-002](TC-COMPAT-002.md) | CPU model readiness via KServe V2 ready endpoint | P1 |
| [TC-COMPAT-003](TC-COMPAT-003.md) | CPU inference correctness via KServe V2 infer endpoint | P0 |
| [TC-COMPAT-004](TC-COMPAT-004.md) | CPU runtime unaffected after GPU template addition | P0 |

## Fallback Prevention (TC-FALLBACK)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-FALLBACK-001](TC-FALLBACK-001.md) | GPU runtime without GPU HardwareProfile results in pod Pending | P0 |
| [TC-FALLBACK-002](TC-FALLBACK-002.md) | GPU runtime on node without NVIDIA GPU results in pod Pending | P0 |
| [TC-FALLBACK-003](TC-FALLBACK-003.md) | No inference accessible during Pending state | P0 |

## Dashboard Discovery (TC-DISC)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-DISC-001](TC-DISC-001.md) | GPU runtime appears in Dashboard via dashboard label | P1 |
| [TC-DISC-002](TC-DISC-002.md) | recommended-accelerators annotation links GPU runtime to HardwareProfile | P0 |
| [TC-DISC-003](TC-DISC-003.md) | CPU runtime remains visible in Dashboard after GPU runtime addition | P1 |

## Build and Image (TC-BUILD)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-BUILD-001](TC-BUILD-001.md) | CSV relatedImages contains both CPU and GPU image digests | P1 |
| [TC-BUILD-002](TC-BUILD-002.md) | GPU image contains CUDA libraries | P1 |
| [TC-BUILD-003](TC-BUILD-003.md) | CPU image unchanged -- no CUDA libraries present | P1 |
| [TC-BUILD-004](TC-BUILD-004.md) | GPU runtime template references image via variable | P1 |

## Performance (TC-PERF)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-PERF-001](TC-PERF-001.md) | GPU inference latency baseline under concurrent requests | P2 |
| [TC-PERF-002](TC-PERF-002.md) | GPU vs CPU inference throughput comparison baseline | P2 |
| [TC-PERF-003](TC-PERF-003.md) | GPU memory utilization under sustained load | P2 |

## Air-Gapped (TC-AIRGAP)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-AIRGAP-001](TC-AIRGAP-001.md) | CPU-only air-gapped deployment works without GPU image | P2 |
| [TC-AIRGAP-002](TC-AIRGAP-002.md) | GPU runtime in air-gapped env with mirrored image and pre-staged model | P2 |

## RBAC (TC-RBAC)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-RBAC-001](TC-RBAC-001.md) | Unprivileged user cannot create ClusterServingRuntime | P1 |
| [TC-RBAC-002](TC-RBAC-002.md) | Namespace admin deploys InferenceService with GPU runtime | P1 |
| [TC-RBAC-003](TC-RBAC-003.md) | Unprivileged user cannot modify recommended-accelerators annotation | P1 |

## Upgrade Testing (TC-UPGRADE)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-UPGRADE-001](TC-UPGRADE-001.md) | GPU template appears after operator upgrade | P1 |
| [TC-UPGRADE-002](TC-UPGRADE-002.md) | CPU InferenceServices unaffected by operator upgrade | P0 |
| [TC-UPGRADE-003](TC-UPGRADE-003.md) | Rollback removes GPU template without CPU impact | P1 |
| [TC-UPGRADE-004](TC-UPGRADE-004.md) | kustomization.yaml changes applied during upgrade | P1 |

## End-to-End (TC-E2E)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-E2E-001](TC-E2E-001.md) | Data scientist deploys GPU model via Dashboard and runs inference | P0 |
| [TC-E2E-002](TC-E2E-002.md) | GPU deployment with HardwareProfile and fallback verification | P0 |
| [TC-E2E-003](TC-E2E-003.md) | Side-by-side CPU and GPU deployment with cross-validation | P0 |
