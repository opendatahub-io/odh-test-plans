# GPU Enabled Runtimes for Predictive Machine Learning

NVIDIA GPU-accelerated inference for predictive ML workloads via a
separate MLServer GPU container image (`mlserver-cuda-runtime`) and
ClusterServingRuntime template.

**Last modified**: 2026-07-17

## Links

- **Strategy**: [RHAISTRAT-1868](https://issues.redhat.com/browse/RHAISTRAT-1868)
- **Test Plan**: [TestPlan.md](TestPlan.md)
- **Review**: [TestPlanReview.md](TestPlanReview.md)
- **Gaps**: [TestPlanGaps.md](TestPlanGaps.md)

## Test Cases

- **Test Case Index**: [test_cases/INDEX.md](test_cases/INDEX.md)
- **Total**: 41 test cases (19 P0, 17 P1, 5 P2)
- **Categories**: DEPLOY, INFER, COMPAT, FALLBACK, DISC, BUILD, PERF,
  AIRGAP, RBAC, UPGRADE, E2E

## Test Implementation

Automated tests will be implemented in:
- `opendatahub-tests` repository for E2E and integration tests
- Component-level tests in `odh-model-controller` for template validation

## Changelog

### v1.2.0 (2026-07-17)
- Updated with RHAISTRAT-1868-context.md
- Corrected runtime name from `mlserver-onnx-gpu` to
  `mlserver-cuda-runtime`
- Added HardwareProfile specs, probe timings, security context details
- Resolved 16 gaps (7 remain open)
- Regenerated all 41 test cases with updated specifications
- Review score: 9/10
