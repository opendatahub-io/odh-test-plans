---
test_case_id: TC-PERF-001
source_key: RHAISTRAT-1868
priority: P2
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-PERF-001: GPU inference latency baseline under concurrent requests

**Objective**: Establish baseline GPU inference latency measurements
under increasing concurrency using the `mlserver-cuda-runtime`.
Performance SLOs are not yet defined — this test collects metrics for
future SLO establishment only, with no pass/fail assertions on
latency values.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator 12.9+
- GPU InferenceService `resnet-gpu` deployed with
  `mlserver-cuda-runtime` and Ready (TC-DEPLOY-002)
- Load testing tool available (Locust or k6 recommended)

**Test Steps**:

1. Establish baseline latency with a single request:

   ```bash
   URL=$(oc get inferenceservice resnet-gpu \
     -o jsonpath='{.status.url}')
   curl -s -w "time_total: %{time_total}s\n" -o /dev/null \
     -X POST "${URL}/v2/models/resnet-gpu/infer" \
     -H "Content-Type: application/json" \
     -d @resnet-input.json
   ```

2. Run a load test at 10 concurrent requests for 60 seconds.
3. Run a load test at 50 concurrent requests for 60 seconds.
4. Run a load test at 100 concurrent requests for 60 seconds.
5. Record p50, p95, p99 latencies and error rates at each
   concurrency level.

**Expected Results**:

- All requests at each concurrency level return HTTP `200` with
  valid inference output (error rate below 1%)
- Latency metrics (p50, p95, p99) are recorded for each concurrency
  level for future SLO definition
- No server crashes or unresponsive periods during load tests
- Metrics are documented for baseline establishment (no pass/fail
  thresholds applied — see TestPlanGaps.md)

**Notes**: To be filled later in the process.
