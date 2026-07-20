# Modul 1: Install CasaOS

CasaOS mengubah terminal server OCI jadi dashboard visual. Dari sini kamu bisa install aplikasi, manage file, dan lihat status server — semua dari browser.

## Penting: Masalah Port 80

CasaOS secara default menggunakan port **80**. Tapi di server OCI kita, port 80 sudah dipakai oleh **Caddy** (web server dari kursus sebelumnya). Jadi kita harus ganti port CasaOS ke **81** setelah install.

> **Catatan**: Kalau belum punya Caddy di server, port 80 aman. Tapi karena kursus ini lanjutan dari oci-domain-cloudflare, Caddy pasti sudah jalan.
>
> **Catatan Tambahan**: `systemd-resolved` sudah dinonaktifkan dari course Pi-hole sebelumnya. Ini tidak masalah untuk CasaOS.

## Langkah

### 1. SSH ke Server

```bash
ssh user@server-oci
```

### 2. Install CasaOS

Jalankan perintah ini:

```bash
curl -fsSL https://get.casaos.io | sudo bash
```

Proses instalasi sekitar 2-5 menit. Script akan:
- Install Docker jika belum ada — CasaOS butuh Docker
- Download dan setup CasaOS
- Membuka port 80 dan 81 di iptables

> **Jangan langsung akses CasaOS dulu.** Karena Caddy pakai port 80, kita harus ganti port CasaOS dulu.

### 3. Cek Apakah CasaOS Berjalan

```bash
sudo systemctl status casaos-gateway
```

Kalau berjalan, output akan seperti:

```
● casaos-gateway.service - CasaOS Gateway
     Loaded: loaded
     Active: active (running)
```

Kalau ada error, tunggu 1 menit lalu coba lagi.

Cek juga containernya:

```bash
docker ps | grep casaos
```

Harusnya ada 4 container:

```
casaos-gateway
casaos-user-service
casaos-app-management
casaos-message-bus
```

### 4. Ganti Port CasaOS ke 81 (PENTING)

Karena Caddy sudah pakai port 80, CasaOS harus dipindah.

Buka file konfigurasi gateway:

```bash
sudo nano /etc/casaos/gateway.ini
```

Di dalam file, kamu akan lihat:

```ini
[gateway]
port = 80
```

Ganti `port = 80` menjadi `port = 81`:

```ini
[gateway]
port = 81
```

Simpan: tekan `Ctrl+X`, lalu `Y`, lalu `Enter`.



Restart service CasaOS:

```bash
sudo systemctl restart casaos-gateway
```

Sekarang CasaOS berjalan di port **81**.

### 5. Akses CasaOS

Kamu butuh alamat IP server OCI untuk akses dari browser. Ada dua cara:

**Cara 1 — IP Publik (Akses dari mana saja)**

```bash
curl -4 ifconfig.me
```

Output: `140.238.xx.xx`

**Cara 2 — IP Tailscale (Akses dari device di mesh network)**

```bash
tailscale ip -4
```

Output: `100.xx.xx.xx`

> **Catatan**: `hostname -I` tidak bisa dipakai karena hanya menampilkan IP internal OCI (10.x.x.x) yang tidak bisa diakses dari luar.

Buka browser di laptop/PC kamu, ketik:

```
http://<IP_PUBLIK_ATAU_TAILSCALE>:81
```

Contoh IP publik: `http://140.238.xx.xx:81`
Contoh IP Tailscale: `http://100.xx.xx.xx:81`

Kamu akan melihat halaman setup CasaOS:

1. Buat **username** dan **password** — simpan baik-baik
2. Klik **Submit**
3. Selamat datang di dashboard CasaOS

### 6. Jelajah Dashboard

CasaOS punya beberapa bagian:

| Bagian | Fungsi |
| --- | --- |
| **Dashboard** | Status server — CPU, RAM, disk, uptime |
| **File** | File manager — upload/download file, mirip File Explorer |
| **Apps** | App store — install aplikasi dengan 1 klik |
| **Settings** | Konfigurasi system, update, network |

### 7. Verifikasi

```bash
# Cek apakah container CasaOS berjalan normal
docker ps | grep casaos

# Cek port 81 sudah aktif
curl -I http://localhost:81
```

Output `curl` harus seperti:

```
HTTP/1.1 302 Found
Location: /
```

> **Catatan**: Kalau `curl` error, restart service: `sudo systemctl restart casaos-gateway`

## Troubleshooting

| Masalah | Solusi |
| --- | --- |
| Port 81 tidak bisa diakses | Cek firewall: `sudo iptables -L INPUT -n | grep 81` |
| Gagal akses dari browser | Pastikan pakai `http://`, bukan `https://` — CasaOS belum pakai SSL |
| Container restart terus | `docker logs casaos-gateway` — cek error |

## Hasil Akhir

- [ ] CasaOS terinstall dan berjalan
- [ ] Port sudah diganti ke 81 (tidak bentrok dengan Caddy)
- [ ] Dashboard bisa diakses via `http://<IP_PUBLIK_ATAU_TAILSCALE>:81`
- [ ] Siap install aplikasi pendukung

## Berikutnya

[Modul 2 — Uptime Kuma](02_UPTIME_KUMA.md)
