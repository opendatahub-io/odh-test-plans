---
test_case_id: TC-ALERT-001
source_key: RHAISTRAT-2418
priority: P0
status: Draft
automation_status: Not Started
last_updated: "2026-08-14"
---
# TC-ALERT-001: Verify alert fires on high deny rate

**Objective**: Confirm that `MaaSHighAuthDenyRate` alert fires when
the deny ratio exceeds 10% and is sustained for 10 minutes.

**Preconditions**:

- PrometheusRule with `MaaSHighAuthDenyRate` alert applied
- Ability to generate denied auth requests (invalid credentials)

**Test Steps**:

1. Generate auth traffic with approximately 20% deny rate for
   15 minutes: 80% valid requests (OK) and 20% invalid requests
   (UNAUTHENTICATED).
2. After 15 minutes, query the Prometheus alerts API:

   ```bash
   curl -s "https://<prometheus-route>/api/v1/alerts" | \
     jq '.data.alerts[] | select(.labels.alertname ==
     "MaaSHighAuthDenyRate")'
   ```

3. Verify the alert is in `firing` state.
4. Verify the alert has `severity: warning` label.
5. Verify the alert annotations include a description mentioning
   the specific `authconfig` and `namespace`.

**Expected Results**:

- `MaaSHighAuthDenyRate` alert appears in the alerts API
- Alert state is `firing`
- Alert labels include `severity: warning`, `authconfig`, and
  `namespace`
- Alert annotation `summary` contains "High auth denial rate
  for an Authorino AuthConfig"
- Alert annotation `description` references the specific
  authconfig hash and namespace

**Notes**: To be filled later in the process.
