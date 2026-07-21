---
test_case_id: TC-PERF-001
source_key: RHAISTRAT-1868
priority: P2
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-PERF-001: GPU performance baseline with CPU comparison and memory monitoring

**Objective**: Establish baseline GPU inference latency measurements,
compare GPU vs CPU performance under identical load patterns, and
monitor GPU memory usage via DCGM/Prometheus metrics. Performance
SLOs are not yet defined -- this test collects metrics for future
SLO establishment only, with no pass/fail assertions on latency
values.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator
  12.9+
- GPU and CPU ServingRuntimes available in
  `redhat-ods-applications`
- DCGM exporter or Prometheus GPU metrics endpoint accessible
- `pytest.mark.slow` marker applied (test runs 300+ second load
  sessions)

**Test Steps**:

1. Deploy a GPU InferenceService and a CPU InferenceService:

   ```python
   gpu_isvc = create_isvc(
       client=admin_client,
       name="resnet-gpu",
       namespace=project.name,
       runtime=ModelInferenceRuntime.MLSERVER_CUDA_RUNTIME,
       model_format="sklearn",
       gpu_count=1,
       external_route=True,
   )
   cpu_isvc = create_isvc(
       client=admin_client,
       name="resnet-cpu",
       namespace=project.name,
       runtime=ModelInferenceRuntime.MLSERVER_RUNTIME,
       model_format="sklearn",
       external_route=True,
   )
   gpu_url = get_exposed_isvc_url(gpu_isvc)
   cpu_url = get_exposed_isvc_url(cpu_isvc)
   ```

2. Record single-request GPU latency baseline. Send 100 sequential
   requests via `send_rest_request` with timing. Compute p50, p95,
   p99 latencies:

   ```python
   latencies = []
   for _ in range(100):
       start = time.monotonic()
       response = send_rest_request(
           url=gpu_url,
           data=inference_payload,
       )
       elapsed = time.monotonic() - start
       latencies.append(elapsed)
   p50 = numpy.percentile(latencies, 50)
   p95 = numpy.percentile(latencies, 95)
   p99 = numpy.percentile(latencies, 99)
   ```

3. Run concurrent load test on GPU ISVC using
   `concurrent.futures.ThreadPoolExecutor`. Test at concurrency
   levels of 10, 50, and 100 for 60 seconds each. Record p50, p95,
   p99 latencies and error rates at each level:

   ```python
   from concurrent.futures import ThreadPoolExecutor, as_completed

   def send_timed_request(url, payload):
       start = time.monotonic()
       resp = send_rest_request(url=url, data=payload)
       return time.monotonic() - start, resp.status_code

   with ThreadPoolExecutor(max_workers=concurrency) as pool:
       futures = [
           pool.submit(send_timed_request, gpu_url, payload)
           for _ in range(num_requests)
       ]
       for f in as_completed(futures):
           elapsed, status = f.result()
           # collect latencies and error counts
   ```

4. CPU comparison -- repeat the same load pattern (step 3) against
   the CPU ISVC at each concurrency level. Record the same p50,
   p95, p99 metrics. Compute the GPU-to-CPU latency ratio at each
   concurrency level.

5. GPU memory monitoring -- query DCGM/Prometheus metrics for GPU
   memory utilization during the load test. If Prometheus is not
   accessible, fall back to node-level commands:

   ```bash
   oc adm top node --selector=nvidia.com/gpu.present=true
   oc debug node/<gpu-node> -- nvidia-smi --query-gpu=memory.used,memory.total --format=csv
   ```

   Record peak GPU memory usage and utilization percentage.

**Expected Results**:

- All requests at each concurrency level return HTTP 200 with valid
  inference output (error rate below 1%)
- p50, p95, p99 latency metrics are recorded for both GPU and CPU
  ISVCs at each concurrency level
- GPU-to-CPU latency ratio is computed and documented
- Peak GPU memory usage is recorded
- No server crashes or unresponsive periods during load tests
- No pass/fail thresholds applied -- baseline metrics recording
  only

**Notes**: This test requires `pytest.mark.slow` due to 300+ second
load sessions. Locust and k6 are not used because they are not
available in the `opendatahub-tests` repository. The
`concurrent.futures.ThreadPoolExecutor` approach provides sufficient
concurrency for baseline collection.
