# GPU Enabled Runtimes for Predictive Machine Learning

NVIDIA GPU-accelerated inference for predictive ML workloads via a
separate MLServer GPU container image (`mlserver-cuda-runtime`) and
ServingRuntime template.

**Last modified**: 2026-07-21

## Links

- **Strategy**: [RHAISTRAT-1868](https://issues.redhat.com/browse/RHAISTRAT-1868)
- **Test Plan**: [TestPlan.md](TestPlan.md)
- **Review**: [TestPlanReview.md](TestPlanReview.md)
- **Gaps**: [TestPlanGaps.md](TestPlanGaps.md)

## Test Cases

- **Test Case Index**: [test_cases/INDEX.md](test_cases/INDEX.md)
- **Total**: 17 test cases (8 P0, 8 P1, 1 P2)
- **Categories**: DEPLOY, INFER, COMPAT, FALLBACK, DISC, BUILD, PERF,
  RBAC, UPGRADE

## Test Implementation

Automated tests will be implemented in:

- `opendatahub-tests` repository using `ocp-resources` Python library,
  `ServingRuntimeFromTemplate`, `create_isvc` fixtures
- TC-DISC-001 is manual (Dashboard UI — incompatible with pytest)

## Changelog

### v1.3.0 (2026-07-21)

- PR review feedback: consolidated 41 to 17 TCs (redundancy analysis)
- Corrected resource kind to namespace-scoped ServingRuntime
- Corrected CPU runtime name to mlserver-runtime
- Fixed CSV namespace to redhat-ods-operator
- Corrected CUDA EP configurability (overridable via env var)
- Fixed silent CPU fallback behavior (Running, not Pending)
- Aligned all TCs with opendatahub-tests framework conventions
- Removed AIRGAP and E2E categories

### v1.2.0 (2026-07-17)

- Updated with RHAISTRAT-1868-context.md
- Corrected runtime name from `mlserver-onnx-gpu` to
  `mlserver-cuda-runtime`
- Added HardwareProfile specs, probe timings, security context details
- Resolved 16 gaps (7 remain open)
- Regenerated all 41 test cases with updated specifications
- Review score: 9/10
