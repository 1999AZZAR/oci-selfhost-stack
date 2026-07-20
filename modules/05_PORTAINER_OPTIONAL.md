# Modul 5: Portainer — Docker Management UI (Opsional)

Portainer adalah web UI untuk mengelola Docker. Kalau CasaOS terlalu sederhana dan kamu mau kontrol penuh ke container— restart, lihat log, deploy compose dari browser — Portainer jawabannya.

## Cara Kerja

```
Browser → https://portainer.example.com
  → Tunnel → cloudflared
  → Portainer UI (port 9000)
  → Docker API via socket
  → Lihat & manage semua container
```

## Install

### 1. Buat Folder & compose.yaml

```bash
mkdir -p ~/portainer && cd ~/portainer
nano compose.yaml
```

```yaml
services:
  portainer:
    image: portainer/portainer-ce:lts
    container_name: portainer
    restart: unless-stopped
    command:
      - --http-enabled
    ports:
      - "9000:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./data:/data
    networks:
      - proxy-net

networks:
  proxy-net:
    external: true
```

> **Penjelasan**:
> - `--http-enabled` — Portainer modern defaultnya HTTPS (port 9443). Flag ini mengaktifkan HTTP di port 9000
> - `- /var/run/docker.sock` — akses ke Docker API. Ini yang bikin Portainer bisa lihat semua container
> - `:lts` — Long Term Support version. Lebih stabil dari `:latest`

### 2. Jalankan

```bash
docker compose up -d
```

### 3. Verifikasi

```bash
docker ps | grep portainer
```

Output:

```
<id>   portainer/portainer-ce:lts   ...   Up X seconds   0.0.0.0:9000->9000/tcp   portainer
```

## Setup Awal

1. Cari IP server: `curl -4 ifconfig.me` (publik) atau `tailscale ip -4` (Tailscale)
2. Buka browser: `http://<IP_PUBLIK_ATAU_TAILSCALE>:9000`
2. Buat **password admin** — minimal 8 karakter, kuat
3. Klik **Create user**
4. Pilih **Local** → klik **Connect**
5. Selesai — dashboard Portainer muncul

Di dashboard kamu bisa lihat:

| Card | Arti |
| --- | --- |
| **Stacks** | Compose project yang berjalan |
| **Containers** | Semua container (termasuk yang bukan dari Portainer) |
| **Images** | Docker images yang sudah didownload |
| **Volumes** | Data persistent |
| **Networks** | Semua Docker network |

## Tambah Tunnel (Kalau Mau)

Kalau kamu ingin akses Portainer via domain (sama seperti service lainnya):

1. Cloudflare Dashboard → Tunnel → **Add a public hostname**

| Field | Isi |
| --- | --- |
| Subdomain | `portainer` |
| Domain | `example.com` |
| Type | `HTTP` |
| URL | `http://portainer:9000` |

2. Save
3. Buka `https://portainer.example.com`

## Yang Bisa Dilakukan di Portainer

### Lihat Log Container
1. Klik **Containers** di kiri
2. Klik nama container (misal: `uptime-kuma`)
3. Klik tab **Logs**
4. Lihat log real-time — tanpa SSH

### Restart Container
1. **Containers** → centang container
2. Klik tombol **Restart** (atas)
3. Selesai — container restart

### Deploy Aplikasi Baru
1. Klik **Stacks** → **Add stack**
2. Nama: `my-app`
3. Build method: **Web editor**
4. Paste compose.yaml
5. Klik **Deploy the stack**

### Update Container
1. **Containers** → klik container
2. Klik **Recreate** → **Pull latest image** → centang
3. Klik **Recreate**

## Portainer vs CasaOS

| Aspek | CasaOS | Portainer |
| --- | --- | --- |
| **Kemudahan** | Sangat mudah — app store, 1 klik install | Butuh paham dasar Docker |
| **Kontrol** | Terbatas pada yang ada di app store | Full control — container, images, volumes, network |
| **Target** | Pemula, keluarga, non-teknis | System admin, advanced user |
| **Install apps** | Tinggal klik di UI | Manual compose atau stack |
| **Log container** | Terbatas | Real-time streaming, lengkap |

**Tips**: Pakai CasaOS untuk install cepat. Pakai Portainer kalau butuh debug atau deploy stack kustom.

## Hasil Akhir

- [ ] Portainer terinstall (kalau kamu pilih)
- [ ] Dashboard menampilkan semua container
- [ ] (Opsional) Bisa diakses via `https://portainer.example.com`
- [ ] Kamu bisa lihat log, restart, dan deploy dari UI

---

**Selamat!** Semua service sudah terinstall dan bisa diakses via domain + HTTPS. Sekarang kamu punya:

| URL | Service | Fungsi |
| --- | --- | --- |
| `https://casaos.example.com` | CasaOS | Server manager + app store |
| `https://uptime.example.com` | Uptime Kuma | Monitoring 24/7 |
| `https://dashy.example.com` | Dashy | Central startpage |
| `https://pihole.example.com` | Pi-hole | DNS adblock |
| `https://portainer.example.com` | Portainer | Docker UI (opsional) |

Semua aman di belakang Cloudflare Tunnel — tanpa port terbuka, semua HTTPS otomatis.
