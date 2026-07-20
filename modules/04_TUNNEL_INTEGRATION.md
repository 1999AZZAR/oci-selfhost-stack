# Modul 4: Subdomain & Tunnel Integration

Sekarang semua service sudah jalan di port lokal. Saatnya akses via domain + HTTPS — subdomain via Cloudflare Tunnel.

## Cara Kerja

```
https://casaos.example.com  →  Cloudflare Tunnel  →  cloudflared  →  http://casaos-gateway:81
```

Tunnel sudah jalan dari kursus sebelumnya. Tinggal nambah ingress.

## 1. Cek Port Service

Catat port masing-masing service:

| Service | Port Internal | Container Name |
| --- | --- | --- |
| CasaOS | `81` | `casaos-gateway` |
| Uptime Kuma | `3001` | `uptime-kuma` |
| Dashy | `80` | `dashy` |
| Pi-hole (dari course sebelumnya) | `80` | `pihole` |

## 2. Cek Network

Pastikan container bisa saling akses:

```bash
docker network ls
```

Kalau ada `proxy-net` — bagus. Kalau belum:

```bash
docker network create proxy-net
```

Hubungkan setiap container ke `proxy-net`:

```bash
docker network connect proxy-net casaos-gateway
docker network connect proxy-net uptime-kuma
docker network connect proxy-net dashy
```

> **Catatan**: CasaOS container biasanya pake `casaos` network sendiri. Jangan lepas dari situ — cukup tambah `proxy-net` sebagai network kedua.

## 3. Tambah Ingress di Dashboard Tunnel

1. Buka Cloudflare Dashboard → **Zero Trust** → **Access** → **Tunnels**
2. Klik tunnel kamu
3. Tab **Public Hostname** → **Add a public hostname**

Tambah satu per satu:

### CasaOS
| Field | Isi |
| --- | --- |
| Subdomain | `casaos` |
| Domain | `example.com` |
| Type | `HTTP` |
| URL | `http://casaos-gateway:81` |

### Uptime Kuma
| Field | Isi |
| --- | --- |
| Subdomain | `uptime` |
| Domain | `example.com` |
| Type | `HTTP` |
| URL | `http://uptime-kuma:3001` |

### Dashy
| Field | Isi |
| --- | --- |
| Subdomain | `dashy` |
| Domain | `example.com` |
| Type | `HTTP` |
| URL | `http://dashy:80` |

### Pi-hole (dari kursus sebelumnya)

Kalau ada:

| Field | Isi |
| --- | --- |
| Subdomain | `pihole` |
| Domain | `example.com` |
| Type | `HTTP` |
| URL | `http://pihole:80` |

## 4. Verifikasi

```bash
# Cek apakah container bisa diakses tunnel
docker logs cloudflared --tail 20
```

Buka setiap URL di browser:

```
https://casaos.example.com
https://uptime.example.com
https://dashy.example.com
```

Semua harus muncul dengan HTTPS valid (gembok hijau).

## 5. Update Dashy dengan URL Final

Edit Dashy config — ganti semua URL dari `http://IP:port` ke `https://subdomain.example.com`:

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

## Hasil Akhir

- [ ] Semua service bisa diakses via `https://*.example.com`
- [ ] HTTPS valid — gembok hijau
- [ ] Dashy menampilkan semua service dengan link benar
- [ ] Tunnel menangani semua traffic — tidak ada port terbuka

## Berikutnya

[Modul 5 — Portainer (Opsional)](05_PORTAINER_OPTIONAL.md)
