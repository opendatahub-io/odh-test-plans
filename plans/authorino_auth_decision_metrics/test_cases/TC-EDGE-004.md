---
test_case_id: TC-EDGE-004
source_key: RHAISTRAT-2418
priority: P2
status: Draft
automation_status: Not Started
last_updated: "2026-08-14"
---
# TC-EDGE-004: Verify multi-replica metric aggregation

**Objective**: Confirm that recording rules correctly aggregate
auth decision metrics across multiple Authorino replicas using
`sum by()` grouping.

**Preconditions**:

- Authorino deployment scaled to at least 2 replicas
- Auth traffic distributed across replicas

**Test Steps**:

1. Scale Authorino to 2 replicas:

   ```bash
   kubectl scale deployment/authorino \
     -n <authorino-namespace> --replicas=2
   ```

2. Wait for both pods to be ready.
3. Generate auth traffic for 6 minutes (traffic should be
   load-balanced across replicas).
4. Query raw metrics to verify both targets are scraped:

   ```bash
   curl -s "https://<prometheus-route>/api/v1/query?query=\
     auth_server_authconfig_response_status" | \
     jq '.data.result | length'
   ```

5. Query the recording rule:

   ```bash
   curl -s "https://<prometheus-route>/api/v1/query?query=\
     maas:auth_decisions:rate5m" | jq '.data.result[]'
   ```

6. Verify the recording rule aggregates across both targets
   (result should be grouped by `authconfig, namespace, status`
   — not by `instance` or `pod`).

**Expected Results**:

- Raw metrics show separate series per replica (2x the number
  of series)
- Recording rule `sum by (authconfig, namespace, status)` merges
  replica-level metrics into single per-policy series
- Total rate approximately equals the sum of per-replica rates
- Dashboard panels show aggregated data, not per-replica data

**Notes**: To be filled later in the process.
