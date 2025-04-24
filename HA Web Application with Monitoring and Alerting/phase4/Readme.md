
---

## 📦 Phase 4: Centralized Logging with Loki and Grafana

In this phase, we’ll add **Loki**, a log aggregation system from Grafana Labs, and configure **Grafana** to visualize logs alongside metrics. This helps with root cause analysis during incidents and monitoring application logs in real time.

---

## ✅ Goal

- Collect logs from all app containers
- Centralize logs using Loki
- Visualize logs inside Grafana
- Correlate logs with metrics for faster debugging

---

## 🧰 Step 4.1: Update `docker-compose.yml` to Add Loki and Promtail

We'll use **Loki** to store logs and **Promtail** to collect logs from containers.

### Add to `docker-compose.yml`:

```yaml
  loki:
    image: grafana/loki:2.9.0
    container_name: loki
    ports:
      - "3100:3100"
    volumes:
      - ./monitoring/loki-config.yml:/etc/loki/loki-config.yml
    command: -config.file=/etc/loki/loki-config.yml

  promtail:
    image: grafana/promtail:2.9.0
    container_name: promtail
    volumes:
      - /var/log:/var/log
      - /var/lib/docker/containers:/var/lib/docker/containers
      - ./monitoring/promtail-config.yml:/etc/promtail/promtail-config.yml
      - /etc/hostname:/etc/hostname
      - /etc/localtime:/etc/localtime
      - /var/run/docker.sock:/var/run/docker.sock
    command: -config.file=/etc/promtail/promtail-config.yml
```

> 🛑 Make sure Docker has access to `/var/lib/docker/containers` and log files. You might need to run with elevated permissions or adjust Docker’s log driver settings.

---

## ⚙️ Step 4.2: Loki Configuration

### Create `monitoring/loki-config.yml`:

```yaml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9095

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
  chunk_idle_period: 3m
  max_chunk_age: 1h
  chunk_target_size: 1048576

schema_config:
  configs:
    - from: 2022-01-01
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /tmp/loki/index
    cache_location: /tmp/loki/cache
    shared_store: filesystem
  filesystem:
    directory: /tmp/loki/chunks

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h
```

---

## 📋 Step 4.3: Promtail Configuration

### Create `monitoring/promtail-config.yml`:

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: docker
    static_configs:
      - targets:
          - localhost
        labels:
          job: container-logs
          __path__: /var/lib/docker/containers/*/*.log
```

---

## 📊 Step 4.4: Configure Loki as a Data Source in Grafana

1. Open Grafana: [http://localhost:3000](http://localhost:3000)
2. Go to **Settings → Data Sources → Add Data Source**
3. Choose **Loki**
4. Set URL: `http://loki:3100`
5. Click **Save & Test**

---

## 🔎 Step 4.5: Explore Logs in Grafana

- Go to **Explore** tab in Grafana
- Select **Loki** as the data source
- Use a query like:
  ```logql
  {job="container-logs"}
  ```
- You’ll see real-time logs from your app containers

---

## ✅ Phase 4 Complete!

You now have:

- 🪵 Centralized logs using Promtail → Loki
- 📊 Logs visualized alongside metrics in Grafana
- 🧠 Enhanced observability and debugging ability

This gives your SRE stack **metrics + logs** in one place. You're now ready to move toward more advanced reliability techniques.

---

