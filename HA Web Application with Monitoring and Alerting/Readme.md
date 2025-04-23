
```markdown
# High Availability Web Application with Monitoring and Alerting

This project implements a high-availability web application infrastructure using **Docker**, **HAProxy**, and **Prometheus** for monitoring and alerting. The web app is containerized, with traffic load-balanced between multiple instances to ensure high availability.

---

## 🛠️ Phase 1: Set Up a Simple Web App with Docker

### Step 1.1: Create Project Structure

Create the necessary directory structure for your project.

```bash
mkdir sre-ha-webapp && cd sre-ha-webapp
mkdir app monitoring logging ansible watchdog
touch docker-compose.yml
```

### Step 1.2: Create a Simple Web App

We'll use a basic Python Flask app to serve HTTP responses.

Create the following files:

#### `app/app.py`
```python
from flask import Flask
import socket

app = Flask(__name__)

@app.route("/")
def home():
    return f"Hello from {socket.gethostname()}!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

#### `app/requirements.txt`
```plaintext
flask
```

#### `app/Dockerfile`
```Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

### Step 1.3: Add Docker Compose for Multiple App Instances and Load Balancer

We’ll use HAProxy as the load balancer and Docker Compose to manage the containers.

#### `docker-compose.yml`
```yaml
version: '3.8'

services:
  app1:
    build: ./app
    container_name: app1
    ports:
      - "5001:5000"

  app2:
    build: ./app
    container_name: app2
    ports:
      - "5002:5000"

  haproxy:
    image: haproxy:latest
    container_name: haproxy
    ports:
      - "8080:80"
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg
    depends_on:
      - app1
      - app2
```

#### `haproxy.cfg`
```cfg
global
  daemon
  maxconn 256

defaults
  mode http
  timeout connect 5000ms
  timeout client  50000ms
  timeout server  50000ms

frontend http-in
  bind *:80
  default_backend servers

backend servers
  balance roundrobin
  server app1 app1:5000 check
  server app2 app2:5000 check
```

### Step 1.4: Build and Run the Application

Run the following command to build and start the containers:

```bash
docker-compose up --build -d
```

### Step 1.5: Test the Setup

Now, open your browser and go to [http://localhost:8080](http://localhost:8080). You should see the web page alternating between `app1` and `app2` indicating the load balancing is working.

---

## ✅ Phase 1 Complete!

You have now successfully set up:

- A simple web application in Python using Flask
- Two app containers running behind a load balancer (HAProxy)
- Load balancing and high availability with traffic distribution between app instances

---

