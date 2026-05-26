# Grafana for TypeDB

Self-contained Prometheus + Grafana stack for local TypeDB metrics. Scrapes a running TypeDB
server's `/diagnostics` endpoint and renders it through a pre-provisioned dashboard.

## Usage

```bash
tool/grafana/start                          # defaults: scrape 127.0.0.1:4104
tool/grafana/start --target some-host:4104  # different endpoint
```

Then open <http://127.0.0.1:3000> (anonymous Admin, no login).

Requires a TypeDB server with the diagnostics/metrics endpoint enabled. On first run, downloads
pinned Prometheus + Grafana releases into `tool/grafana/bin/` (gitignored). Ctrl-C tears both down.

See `start --help` for all options.
