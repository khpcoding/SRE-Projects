
---
## 🛠️ Phase 6: Auto-Healing – Restart Failed Containers Automatically

In this phase, we implement **auto-healing** to ensure failed services are restarted automatically without manual intervention. This is a critical step in achieving a resilient, self-healing infrastructure.

---

## ✅ Goal

- Automatically detect and restart failed containers
- Ensure high availability of core services (App, Prometheus, HAProxy, etc.)
- Add watchdog mechanisms for advanced failure detection (optional)

---

## 🧰 Step 6.1: Use Docker’s Built-in Restart Policies

Update each service in your `docker-compose.yml` to include a `restart:` policy.

### Example: app service

```yaml
  app:
    image: your-app-image
    container_name: app
    ports:
      - "5000:5000"
    restart: always
```

### Recommended Restart Policies

| Policy       | Behavior                                                                 |
|--------------|--------------------------------------------------------------------------|
| `no`         | Never restart (default)                                                  |
| `always`     | Always restart if the container stops                                    |
| `on-failure` | Restart only if the container exits with a non-zero status              |
| `unless-stopped` | Restart unless the container was explicitly stopped by the user     |

### Apply to all critical services:

Example for Prometheus:

```yaml
  prometheus:
    image: prom/prometheus:latest
    restart: always
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
```

---

## 🧪 Step 6.2: Test Auto-Restart

Try this:

```bash
docker stop app
docker ps  # Wait a few seconds and check again
```

If set to `restart: always`, Docker will automatically restart the container.

---

## 🔁 Step 6.3: Optional – Use Watchdog Containers or Scripts

For advanced auto-healing (beyond Docker), you can:

- Use a **sidecar** container to monitor logs and restart services
- Write a script with `cron` or `systemd` that pings health endpoints

Example Watchdog Bash script:

```bash
#!/bin/bash

if ! curl -s http://localhost:5000/health; then
  echo "App is down. Restarting..."
  docker restart app
fi
```

Then schedule it with cron:

```bash
*/2 * * * * /usr/local/bin/watchdog.sh
```

---

## 🔔 Step 6.4: Alert If Auto-Healing Fails

In Phase 3 (Alerting), you already set up Prometheus alerts for service downtime. Now, make sure those alerts still fire if Docker cannot recover the service (e.g., due to bad image or crash loop).

You can also add an alert if a container restarts too frequently:

```yaml
- alert: ContainerCrashLoop
  expr: rate(container_restart_count[5m]) > 3
  for: 2m
  labels:
    severity: warning
  annotations:
    summary: "Container {{ $labels.container }} is restarting too frequently"
```

---

## ✅ Phase 6 Complete!

You now have:

- 🔁 Auto-recovery using Docker's native restart policy
- ⚙️ Optional watchdog script for custom healing logic
- 🔔 Alerting if recovery isn't working

This ensures high service availability and **reduces MTTR (Mean Time to Recovery)** significantly.

---

