---
test_case_id: TC-PERF-002
source_key: RHAISTRAT-1868
priority: P2
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-PERF-002: GPU vs CPU inference throughput comparison baseline

**Objective**: Collect comparative throughput measurements between GPU
(`mlserver-cuda-runtime`) and CPU (`mlserver-onnx`) runtimes to
validate the performance benefit of GPU acceleration. No pass/fail
thresholds — baseline establishment only until SLOs are defined.

**Preconditions**:
- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator 12.9+
- GPU InferenceService `resnet-gpu` deployed with
  `mlserver-cuda-runtime` and Ready (TC-DEPLOY-002)
- CPU InferenceService `resnet-cpu` deployed with `mlserver-onnx`
  and Ready (TC-COMPAT-001)
- Same ResNet-50 ONNX model loaded on both runtimes

**Test Steps**:
1. Run a throughput test against the GPU runtime at 50 concurrent
   requests for 120 seconds. Record total requests completed and
   average latency.
2. Run the identical throughput test against the CPU runtime.
3. Calculate the throughput ratio (GPU requests/sec vs CPU
   requests/sec).
4. Compare average and p95 latencies between runtimes.

**Expected Results**:
- Both runtimes maintain error rates below 1% during the test
- Throughput and latency metrics are recorded for both GPU and CPU
  runtimes for comparison
- GPU runtime is expected to show higher throughput but no specific
  improvement targets are defined (see TestPlanGaps.md)
- Metrics are documented for baseline establishment

**Notes**: To be filled later in the process.
