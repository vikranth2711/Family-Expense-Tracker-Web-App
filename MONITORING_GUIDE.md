# 📊 Grafana & Prometheus Monitoring Guide

Complete guide to using Grafana and Prometheus for monitoring your Family Expense Tracker application.

---

## 🌐 Access URLs

- **Grafana Dashboard**: http://3.226.196.225:3001
- **Prometheus Metrics**: http://3.226.196.225:9090

---

## 📊 Part 1: Using Grafana (Visualization)

### Step 1: Login to Grafana

1. Open http://3.226.196.225:3001 in your browser
2. **Username**: `admin`
3. **Password**: `admin`
4. Change password when prompted (first login)

### Step 2: Add Prometheus Data Source

1. Click **⚙️ (Settings icon)** in left sidebar
2. Select **Data Sources**
3. Click **Add data source**
4. Select **Prometheus**
5. Configure:
   - **Name**: Prometheus
   - **URL**: `http://prometheus:9090`
   - Leave other settings as default
6. Click **Save & Test** (should show green "Data source is working")

### Step 3: Import Pre-Built Dashboard (Easiest!)

1. Click **+ (Plus icon)** in left sidebar
2. Select **Import**
3. Enter Dashboard ID: **1860** (Node Exporter Full)
4. Click **Load**
5. Select **Prometheus** as data source
6. Click **Import**

**🎉 You now have a full monitoring dashboard!**

#### Recommended Dashboard IDs:

| ID    | Name                         | Description                |
| ----- | ---------------------------- | -------------------------- |
| 1860  | Node Exporter Full           | Complete server metrics    |
| 405   | Node Exporter Server Metrics | Simplified server view     |
| 11074 | Node Exporter for Prometheus | Detailed system monitoring |
| 893   | Docker and System Monitoring | Container-focused metrics  |

### Step 4: Create Custom Dashboard

1. Click **+ → Dashboard**
2. Click **Add new panel**
3. In the query editor, try these examples:

#### Example 1: CPU Usage (%)

