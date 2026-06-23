# Github Copilot Monitoring

Monitor VS Code Copilot chat metrics in real-time with Grafana. Tracks token usage (input/output).

## Quick Start

### Prerequisites
- Docker Desktop or Docker Engine (v20.10+)
- Docker Compose (v1.29+)
- VS Code with Copilot Chat extension (telemetry enabled)

### 1. Start the Stack

```bash
docker-compose up -d
```

Wait for the container to be healthy (~30 seconds):
```bash
docker-compose ps
# STATUS should show "healthy"
```

### 2. Configure VS Code

Ensure these settings are in your VS Code `settings.json`:

```json
{
  "github.copilot.chat.otel.enabled": true,
  "github.copilot.chat.otel.captureContent": true
}
```

### 3. Open Grafana

- **URL:** http://localhost:3000
- **Username:** admin
- **Password:** admin

Metrics should appear in Grafana within 1-2 minutes of using Copilot Chat.

## Dashboards

- **Token Usage** — Input/output tokens usage

## Add Custom Dashboards

This project uses Grafana provisioning to auto-load dashboard JSON files from:

- Container path: `/otel-lgtm/grafana/conf/provisioning/dashboards/custom`
- Provisioning file: [dashboards-provisioning.yaml](dashboards-provisioning.yaml)

### 1. Create your dashboard JSON

Add your exported dashboard file under [grafana/dashboards](grafana/dashboards), for example:

- grafana/dashboards/my-new-dashboard.json

### 2. Mount it in Compose

Add a new volume line in [docker-compose.yml](docker-compose.yml) under the `lgtm` service:

```yaml
volumes:
  - ./grafana/dashboards/token-usage-by-model.json:/otel-lgtm/grafana/conf/provisioning/dashboards/custom/token-usage-by-model.json:ro
  - ./grafana/dashboards/my-new-dashboard.json:/otel-lgtm/grafana/conf/provisioning/dashboards/custom/my-new-dashboard.json:ro
  - ./dashboards-provisioning.yaml:/otel-lgtm/grafana/conf/provisioning/dashboards/custom.yaml:ro
```

### 3. (Optional) Set it as home dashboard

Update `GF_DASHBOARDS_DEFAULT_HOME_DASHBOARD_PATH` in [docker-compose.yml](docker-compose.yml):

```yaml
environment:
  - GF_DASHBOARDS_DEFAULT_HOME_DASHBOARD_PATH=/otel-lgtm/grafana/conf/provisioning/dashboards/custom/my-new-dashboard.json
```

## Documentation

- [VS Code Agent Monitoring Guide](https://code.visualstudio.com/docs/agents/guides/monitoring-agents) — Official guide for monitoring VS Code agents
- [Grafana Docker OTEL-LGTM](https://github.com/grafana/docker-otel-lgtm) — Open source observability stack used by this project

## Architecture

```
VS Code OTEL Exporter
    ↓ [gRPC: localhost:4317]
Grafana OTEL-LGTM Container
├── OTEL Collector
├── Mimir (metrics storage)
├── Loki (logs, optional)
├── Tempo (traces, optional)
└── Grafana Web UI (localhost:3000)
```

## Commands

**Start the stack:**
```bash
docker-compose up -d
```

**Stop the stack:**
```bash
docker-compose down
```

**View logs:**
```bash
docker-compose logs -f lgtm
```

**Reset data (delete volumes):**
```bash
docker-compose down -v
```

**Validate compose config:**
```bash
docker-compose config
```

Then open Grafana at http://localhost:3000 and verify the dashboard appears in the dashboard list.

Tip: Keep dashboard filenames stable to avoid resetting home dashboard paths.

## License

MIT
