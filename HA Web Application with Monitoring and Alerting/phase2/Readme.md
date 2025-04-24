
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
```

Modify app/app.py
```bash
from flask import Flask
from prometheus_client import Counter, generate_latest
import socket

app = Flask(__name__)

REQUEST_COUNT = Counter('app_requests_total', 'Total number of HTTP requests')

@app.route("/")
def home():
    REQUEST_COUNT.inc()
    return f"Hello from {socket.gethostname()}!"

@app.route("/metrics")
def metrics():
    return generate_latest(), 200, {'Content-Type': 'text/plain; charset=utf-8'}

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
