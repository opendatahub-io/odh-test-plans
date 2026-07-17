---
test_case_id: TC-PERF-003
source_key: RHAISTRAT-1868
priority: P2
status: Draft
automation_status: Not Started
last_updated: "2026-07-17"
upgrade_phase: post
---
# TC-PERF-003: GPU memory utilization under sustained load

**Objective**: Monitor GPU memory utilization during sustained
inference load on `mlserver-cuda-runtime` to verify no GPU memory
leaks occur and resource consumption remains stable.

**Preconditions**:

- OCP 4.20+ cluster with RHOAI 3.5 GA and NVIDIA GPU Operator 12.9+
- GPU InferenceService `resnet-gpu` deployed with
  `mlserver-cuda-runtime` and Ready (TC-DEPLOY-002)
- Access to run `nvidia-smi` on the GPU node or inside the pod

**Test Steps**:

1. Record baseline GPU memory usage before load:

   ```bash
   POD=$(oc get pod \
     -l serving.kserve.io/inferenceservice=resnet-gpu \
     -o jsonpath='{.items[0].metadata.name}')
   oc exec "$POD" -c kserve-container -- nvidia-smi \
     --query-gpu=memory.used,memory.total \
     --format=csv,noheader
   ```

2. Run sustained load at 50 concurrent requests for 300 seconds.
3. Sample GPU memory usage every 30 seconds during the load test:

   ```bash
   oc exec "$POD" -c kserve-container -- nvidia-smi \
     --query-gpu=memory.used,memory.total,utilization.gpu \
     --format=csv,noheader
   ```

4. Record GPU memory usage 60 seconds after load stops.

**Expected Results**:

- GPU memory usage stabilizes during sustained load (no monotonic
  increase across samples)
- GPU memory usage after the load test ends returns to within 10%
  of the pre-load baseline
- No GPU out-of-memory (OOM) errors in pod logs during the test
- GPU utilization percentage is recorded for each sample interval

**Notes**: To be filled later in the process.
