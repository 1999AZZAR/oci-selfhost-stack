# Modul 4: Subdomain & Tunnel Integration

Service-service sudah jalan di port lokal. Sekarang kita akses via domain dengan HTTPS — semuanya lewat Cloudflare Tunnel, tanpa port terbuka.

## Cara Kerja

```
Browser → https://casaos.example.com
  → Cloudflare Tunnel (outbound)
  → cloudflared di OCI
  → kirim ke http://casaos-gateway:81
  → CasaOS response balik

Semua aman — tidak ada port terbuka di firewall.
```

## Prasyarat

- Cloudflare Tunnel sudah berjalan (dari kursus oci-domain-cloudflare)
- Semua container sudah running (`docker ps`)
- Kamu sudah pernah nambah ingress sebelumnya

## Langkah

### 1. Catat Port & Nama Container

| Service | Port Internal | Nama Container |
| --- | --- | --- |
| CasaOS | `81` | `casaos-gateway` |
| Uptime Kuma | `3001` | `uptime-kuma` |
| Dashy | `8080` | `dashy` |

Pi-hole dari kursus sebelumnya — port internal `80`, container name `pihole`. Tapi Pi-hole masih di **default Docker bridge**, belum di `proxy-net`. Kita akan hubungkan nanti.

### 2. Pastikan Container Satu Network

Tunnel butuh akses ke container-container ini. Caranya: semua container harus terhubung ke `proxy-net`.

Cek network yang ada:

```bash
docker network ls
```

Harusnya sudah ada `proxy-net` dari kursus domain sebelumnya. Kalau belum:

```bash
docker network create proxy-net
```

#### CasaOS

CasaOS punya network sendiri (`casaos`). Jangan lepaskan. Cukup tambah `proxy-net`:

```bash
docker network connect proxy-net casaos-gateway
```

#### Pi-hole (Penting!)

Pi-hole dari course sebelumnya berjalan di **default bridge** network, bukan `proxy-net`. Hubungkan:

```bash
docker network connect proxy-net pihole
```

> **Kenapa?** cloudflared ada di `proxy-net`. Kalau Pi-hole tidak di `proxy-net`, tunnel nggak bisa akses `http://pihole:80`.

#### Uptime Kuma & Dashy

Kalau diinstall via CasaOS App Store, cek dulu:

```bash
docker inspect uptime-kuma | grep -A 5 "Networks"
```

Kalau tidak ada `proxy-net`, hubungkan:

```bash
docker network connect proxy-net uptime-kuma
docker network connect proxy-net dashy
```

Kalau diinstall manual via compose, mereka sudah include `proxy-net` — tidak perlu.

### 3. Tambah Ingress di Dashboard Cloudflare

1. Buka [Cloudflare Dashboard](https://dash.cloudflare.com)
2. **Zero Trust** → **Access** → **Tunnels**
3. Klik tunnel kamu (misal: `oci-server`)
4. Klik tab **Public Hostname**
5. Klik **Add a public hostname**

#### CasaOS

| Field | Isi |
| --- | --- |
| Subdomain | `casaos` |
| Domain | `example.com` (ganti punyamu) |
| Type | `HTTP` |
| URL | `http://casaos-gateway:81` |

Klik **Save hostname**.

#### Uptime Kuma

Klik **Add a public hostname** lagi:

| Field | Isi |
| --- | --- |
| Subdomain | `uptime` |
| Domain | `example.com` |
| Type | `HTTP` |
| URL | `http://uptime-kuma:3001` |

Klik **Save hostname**.

#### Dashy

Klik **Add a public hostname** lagi:

| Field | Isi |
| --- | --- |
| Subdomain | `dashy` |
| Domain | `example.com` |
| Type | `HTTP` |
| URL | `http://dashy:8080` |

Klik **Save hostname**.

> **Catatan**: Port internal Dashy di container adalah `8080`, bukan `80`. Tunnel akses via `dashy:8080` (nama container), bukan `8082` (host port). Port host cuma dipakai kalau akses langsung via IP.

#### Pi-hole (Kalau Ada)

| Field | Isi |
| --- | --- |
| Subdomain | `pihole` |
| Domain | `example.com` |
| Type | `HTTP` |
| URL | `http://pihole:80` |

### 4. Verifikasi Tunnel

Cek apakah tunnel menerima konfigurasi baru:

```bash
docker logs cloudflared --tail 20
```

Cari log seperti:

```
Registered tunnel connection
Added ingress rule for casaos.example.com
Added ingress rule for uptime.example.com
Added ingress rule for dashy.example.com
```

Coba akses semua URL di browser:

```
https://casaos.example.com     → Dashboard CasaOS (loginn dulu)
https://uptime.example.com     → Dashboard Uptime Kuma
https://dashy.example.com      → Dashboard Dashy
https://pihole.example.com/admin → Pi-hole (kalau ada)
```

Semua harus muncul dengan **gembok hijau** (HTTPS valid).

> **Catatan**: Kalau error **502 Bad Gateway**, berarti tunnel sudah meneruskan traffic tapi service tujuan belum merespon. Cek: `docker ps` — apakah semua container running?

### 5. Update Dashy dengan URL Final

Edit `conf.yml` Dashy — ganti semua URL dari `http://IP:port` ke `https://subdomain.example.com`:

```bash
nano ~/dashy/user-data/conf.yml
```

```yaml
sections:
  - name: Infrastruktur
    items:
      - title: CasaOS
        url: https://casaos.example.com

      - title: Uptime Kuma
        url: https://uptime.example.com

      - title: Dashy
        url: https://dashy.example.com

      - title: Pi-hole
        url: https://pihole.example.com/admin
```

Simpan. Reload Dashy:

```bash
docker restart dashy
```

Tunggu 30 detik, buka `https://dashy.example.com` — semua link sekarang pakai domain + HTTPS.

## Troubleshooting

| Masalah | Kemungkinan | Solusi |
| --- | --- | --- |
| **502 Bad Gateway** | Tunnel bisa reach Cloudflare, tapi service tujuan tidak merespon | Cek `docker ps` — apakah container hidup? Cek network — apakah container terhubung ke `proxy-net`? |
| **Error 520** | Cloudflare tidak bisa reach tunnel | Cek status tunnel di dashboard Zero Trust |
| **Error 1001 DNS** | DNS record subdomain belum ada | Cloudflare auto-buat CNAME saat kamu save ingress. Tunggu 1-2 menit |
| **Halaman CasaOS error** | Port 81 sudah sesuai? | `curl http://localhost:81` — harus response. Cek `/etc/casaos/gateway.ini` |
| **Dashy putih terus** | Belum selesai build | `docker logs dashy --tail 20`. Biasanya butuh 30-60 detik pertama kali |

## Hasil Akhir

- [ ] CasaOS bisa diakses via `https://casaos.example.com`
- [ ] Uptime Kuma bisa diakses via `https://uptime.example.com`
- [ ] Dashy bisa diakses via `https://dashy.example.com`
- [ ] Semua HTTPS valid — gembok hijau
- [ ] Dashy menampilkan link ke semua service
- [ ] Tunnel menangani semua traffic — tidak ada port terbuka

## Berikutnya

[Modul 5 — Portainer (Opsional)](05_PORTAINER_OPTIONAL.md)
