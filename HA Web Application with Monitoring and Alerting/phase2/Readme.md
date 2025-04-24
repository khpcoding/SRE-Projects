
---

## 📊 Phase 2: Monitoring with Prometheus and Grafana

In this phase, we'll add **Prometheus** for metrics collection and **Grafana** for visualization. We'll also update the Flask app to expose Prometheus-compatible metrics.

---

### ✅ Goal

- Collect real-time metrics from each application instance
- Visualize data using Grafana dashboards
- Monitor traffic, response counts, and availability

---

## 🧩 Step 2.1: Add Prometheus Metrics to Flask App

Update the app to expose a `/metrics` endpoint.

### 1. Update `requirements.txt`:

```text
flask
prometheus_client
```

### 2. Modify `app/app.py`:

```python
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

---

## 🐳 Step 2.2: Add Prometheus and Grafana to Docker Compose

Update your `docker-compose.yml` to include these services.

### Add to `docker-compose.yml`:

```yaml
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
    depends_on:
      - app1
      - app2

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    depends_on:
      - prometheus

volumes:
  grafana-data:
```

---

## ⚙️ Step 2.3: Create Prometheus Configuration

Create the config file Prometheus will use to scrape metrics.

### 1. Create folder if not already:
```bash
mkdir -p monitoring
```

### 2. Create `monitoring/prometheus.yml`:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'app'
    static_configs:
      - targets: ['app1:5000', 'app2:5000']
```

This tells Prometheus to scrape metrics from both Flask app containers on port 5000 (where `/metrics` is exposed).

---

## 🚀 Step 2.4: Rebuild and Start All Services

```bash
docker-compose down
docker-compose up --build -d
```

---

## 🔍 Step 2.5: Access Prometheus and Grafana

- **Prometheus UI:** [http://localhost:9090](http://localhost:9090)
  - Query `app_requests_total` to see request counts

- **Grafana UI:** [http://localhost:3000](http://localhost:3000)
  - Default login: `admin / admin`
  - Add Prometheus as a data source:
    - URL: `http://prometheus:9090`
  - Import dashboards (e.g., use Prometheus ID `3662` from Grafana.com)

---

## 🧪 Step 2.6: Test the Monitoring Stack

1. Open [http://localhost:8080](http://localhost:8080) and refresh a few times
2. In Prometheus, search `app_requests_total` → values should increase
3. In Grafana, create a panel to visualize `app_requests_total`

---

## ✅ Phase 2 Complete!

You’ve now successfully:

- Exposed metrics from a Flask app
- Collected them with Prometheus
- Visualized them in Grafana

Your system is now **observable** and ready for **alerting**, which we’ll cover in the next phase.

---
