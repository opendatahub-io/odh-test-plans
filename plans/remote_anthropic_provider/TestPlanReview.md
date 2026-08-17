---
feature: remote_anthropic_provider
source_key: RHAISTRAT-1246
score: 8
pass: true
verdict: Ready
scores:
  specificity: 2
  grounding: 1
  scope_fidelity: 2
  actionability: 1
  consistency: 2
auto_revised: true
last_updated: '2026-08-14'
before_score: 8
before_scores:
  specificity: 2
  grounding: 1
  scope_fidelity: 2
  actionability: 1
  consistency: 2
error: null
---
## Rubric Scores

| Criterion | Score | Notes |
|-----------|-------|-------|
| Specificity | 2/2 | P0 priorities name Anthropic-specific scenarios; risks cite specific dependencies (AIPCC wheel pipeline, RHAISTRAT-1245 coordination, migration from remote::openai workaround). Swap test passes. |
| Grounding | 1/2 | Section 4 entries are traceable to strategy text, but kubectl/oc CLI is inferred rather than explicitly named. Section 7.5 secret rotation testing and api.anthropic.com endpoint hostname have no direct strategy source. |
| Scope Fidelity | 2/2 | All 5 ACs covered with valid citations (7/7 cited, 0 uncited, 0 invalid). No scope creep. Out-of-scope items match strategy exactly. |
| Actionability | 1/2 | Concrete details include RHOAI 3.5, ANTHROPIC_API_KEY, TLS 1.2+, Kubernetes Secrets, AIPCC wheel pipeline. Gaps remain: Test Users is TBD, no specific OpenShift version, no sample YAML/JSON snippets, Python 3.12 uncertain. |
| Consistency | 2/2 | All 7 interfaces in Section 4 listed in Section 9.2. Section 1.2 in-scope items map to Section 4 interfaces. No contradictions found across sections. |

**Total: 8/10 -- Verdict: Ready**

## Grounding Cross-Reference

| Section 4 Entry | Source Match | Status |
|-----------------|-------------|--------|
| `/v1/models` | Strategy: "Properly implements model listing using AsyncAnthropic SDK (Anthropic's /v1/models endpoint returns different format than OpenAI)"; AC #4: "Given a user requests model listing via /v1/models" | GROUNDED |
| `/v1/providers` | Strategy AC #5: "The remote::anthropic provider is listed in the output of /v1/providers endpoint on a deployed OGX distribution." | GROUNDED |
| Chat completions API | Strategy AC #3: "Anthropic chat completions API works with streaming and temperature parameters"; Workaround section: "Chat completions API (with and without streaming)" | GROUNDED |
| Responses API | Strategy workaround section: "Responses API (with and without streaming)"; AC #4: "Tool calling via chat completions and responses APIs works correctly" | GROUNDED |
| `x-ogx-provider-data` header | Strategy High Level Requirements: "[P1] Per-request API key override via x-ogx-provider-data header with anthropic_api_key field" | GROUNDED |
| OGX K8s Operator CR spec | Strategy Technical Approach: "The Anthropic provider is activated through environment variables set in the user's ConfigMap or CR spec" | GROUNDED |
| `kubectl`/`oc` CLI | Strategy implies OpenShift cluster operations throughout but does not explicitly name kubectl/oc as a testable interface. | INFERRED |

## Section-by-Section Feedback

### Grounding (1/2)

**Section 4 (Interfaces Under Test):** The `kubectl`/`oc` CLI entry is a reasonable inference but is not explicitly named as a testable interface in the strategy. Either add a citation to a specific strategy sentence that implies cluster CLI operations, or mark this interface as "inferred from deployment context" to distinguish it from directly grounded entries.

**Section 7.5 (NFR -- Security):** "Secret rotation: changing the ANTHROPIC_API_KEY environment variable takes effect without requiring pod restart" is not traceable to a specific strategy sentence. Either cite the strategy source that implies this behavior, or mark it as an additional recommended test objective beyond strategy scope.

**Section 4 (Interfaces Under Test):** The `api.anthropic.com` endpoint hostname does not appear in the strategy text. If this is inferred from general Anthropic documentation, note the source. If it comes from the strategy's mention of the AsyncAnthropic SDK, cite that sentence.

### Actionability (1/2)

**Section 5 (Test Environment):** Test Users is listed as TBD. Specify the RBAC roles needed (e.g., cluster-admin for operator deployment, namespace-scoped user for API testing) and whether a dedicated service account is required for API key management.

**Section 5 (Test Environment):** No specific OpenShift version is stated. Add the minimum supported OCP version for RHOAI 3.5 (e.g., OCP 4.14+) so QE can provision the correct cluster.

**Section 5 (Test Data):** No sample YAML or JSON snippets are provided for ConfigMap, CR spec, or API request payloads. Including at least one representative example for each interface type would reduce QE ramp-up time.

**Section 5 (Test Environment):** Python 3.12 is marked as "needs validation." Confirm the supported Python version from the AIPCC wheel pipeline or mark this as a gap that blocks test environment provisioning.

## Revision History

Initial assessment

### Cycle 1 Revision
- **Specificity**: N/A -- scored 2
- **Grounding**: Marked `kubectl`/`oc` CLI as inferred in Section 4 and Section 9.2. Removed `api.anthropic.com` hostname from Section 3.1 (not in strategy). Removed "Secret rotation" bullet from Section 7.5 (no strategy source).
- **Scope Fidelity**: N/A -- scored 2
- **Actionability**: Section 3.1: added minimum OpenShift version as TBD with rationale, clarified Python runtime as TBD pending AIPCC validation. Section 3.2: added sample YAML snippets for native provider config, legacy workaround config, per-request override header, and chat completion payload using strategy examples. Section 3.3: replaced TBD with concrete roles (cluster administrator, namespace-scoped user, API consumer, service account TBD) inferred from strategy operations. Section 3.4: specified concrete tools (`oc`, `curl`, `jq`, `openssl s_client`, pytest) with example commands.
- **Consistency**: N/A -- scored 2
