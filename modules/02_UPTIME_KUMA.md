# Modul 2: Uptime Kuma

Uptime Kuma memantau service kamu 24/7. Kalau Pi-hole mati, Caddy error, atau server down — kamu tahu dalam 30 detik.

## Cara Install

Dua opsi. Pilih salah satu.

### Opsi A: Via CasaOS App Store (Mudah)

1. Buka CasaOS dashboard
2. Klik **Apps** → **App Store**
3. Cari "Uptime Kuma"
4. Klik **Install**
5. Biarkan default — CasaOS handle sisanya

Selesai. Uptime Kuma akan jalan di port 3001.

### Opsi B: Manual via Docker (Advanced)

```bash
mkdir -p ~/uptime-kuma && cd ~/uptime-kuma
```

Buat `compose.yaml`:

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - ./data:/app/data
    networks:
      - proxy-net

networks:
  proxy-net:
    external: true
```

Jalankan:

```bash
docker compose up -d
```

## Setup Monitor Pertama

1. Buka `http://<IP_SERVER>:3001`
2. Buat akun admin
3. Klik **Add Monitor**
4. Isi:

| Field | Isi |
| --- | --- |
| Monitor Type | `HTTP(s)` |
| Friendly Name | `Caddy - Apex Domain` |
| URL | `https://example.com` |
| Interval | `60` detik |

5. Klik **Save**

Nanti setelah tunnel terpasang (Modul 4), kamu bisa akses via `https://uptime.example.com`.

### Monitor yang Disarankan

| Nama | URL | Interval |
| --- | --- | --- |
| Caddy (Apex) | `https://example.com` | 60s |
| Pi-hole | `http://pihole:80` | 60s |
| CasaOS | `http://casaos-gateway:81` | 60s |
| Server (Ping) | IP server OCI | 120s |

## Notifikasi

Uptime Kuma support Telegram, Email, Discord, dll:

1. Settings → **Notifications**
2. Pilih provider (Telegram paling mudah)
3. Ikuti instruksi pairing
4. Centang kirim notifikasi saat service **down** dan **up**

## Hasil Akhir

- [ ] Uptime Kuma berjalan
- [ ] Minimal 1 monitor aktif (Caddy)
- [ ] Notifikasi terkonfigurasi (opsional)
- [ ] Siap lanjut ke dashboard utama

## Berikutnya

[Modul 3 — Dashy Dashboard](03_DASHY_DASHBOARD.md)
