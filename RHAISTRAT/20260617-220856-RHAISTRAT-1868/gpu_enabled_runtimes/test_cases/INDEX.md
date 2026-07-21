# Test Case Index — GPU Enabled Runtimes

**Parent Test Plan**: [TestPlan.md](../TestPlan.md)

## Quick Stats

| Metric | Count |
|--------|-------|
| Total Test Cases | 17 |
| P0 (Critical) | 8 |
| P1 (High) | 8 |
| P2 (Medium) | 1 |

---

## Deployment (TC-DEPLOY)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-DEPLOY-001](TC-DEPLOY-001.md) | Verify GPU runtime template and instantiated ServingRuntime | P0 |
| [TC-DEPLOY-002](TC-DEPLOY-002.md) | Deploy GPU InferenceService and verify pod initialization | P0 |

## Inference (TC-INFER)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-INFER-001](TC-INFER-001.md) | GPU inference correctness via KServe V2 infer endpoint | P0 |
| [TC-INFER-002](TC-INFER-002.md) | GPU vs CPU numerical consistency | P0 |
| [TC-INFER-003](TC-INFER-003.md) | Invalid input returns error on GPU runtime | P1 |

## Compatibility (TC-COMPAT)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-COMPAT-001](TC-COMPAT-001.md) | CPU MLServer deployment and inference baseline | P0 |
| [TC-COMPAT-002](TC-COMPAT-002.md) | CPU runtime unaffected after GPU runtime added | P1 |

## Fallback (TC-FALLBACK)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-FALLBACK-001](TC-FALLBACK-001.md) | Silent CPU fallback detection | P0 |
| [TC-FALLBACK-002](TC-FALLBACK-002.md) | Pod Pending when no GPU nodes available | P1 |

## Dashboard Discovery (TC-DISC)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-DISC-001](TC-DISC-001.md) | Dashboard GPU runtime discovery | P0 |

## Build and Image (TC-BUILD)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-BUILD-001](TC-BUILD-001.md) | CSV relatedImages and template image digest cross-reference | P1 |
| [TC-BUILD-002](TC-BUILD-002.md) | Two-image architecture separation | P1 |

## Performance (TC-PERF)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-PERF-001](TC-PERF-001.md) | GPU performance baseline with CPU comparison and memory monitoring | P2 |

## RBAC (TC-RBAC)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-RBAC-001](TC-RBAC-001.md) | Unprivileged user cannot modify ServingRuntime in redhat-ods-applications | P1 |

## Upgrade Testing (TC-UPGRADE)

| Test Case ID | Title | Priority |
|-------------|-------|----------|
| [TC-UPGRADE-001](TC-UPGRADE-001.md) | GPU template appears after operator upgrade | P0 |
| [TC-UPGRADE-002](TC-UPGRADE-002.md) | CPU InferenceService recovers after operator upgrade | P0 |
| [TC-UPGRADE-003](TC-UPGRADE-003.md) | Rollback removes GPU template without CPU impact | P1 |
