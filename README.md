# Observability Monitoring Homelab

Docker Compose stack untuk monitoring homelab kecil sampai menengah. Stack ini menyediakan metrics, logs, dashboard, dan alerting dasar yang bisa dikembangkan untuk server, VM, container, NAS, router, atau aplikasi internal.

## Isi Stack

| Komponen | Fungsi | URL lokal |
| --- | --- | --- |
| Grafana | Dashboard dan eksplorasi data | http://localhost:3000 |
| Prometheus | Metrics database dan alert rules | http://localhost:9090 |
| Alertmanager | Routing alert dari Prometheus | http://localhost:9093 |
| Loki | Log aggregation | http://localhost:3100 |
| Promtail | Log collector untuk Loki | internal |
| node-exporter | Metrics host Linux | http://localhost:9100 |
| cAdvisor | Metrics container Docker | http://localhost:8080 |
| blackbox-exporter | Probe HTTP/TCP/ICMP | http://localhost:9115 |

## Struktur Repo

```text
.
├── alertmanager/
├── blackbox/
├── docs/
├── grafana/
│   ├── dashboards/
│   └── provisioning/
├── loki/
├── prometheus/
│   └── rules/
├── promtail/
├── docker-compose.yml
└── .env.example
```

## Quick Start

1. Salin file environment.

   ```bash
   cp .env.example .env
   ```

2. Ubah password Grafana di `.env`.

   ```env
   GRAFANA_ADMIN_USER=admin
   GRAFANA_ADMIN_PASSWORD=ganti-password-ini
   ```

3. Jalankan stack.

   ```bash
   docker compose up -d
   ```

4. Buka Grafana di http://localhost:3000.

   Dashboard `Homelab Overview` akan otomatis muncul di folder `Homelab`. Datasource Prometheus dan Loki juga otomatis terpasang.

## Verifikasi

```bash
docker compose ps
docker compose logs -f prometheus
docker compose logs -f grafana
```

Cek target Prometheus:

```text
http://localhost:9090/targets
```

Semua target utama seharusnya berstatus `UP`.

## Konfigurasi Penting

- `prometheus/prometheus.yml`: daftar target metrics.
- `prometheus/rules/homelab-alerts.yml`: alert dasar untuk host, disk, memory, dan target down.
- `alertmanager/alertmanager.yml`: routing notifikasi alert.
- `loki/loki.yml`: storage dan retensi log Loki.
- `promtail/promtail.yml`: sumber log dari host dan container Docker.
- `blackbox/blackbox.yml`: modul probe HTTP, TCP, dan ICMP.
- `grafana/provisioning/`: datasource dan dashboard otomatis.

## Menambah Service Homelab

Untuk service yang sudah expose endpoint `/metrics`, tambahkan scrape job di `prometheus/prometheus.yml`.

```yaml
- job_name: my-service
  static_configs:
    - targets:
        - my-service:8080
```

Jika service ada di mesin lain:

```yaml
- job_name: remote-node
  static_configs:
    - targets:
        - 192.168.1.10:9100
```

Setelah edit, reload Prometheus:

```bash
curl -X POST http://localhost:9090/-/reload
```

## Alerting

Alert bawaan:

- `TargetDown`: target Prometheus tidak bisa di-scrape.
- `HostHighCpuLoad`: CPU host di atas 85%.
- `HostHighMemoryUsage`: memory host di atas 90%.
- `HostLowDiskSpace`: disk usage di atas 85%.
- `ContainerRestartingOften`: container sering restart.

Default Alertmanager belum mengirim notifikasi keluar. Tambahkan receiver di `alertmanager/alertmanager.yml` untuk Slack, Discord, Telegram, email, atau webhook lain.

## Logs

Promtail mengirim:

- log host dari `/var/log/**/*.log`
- log container Docker dari `/var/lib/docker/containers`

Di Grafana, buka `Explore`, pilih datasource `Loki`, lalu coba query:

```logql
{compose_project="observability-homelab"}
```

atau:

```logql
{container="homelab-prometheus"}
```

## Catatan untuk macOS dan Windows

Stack ini paling ideal untuk Linux host. Di Docker Desktop macOS atau Windows, Docker berjalan di VM sehingga mount host seperti `/proc`, `/sys`, `/var/log`, dan `/var/lib/docker` bisa menghasilkan data yang berbeda atau tidak lengkap.

Untuk homelab produksi, jalankan stack ini di server Linux seperti Debian, Ubuntu Server, Proxmox VM, atau mesin mini PC.

## Operasional Lanjutan

Lihat [docs/operations.md](docs/operations.md) untuk backup, health check, reload konfigurasi, dan catatan perawatan.
