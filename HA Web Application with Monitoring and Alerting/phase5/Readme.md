
---

## 📈 Phase 5: SLO Dashboards – SLIs, SLOs & Error Budgets

In this phase, we define and visualize key **SRE reliability metrics**: SLIs, SLOs, and Error Budgets. These help quantify the reliability of your service and drive incident management, prioritization, and engineering velocity.

---

## ✅ Goal

- Define **SLIs**: e.g. request success rate, latency
- Set **SLOs**: target levels for availability
- Track **error budgets** and show burn rates
- Visualize SLOs and alerts using **Grafana**

---

## 🧠 Step 5.1: Understand the Terminology

- **SLI (Service Level Indicator)**: A measurable metric, e.g., "99% of requests are successful"
- **SLO (Service Level Objective)**: A target goal for an SLI, e.g., "99.9% uptime in 30 days"
- **Error Budget**: The acceptable amount of failure, e.g., "0.1% of time can fail in a month"

---

## 📊 Step 5.2: Define SLIs with Prometheus Metrics

Assume:
- `app_requests_total` is the total number of HTTP requests
- `app_requests_failed_total` is a custom metric you'll create for failed requests

You can add this to your Flask app:

```python
from prometheus_client import Counter

REQUESTS = Counter('app_requests_total', 'All requests')
FAILED_REQUESTS = Counter('app_requests_failed_total', 'Failed requests')

@app.route("/")
def index():
    try:
        REQUESTS.inc()
        # Simulate normal logic
        return "Hello"
    except Exception:
        FAILED_REQUESTS.inc()
        raise
```

> You can simulate failures or add logic to define what counts as a "failure" (e.g., HTTP 5xx).

---

## 📏 Step 5.3: Create SLO Recording Rules

Update or create `monitoring/slo_rules.yml`:

```yaml
groups:
  - name: slo.rules
    rules:
      - record: slo:request_success_ratio
        expr: (sum(rate(app_requests_total[5m])) - sum(rate(app_requests_failed_total[5m]))) / sum(rate(app_requests_total[5m]))

      - alert: SLOViolation
        expr: slo:request_success_ratio < 0.999
        for: 5m
        labels:
          severity: high
        annotations:
          summary: "SLO Violation: success ratio below target"
          description: "The request success ratio dropped below 99.9% over the last 5 minutes."
```

Then update `prometheus.yml` to include it:

```yaml
rule_files:
  - "alert_rules.yml"
  - "slo_rules.yml"
```

---

## 📉 Step 5.4: Visualize in Grafana

### 1. In Grafana → Create New Dashboard:
- Add **Time Series Panel**
- Query: `slo:request_success_ratio`
- Set thresholds:
  - Green: above 0.999
  - Yellow: 0.995–0.999
  - Red: below 0.995

### 2. Create a Stat Panel:
- Query: `1 - slo:request_success_ratio`
- Title: `Error Budget Burned`
- Multiply by 100 for percentage using `* 100`

---

## 🔔 Step 5.5: Alerting on SLO Burn

Already handled via `SLOViolation` rule in Prometheus → Alertmanager → Slack (Phase 3)

---

## 📚 Step 5.6: Error Budget Policy (Optional)

Use the dashboard to enforce an error budget policy. For example:

> If more than 25% of your monthly error budget is burned in a day, freeze feature deployments.

You can model error budget as:
```prometheus
error_budget_remaining = 1 - slo:request_success_ratio
```

---

## ✅ Phase 5 Complete!

You now have:

- 🎯 SLOs based on actual usage
- 📊 Dashboards to track error budgets
- 🔔 Alerting when SLOs are violated

This forms the foundation of **SRE-driven reliability management** — tying observability to business goals and engineering priorities.

---



