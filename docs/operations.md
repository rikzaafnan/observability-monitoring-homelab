# Operasional Observability Homelab

Dokumen ini berisi catatan harian untuk menjalankan, mengecek, dan merawat stack observability.

## Perintah Utama

```bash
docker compose up -d
docker compose ps
docker compose logs -f
docker compose down
```

Untuk restart satu servis:

```bash
docker compose restart prometheus
```

Untuk reload konfigurasi Prometheus tanpa restart container:

```bash
curl -X POST http://localhost:9090/-/reload
```

## Cek Kesehatan

- Grafana: http://localhost:3000
- Prometheus targets: http://localhost:9090/targets
- Prometheus alerts: http://localhost:9090/alerts
- Alertmanager: http://localhost:9093
- Loki ready endpoint: http://localhost:3100/ready
- cAdvisor: http://localhost:8080
- node-exporter metrics: http://localhost:9100/metrics
- blackbox-exporter: http://localhost:9115

## Backup

Data disimpan di Docker volume:

- `observability-homelab_prometheus-data`
- `observability-homelab_grafana-data`
- `observability-homelab_loki-data`
- `observability-homelab_alertmanager-data`

Contoh backup volume Grafana:

```bash
docker run --rm \
  -v observability-homelab_grafana-data:/data:ro \
  -v "$PWD/backups:/backup" \
  alpine tar czf /backup/grafana-data.tar.gz -C /data .
```

## Menambah Target Scrape

Tambahkan target di `prometheus/prometheus.yml`, lalu reload Prometheus.

Contoh:

```yaml
- job_name: app-example
  static_configs:
    - targets:
        - 192.168.1.20:9100
```

## Menambah Notifikasi Alertmanager

Edit `alertmanager/alertmanager.yml`, lalu restart Alertmanager.

Contoh placeholder webhook:

```yaml
receivers:
  - name: default
    webhook_configs:
      - url: http://example-webhook:8080/alert
```

Untuk Telegram, Discord, Slack, atau email, isi receiver sesuai channel yang dipakai di homelab.

## Catatan Platform

Stack ini paling cocok dijalankan di Linux host karena `node-exporter`, `cAdvisor`, dan `promtail` membaca path host seperti `/proc`, `/sys`, `/var/log`, dan `/var/lib/docker/containers`.

Di Docker Desktop macOS atau Windows, sebagian metrik host dan log container bisa berbeda karena Docker berjalan di VM. Grafana, Prometheus, Loki, dan Alertmanager tetap bisa jalan, tetapi data host mungkin tidak selengkap Linux.