```promql
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

#### Example 2: Memory Usage (%)

```promql
(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100
```

#### Example 3: Disk Usage (%)

```promql
(node_filesystem_size_bytes - node_filesystem_free_bytes) / node_filesystem_size_bytes * 100
```

#### Example 4: Network Traffic (bytes/sec)

```promql
rate(node_network_receive_bytes_total[5m])
rate(node_network_transmit_bytes_total[5m])
```

4. Click **Apply** to save the panel
5. Click **💾 Save dashboard** (top right)

### Step 5: Set Up Alerts

1. Open a panel → Click **Edit**
2. Go to **Alert** tab
3. Click **Create Alert**
4. Configure conditions:
   - **When**: avg()
   - **Of**: your query
   - **Is Above**: 80 (for CPU warning at 80%)
5. Add **Notification Channel**:
   - Email
   - Slack
   - Discord
   - Webhook

---

## 🔍 Part 2: Using Prometheus (Metrics Engine)

### Accessing Prometheus

Open http://3.226.196.225:9090 in your browser

### Main Features:

#### 1. Graph Tab (Query & Visualize)

- Write PromQL queries
- View instant metrics
- Create time-series graphs

#### 2. Targets Tab (Health Check)

- See all monitored endpoints
- Check if services are UP or DOWN
- View scrape duration and errors

#### 3. Alerts Tab (Alert Rules)

- View configured alerts
- Check alert status
- See firing alerts

---

## 📈 Useful Prometheus Queries

### System Health Queries

#### CPU Usage by Core

```promql
rate(node_cpu_seconds_total[5m])
```

#### Available Memory (GB)

```promql
node_memory_MemAvailable_bytes / 1024 / 1024 / 1024
```

#### Disk Free Space (GB)

```promql
node_filesystem_free_bytes / 1024 / 1024 / 1024
```

#### System Uptime (hours)

```promql
(node_time_seconds - node_boot_time_seconds) / 3600
```

### Network Metrics

#### Network Receive Rate (MB/s)

```promql
rate(node_network_receive_bytes_total[5m]) / 1024 / 1024
```

#### Network Transmit Rate (MB/s)

```promql
rate(node_network_transmit_bytes_total[5m]) / 1024 / 1024
```

### Container Metrics (if Docker containers are monitored)

#### Container CPU Usage

```promql
rate(container_cpu_usage_seconds_total[5m])
```

#### Container Memory Usage (MB)

```promql
container_memory_usage_bytes / 1024 / 1024
```

---

## 🚨 Alert Examples

### High CPU Alert

```promql
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
```

**Meaning**: Alert when CPU usage exceeds 80%

### Low Memory Alert

```promql
(node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 < 10
```

**Meaning**: Alert when available memory drops below 10%

### Disk Space Low Alert

```promql
(node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 10
```

**Meaning**: Alert when disk space is below 10%

### High Network Traffic

```promql
rate(node_network_receive_bytes_total[5m]) > 100000000
```

**Meaning**: Alert when receiving more than 100MB/s

---

## 💡 What You Can Monitor

### ✅ Server Health

- **CPU**: Is the server overloaded?
- **Memory**: Running out of RAM?
- **Disk**: Need more storage?
- **Network**: Bandwidth usage patterns

### ✅ Application Performance

- **Response times**: How fast is the app?
- **Request rates**: How many users?
- **Error rates**: Any failures?
- **Container health**: All services running?

### ✅ Cost Optimization

- **Resource usage patterns**: When is peak time?
- **Under-utilization**: Can downgrade instance?
- **Over-utilization**: Need to scale up?

### ✅ Troubleshooting

- **Historical data**: What happened at 3 AM?
- **Correlations**: Did CPU spike cause errors?
- **Trends**: Usage growing over time?

---

## 🎯 Quick Start Checklist

- [ ] Open Grafana: http://3.226.196.225:3001
- [ ] Login with admin/admin
- [ ] Add Prometheus data source (http://prometheus:9090)
- [ ] Import dashboard 1860 (Node Exporter Full)
- [ ] Explore your metrics!
- [ ] Set up email alerts for critical metrics
- [ ] Create custom dashboard for your needs

---

## 📚 Advanced Topics

### Creating Alert Notification Channels

1. Go to **Alerting → Notification channels**
2. Click **Add channel**
3. Choose type:
   - **Email**: Receive alerts via email
   - **Slack**: Post to Slack channel
   - **Webhook**: Send to any HTTP endpoint
   - **Discord**: Discord server notifications
4. Configure and test

### Grafana Variables (Dynamic Dashboards)

Create variables for:

- Instance selection
- Time ranges
- Metric filters

### Prometheus Recording Rules

Pre-calculate expensive queries:

```yaml
groups:
  - name: example
    rules:
      - record: instance:cpu_usage:avg
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

---

## 🐛 Troubleshooting

### Grafana shows "No data"

1. Check Prometheus data source is configured correctly
2. Verify Prometheus URL: `http://prometheus:9090`
3. Check if Prometheus is running: `docker ps`
4. View Prometheus targets: http://3.226.196.225:9090/targets

### Prometheus not scraping metrics

1. Check `monitoring/prometheus/prometheus.yml` configuration
2. Verify node-exporter is running: `docker ps | grep node-exporter`
3. Restart containers: `docker-compose restart prometheus`

### Dashboard shows errors

1. Check PromQL syntax in queries
2. Verify metric names exist in Prometheus
3. Check time range selection (some metrics need time)

---

## 📖 Learning Resources

- **Grafana Official Docs**: https://grafana.com/docs/
- **Prometheus Documentation**: https://prometheus.io/docs/
- **PromQL Cheat Sheet**: https://promlabs.com/promql-cheat-sheet/
- **Grafana Dashboards**: https://grafana.com/grafana/dashboards/
- **Grafana Tutorials**: https://grafana.com/tutorials/

---

## 🎓 Next Steps

1. **Explore**: Play with different dashboard IDs
2. **Customize**: Create panels specific to your app
3. **Alert**: Set up notifications for critical issues
4. **Learn**: Study PromQL query language
5. **Share**: Export and share dashboards with team

---

**Happy Monitoring! 📊🎉**

Your application now has enterprise-grade observability!
