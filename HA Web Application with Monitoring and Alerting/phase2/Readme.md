
---

```markdown
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

#### Modify `app/app.py`

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

### Step 2.2: Update `docker-compose.yml` with Monitoring Stack

Add **Prometheus** and **Grafana** services.

#### Updated `docker-compose.yml`

Add below services:

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

### Step 2.3: Add Prometheus Configuration

#### Create `monitoring/prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'app'
    static_configs:
      - targets: ['app1:5000', 'app2:5000']
```

---

### Step 2.4: Rebuild and Restart Services

```bash
docker-compose down
docker-compose up --build -d
```

---

### Step 2.5: Access Monitoring Dashboards

- **Prometheus UI:** [http://localhost:9090](http://localhost:9090)
- **Grafana UI:** [http://localhost:3000](http://localhost:3000)
  - Default credentials: `admin` / `admin`
  - Add Prometheus as a data source
  - Import a dashboard (e.g., Prometheus 2.0 dashboard ID: `3662` from Grafana.com)

---

### Step 2.6: Test Metrics Collection

- Access your app several times at [http://localhost:8080](http://localhost:8080)
- Go to Prometheus → `http://localhost:9090` → Search for metric: `app_requests_total`
- You should see counts from both app instances

---

## ✅ Phase 2 Complete!

You’ve successfully implemented:

- Prometheus metrics in your Flask app
- A Prometheus service scraping metrics from all app containers
- Grafana dashboard to visualize app metrics

---
