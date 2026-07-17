---
feature: gpu_enabled_runtimes
source_key: RHAISTRAT-1868
score: 9
pass: true
verdict: Ready
scores:
  specificity: 2
  grounding: 1
  scope_fidelity: 2
  actionability: 2
  consistency: 2
auto_revised: false
last_updated: '2026-07-17'
before_score: 8
before_scores:
  specificity: 2
  grounding: 1
  scope_fidelity: 2
  actionability: 2
  consistency: 1
error: null
---
# Test Plan Review: gpu_enabled_runtimes

## Score Summary

| Criterion | Score | Max | Evidence | Notes |
|-----------|-------|-----|----------|-------|
| Specificity | 2 | 2 | P0 definitions name GPU-specific scenarios. Risks name CUDA EP build-time config, ResNet-50 S3 staging, GPU image size, OVMS deprecation. Swap test passes. | No regression. |
| Grounding | 1 | 2 | Probe entries now have transparency notes citing context doc; `additional_docs` lists RHAISTRAT-1868-context.md. However, runtime version "1.7.1", HardwareProfile resource ranges, and container resource defaults remain unsourced. Container defaults (CPU 1-4, Memory 4Gi-8Gi) vs HardwareProfile defaults (CPU 4, Memory 16Gi) discrepancy not explained. | Revision improved probe grounding with transparency notes. Score remains 1 due to other unsourced details that cannot be verified without strategy text. |
| Scope Fidelity | 2 | 2 | All in-scope items mapped. No scope creep. Out-of-scope items absent from endpoints. | No regression. |
| Actionability | 2 | 2 | OCP 4.20+, RHOAI 3.5 GA, GPU Operator 12.9+ all confirmed. Probes with exact timing. HardwareProfile fully specified. Test data concrete. Tools named. Users defined. | No regression. |
| Consistency | 2 | 2 | All six cross-checks pass. Section 6.2 now includes Readiness probe and Liveness probe rows mapped to TC-E2E-001/002. Section 10.2 omits 3 probe entries but this is cosmetic, not contradictory. All P0 endpoints have E2E coverage. | Improved from 1 to 2. Revision successfully addressed the gap. |
| **Total** | **9** | **10** | | |

**Verdict: Ready**

## Grounding Cross-Reference

| Section 4 Entry | Source Match | Status |
|-----------------|-------------|--------|
| `mlserver-cuda-runtime-template` | Internally consistent | Grounded |
| HardwareProfile CR (`nvidia-gpu-a100`) | Resource ranges unsourced | Extrapolated |
| GPU MLServer container image | Consistent with feature description | Grounded |
| CPU MLServer container image | Backwards compatibility | Grounded |
| KServe V2 `/v2/models/{model}/infer` (GPU) | Standard V2 protocol | Grounded |
| KServe V2 `/v2/models/{model}/ready` (GPU) | Standard V2 protocol | Grounded |
| Readiness probe (HTTP) | Transparency note cites context doc, listed in additional_docs | Grounded |
| Liveness probe (HTTP) | Transparency note cites context doc, listed in additional_docs | Grounded |
| GPU pod scheduling behavior | Feature scope | Grounded |
| KServe V2 `/v2/models/{model}/infer` (CPU) | Regression endpoint | Grounded |
| Dashboard label | ODH annotation pattern | Grounded |
| Recommended-accelerators annotation | Feature scope | Grounded |
| model-settings.json | Upstream MLServer schema | Grounded |
| KServe V2 `/v2/health/ready` (GPU) | Standard V2 protocol | Grounded |
| KServe V2 `/v2/models/{model}/ready` (CPU) | Standard V2 protocol | Grounded |
| Startup probe (exec) | Transparency note cites context doc, listed in additional_docs | Grounded |
| CSV relatedImages | Feature scope | Grounded |
| kustomization.yaml | Feature scope | Grounded |
| Environment variables | MLServer conventions, minor extrapolation | Minor extrapolation |
| Security context | Feature scope | Grounded |
| CUDA EP init logs | Reasonable verification | Grounded |

## Section-by-Section Feedback

### Grounding (Score: 1/2)

**What improved in Cycle 1:** The revision added transparency notes to probe entries (Readiness, Liveness, Startup) citing the context document. The `additional_docs` frontmatter field now lists `RHAISTRAT-1868-context.md`, establishing provenance for those entries.

**What still needs attention:**

1. **Runtime version "1.7.1"** -- The test plan references MLServer runtime version 1.7.1 but this version number does not appear in the available source material. Without the full strategy text to confirm, this remains unsourced.

2. **HardwareProfile resource ranges** -- The `nvidia-gpu-a100` HardwareProfile CR specifies resource limits and requests (e.g., GPU count, memory ranges) that are not traceable to the strategy or any linked documentation. These values may be reasonable defaults, but they are extrapolated rather than grounded.

3. **Container resource defaults discrepancy** -- The plan specifies container defaults of CPU 1-4 and Memory 4Gi-8Gi in one location, while the HardwareProfile defaults specify CPU 4 and Memory 16Gi in another. This inconsistency is not explained or sourced. If both values are correct for different contexts (container-level vs node-level), the plan should clarify the relationship.

4. **Environment variables** -- Some MLServer environment variables referenced in the test plan follow upstream conventions but include minor extrapolation regarding exact variable names and default values.

**Recommendation:** If a future revision cycle is undertaken, source these values from the strategy document or add transparency notes explaining the basis for extrapolation. The current score of 1 reflects that the majority of entries are grounded, but enough unsourced specifics remain to prevent a score of 2.

## Consistency Cross-Checks

| Check | Result |
|-------|--------|
| Section 4 vs Section 1.2 scope | PASS |
| Section 2.1 vs Section 4 interface types | PASS |
| Section 4 priorities vs Section 2.3 | PASS |
| Section 10.2 vs Section 4 endpoints | PASS (cosmetic gap: 3 probe entries) |
| Section 7 NFR categories vs feature scope | PASS |
| Section 6.2 E2E coverage vs Section 4 P0 endpoints | PASS (all P0 covered) |

## Revision History

| Date | Version | Change |
|------|---------|--------|
| 2026-07-17 | 1.0 | Initial assessment |

### Cycle 1 Revision

- **Specificity**: N/A -- scored 2
- **Grounding**: Added transparency notes to probe entries in Section 4 indicating they are sourced from context document's ServingRuntime template specification
- **Scope Fidelity**: N/A -- scored 2
- **Actionability**: N/A -- scored 2
- **Consistency**: Added Readiness probe and Liveness probe rows to Section 6.2 E2E Coverage Matrix mapping to TC-E2E-001 and TC-E2E-002

| Cycle | Action | Score | Notes |
|-------|--------|-------|-------|
| 1 | Re-assessment after Cycle 1 revision | 9/10 | Consistency improved from 1 to 2. Grounding remains at 1 due to unsourced runtime version, HardwareProfile ranges, and container defaults. Overall verdict: Ready. |
