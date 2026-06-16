# Docker Logs dengan Grafana Alloy

Promtail sudah diganti dengan Grafana Alloy. Alloy membaca log container Docker dari Docker socket dan mengirimnya ke Loki.

## Cara Kerja

Service `alloy` di `docker-compose.yml` memakai:

- `/var/run/docker.sock` untuk auto-discovery container.
- `/var/lib/docker/containers` untuk membaca file log container.
- `/var/log` untuk log host seperti Nginx dan system log.
- `${MONITORING_PATH}/alloy` untuk menyimpan state Alloy.

Konfigurasinya ada di:

```text
alloy/config.alloy
```

## Query Log Docker di Grafana

Buka Grafana, masuk ke `Explore`, pilih datasource `Loki`, lalu gunakan LogQL.

Semua log Docker:

```logql
{job="docker"}
```

Log berdasarkan nama container:

```logql
{container="nama-container"}
```

Log berdasarkan Docker Compose service:

```logql
{compose_service="nama-service"}
```

Log berdasarkan Docker Compose project:

```logql
{compose_project="nama-project"}
```

## Menambahkan Docker dari Server Lain

Jalankan Alloy juga di server lain, lalu arahkan `loki.write` ke Loki utama:

```alloy
loki.write "default" {
  endpoint {
    url = "http://IP_SERVER_MONITORING:3100/loki/api/v1/push"
  }
}
```

Tambahkan label `host` yang berbeda agar log mudah difilter:

```alloy
labels = {
  job  = "docker",
  host = "server-app-01",
}
```

Lalu query:

```logql
{host="server-app-01"}
```
