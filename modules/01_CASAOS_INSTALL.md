# Modul 1: Install CasaOS

CasaOS mengubah terminal server OCI jadi dashboard visual. Dari sini kamu bisa install aplikasi, manage file, dan lihat status server — semua dari browser.

## Cara Kerja

```
Browser → <IP_SERVER>:81 → CasaOS UI
```

Nanti di Modul 4 kita pasang tunnel biar aksesnya lewat domain + HTTPS.

## Langkah

### 1. Install CasaOS

SSH ke server OCI:

```bash
curl -fsSL https://get.casaos.io | sudo bash
```

Proses instalasi ~2-5 menit. Script akan:
- Install Docker (jika belum ada)
- Download & jalankan CasaOS container
- Buka port 81 di iptables otomatis

> **Catatan**: CasaOS otomatis buka port 81 di iptables. Tapi ini hanya akses lokal via IP. Nanti akses publik via tunnel — jadi port 81 nggak perlu dibuka di Security List OCI.

### 2. Akses CasaOS

Cari IP server OCI:

```bash
hostname -I | awk '{print $1}'
```

Buka browser: `http://<IP_SERVER>:81`

Kamu akan lihat halaman setup CasaOS:
- Buat username & password
- Selesai — masuk ke dashboard

### 3. Jelajah Dashboard

CasaOS punya beberapa bagian utama:

| Menu | Fungsi |
| --- | --- |
| **Dashboard** | Status server (CPU, RAM, disk) |
| **File** | File manager — upload/download lewat browser |
| **Apps** | App store — install aplikasi 1 klik |
| **Settings** | Konfigurasi sistem |

### 4. Verifikasi

```bash
# Cek container
docker ps | grep casaos
```

Output:

```
casaos-gateway   Up X minutes   80/tcp, :81->81/tcp
casaos-app-management  Up X minutes
...
```

## Catatan

- CasaOS jalan di port **81** (bukan 80/443). Ini sengaja biar nggak bentrok dengan Caddy
- File manager CasaOS bisa akses seluruh filesystem server — hati-hati dengan izin file
- App store berisi aplikasi-aplikasi yang sudah di-test untuk CasaOS

## Hasil Akhir

- [ ] CasaOS terinstall dan bisa diakses via `http://<IP>:81`
- [ ] Dashboard menampilkan status server
- [ ] Siap install aplikasi pendukung

## Berikutnya

[Modul 2 — Uptime Kuma](02_UPTIME_KUMA.md)
