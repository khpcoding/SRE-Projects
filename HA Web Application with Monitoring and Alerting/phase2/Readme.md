
## 📊 Phase 2: Add Monitoring with Prometheus and Grafana

In this phase, we'll integrate **Prometheus** for metrics collection and **Grafana** for data visualization. We'll also expose metrics from our web applications using a Prometheus exporter for Python.

---

### Step 2.1: Add Metrics to the Flask App

Update the Flask app to expose Prometheus-compatible metrics.

#### Install Prometheus client

Add it to your `requirements.txt`:

```txt
flask
prometheus_client
