# Modul 5: Portainer — Docker Management UI (Opsional)

Portainer memberikan kontrol penuh ke Docker container lewat browser. Cocok kalau kamu merasa CasaOS terlalu terbatas untuk manage container.

## Install

### 1. Setup Docker Compose

```bash
mkdir -p ~/portainer && cd ~/portainer
```

Buat `compose.yaml`:

```yaml
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: unless-stopped
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

### 2. Jalankan

```bash
docker compose up -d
```

### 3. Akses

Buka `http://<IP_SERVER>:9000`

- Buat password admin
- Pilih **Local** → **Connect**
- Selesai — kamu lihat semua container di server

## Integrasi Tunnel

1. Cloudflare Dashboard → Tunnel → Add public hostname:

| Field | Isi |
| --- | --- |
| Subdomain | `portainer` |
| Domain | `example.com` |
| Type | `HTTP` |
| URL | `http://portainer:9000` |

2. Buka `https://portainer.example.com`

## Yang Bisa Dilakukan di Portainer

| Fitur | Fungsi |
| --- | --- |
| **Dashboard** | Lihat status semua container sekilas |
| **Containers** | Start, stop, restart, view logs dari UI |
| **Stacks** | Deploy compose.yaml langsung dari UI |
| **Volumes** | Manage persistent data |
| **Networks** | Buat & hubungkan network |
| **Logs** | Streaming log tanpa SSH |

## Portainer vs CasaOS

| Aspek | CasaOS | Portainer |
| --- | --- | --- |
| Kemudahan | Sangat mudah — app store | Perlu paham Docker |
| Kontrol | Terbatas pada app store | Full control |
| Target | Pemula, keluarga | Advanced user, sysadmin |
| Install apps | 1 klik | Manual compose |

Gunakan CasaOS untuk daily management. Gunakan Portainer kalau butuh debug container, lihat log, atau deploy stack kustom.

## Hasil Akhir

- [ ] Portainer terinstall (opsional)
- [ ] Bisa diakses via `https://portainer.example.com`
- [ ] Semua container visible di dashboard Portainer
- [ ] Siap deploy aplikasi apapun via compose dari UI
