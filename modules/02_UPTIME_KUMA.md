# Modul 2: Uptime Kuma

Uptime Kuma adalah monitoring tool. Dia akan mengecek service-service kamu setiap beberapa detik — kalau ada yang mati, kamu langsung tahu.

## Cara Kerja

```
Uptime Kuma → cek https://example.com setiap 60 detik
            → cek https://pihole.example.com setiap 60 detik
            → kalau mati → kirim notifikasi ke Telegram/Email
```

## Metode Install

Kamu bisa pilih salah satu:

- **Via CasaOS** — lebih mudah, 1 klik
- **Manual Docker** — lebih kontrol

---

## Opsi A: Via CasaOS App Store (Termudah)

1. Buka CasaOS dashboard di `http://<IP_PUBLIK_ATAU_TAILSCALE>:81`
2. Klik menu **Apps** di kiri
3. Klik **App Store**
4. Cari "Uptime Kuma" di kolom pencarian
5. Klik **Install**
6. Biarkan semua setting default, klik **Confirm**

CasaOS akan otomatis download dan jalankan Uptime Kuma. Setelah selesai (1-2 menit), Uptime Kuma akan muncul di halaman **My Apps** di CasaOS.

CasaOS akan menggunakan port **3001** untuk Uptime Kuma.

---

## Opsi B: Manual via Docker (Advanced)

1. Buat folder untuk Uptime Kuma:

```bash
mkdir -p ~/uptime-kuma && cd ~/uptime-kuma
```

2. Buat file `compose.yaml`:

```bash
nano compose.yaml
```

3. Copy-paste ini:

```yaml
services:
  uptime-kuma:
    image: louislam/uptime-kuma:2
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

4. Simpan: `Ctrl+X`, `Y`, `Enter`

5. Jalankan:

```bash
docker compose up -d
```

6. Verifikasi:

```bash
docker ps | grep uptime-kuma
```

Output:

```
<id>   louislam/uptime-kuma:2   ...   Up X minutes   0.0.0.0:3001->3001/tcp   uptime-kuma
```

> **Catatan**: `:2` di image artinya versi 2.x terbaru. Jangan pakai `:latest` karena sudah deprecated (tidak disarankan oleh developer).

> **Cara dapat IP server**: `curl -4 ifconfig.me` (IP publik) atau `tailscale ip -4` (IP Tailscale).

---

## Setup Awal

1. Buka browser, akses:

```
http://<IP_PUBLIK_ATAU_TAILSCALE>:3001
```

Contoh: `http://140.238.xx.xx:3001`

2. Kamu akan diminta membuat akun admin:

| Field | Isi |
| --- | --- |
| Username | Terserah (misal: `admin`) |
| Password | Buat password yang kuat |
| Repeat Password | Ketik ulang password |

3. Klik **Create**

## Tambah Monitor Pertama

1. Di dashboard Uptime Kuma, klik tombol **Add Monitor** (tombol biru)
2. Isi form:

| Field | Isi | Contoh |
| --- | --- | --- |
| Monitor Type | Pilih `HTTP(s)` | |
| Friendly Name | Nama service | `Caddy - Apex Domain` |
| URL | Alamat service | `https://example.com` |
| Interval | Detik | `60` |

3. Klik **Save**

Setelah disimpan, Uptime Kuma akan mulai mengecek service itu setiap 60 detik. Statusnya:

- 🟢 **UP** — service berjalan normal
- 🔴 **DOWN** — service mati, Uptime Kuma akan kasih tahu

### Monitor yang Disarankan

Kamu bisa tambah beberapa monitor sekaligus:

| Nama | URL | Interval |
| --- | --- | --- |
| Caddy - Apex Domain | `https://example.com` | 60s |
| Pi-hole | `http://pihole:80` | 60s |
| CasaOS | `http://casaos-gateway:81` | 60s |
| OCI Server (Ping) | IP server OCI | 120s |

> **Catatan untuk Pi-hole & CasaOS**: Karena nanti semua service akan diakses via tunnel (Modul 4), Uptime Kuma juga harus bisa menjangkaunya. Untuk sementara, pakai alamat internal (`http://pihole:80`). Setelah tunnel terpasang, ganti ke `https://pihole.example.com`.

## Notifikasi (Opsional, Tapi Berguna)

Uptime Kuma bisa kirim pesan ke Telegram atau Email kalau ada service mati.

1. Klik **Settings** (icon gear di kiri bawah)
2. Klik **Notifications**
3. Klik **Add Notification**
4. Pilih provider — **Telegram** paling mudah:
   - Kirim pesan ke [@BotFather](https://t.me/botfather) di Telegram
   - Ketik `/newbot` dan ikuti petunjuk
   - Dapatkan **Token Bot** (format: `123456:ABC-DEF1234`)
   - Cari **Chat ID** kamu (kirim pesan ke [@userinfobot](https://t.me/userinfobot))
   - Masukkan token dan chat ID di Uptime Kuma
5. Klik **Save**
6. Test: Klik **Test** — kamu akan terima notifikasi test

## Verifikasi

```bash
# Cek apakah container berjalan
docker ps | grep uptime-kuma

# Cek log
docker logs uptime-kuma --tail 10
```

Buka browser ke `http://<IP_PUBLIK_ATAU_TAILSCALE>:3001` — dashboard harus muncul dengan monitor yang sudah ditambah.

## Hasil Akhir

- [ ] Uptime Kuma berjalan di port 3001
- [ ] Minimal 1 monitor aktif (Caddy)
- [ ] Dashboard menampilkan status UP
- [ ] (Opsional) Notifikasi Telegram terkonfigurasi
- [ ] Siap lanjut ke dashboard utama

## Berikutnya

[Modul 3 — Dashy Dashboard](03_DASHY_DASHBOARD.md)
