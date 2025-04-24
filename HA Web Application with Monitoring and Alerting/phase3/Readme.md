
---

## 🚨 Phase 3: Alerting with Prometheus and Alertmanager

In this phase, we’ll set up **Prometheus Alertmanager** to send alerts when things go wrong (e.g. app is down, endpoint unresponsive, etc.). Alerts can be routed to **Slack**, **email**, or other notification systems.

---

## ✅ Goal

- Detect failures or abnormal conditions automatically
- Notify on-call engineers via Slack or email
- Establish the foundation for incident response

---

## 📦 Step 3.1: Add Alertmanager to Docker Compose

Update `docker-compose.yml` to include the Alertmanager service.

### 🔧 Add to `docker-compose.yml`

```yaml
  alertmanager:
    image: prom/alertmanager
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./monitoring/alertmanager.yml:/etc/alertmanager/alertmanager.yml
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
```

Also, update Prometheus service to point to Alertmanager:

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
      - alertmanager
```

---

## ⚙️ Step 3.2: Configure Alertmanager

### Create `monitoring/alertmanager.yml`:

```yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'slack-notifications'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#alerts'
        send_resolved: true
```

> 💡 Replace `YOUR/SLACK/WEBHOOK` with your actual Slack webhook URL. You can create a Slack webhook from [Slack API > Incoming Webhooks](https://api.slack.com/messaging/webhooks).

---

## 📡 Step 3.3: Define Alert Rules in Prometheus

### Update `monitoring/prometheus.yml`:

Add alerting and rules configuration:

```yaml
global:
  scrape_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

rule_files:
  - "alert_rules.yml"

scrape_configs:
  - job_name: 'app'
    static_configs:
      - targets: ['app1:5000', 'app2:5000']
```

---

### Create Alert Rules File

Create `monitoring/alert_rules.yml`:

```yaml
groups:
  - name: instance-down
    rules:
      - alert: AppInstanceDown
        expr: up == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "Prometheus target {{ $labels.instance }} has been down for more than 30 seconds."
```

This alert fires if any app container is not responding for more than 30 seconds.

---

## 🚀 Step 3.4: Restart and Apply Changes

Rebuild and bring up all services:

```bash
docker-compose down
docker-compose up --build -d
```

---

## 🔍 Step 3.5: Verify Alerting

1. Visit Prometheus Alerts: [http://localhost:9090/alerts](http://localhost:9090/alerts)
2. Stop `app1` manually using `docker stop app1`
3. Wait 30–60 seconds, then check:
   - Prometheus UI: Should show "AppInstanceDown"
   - Alertmanager UI: [http://localhost:9093](http://localhost:9093)
   - Slack: Should receive an alert message (if webhook is configured)

---

## ✅ Phase 3 Complete!

You now have:

- 🚨 Alertmanager configured
- 🔔 Alert rules defined for service availability
- 📬 Slack integrated for real-time alerts

This is the foundation for **incident detection and response** in an SRE workflow.

---

