# Modul 3: Dashy Dashboard

Dashy adalah central startpage. Buka `dashy.example.com` dan semua service terdaftar rapi — tinggal klik untuk akses.

## Cara Install

### Opsi A: Via CasaOS App Store (Mudah)

1. Buka CasaOS dashboard
2. **Apps** → **App Store**
3. Cari "Dashy" (atau "Dashy - Homepage")
4. Klik **Install**
5. Biarkan default

Dashy akan jalan di port 80 (dalam container).

### Opsi B: Manual via Docker

```bash
mkdir -p ~/dashy && cd ~/dashy
```

Buat `compose.yaml`:

```yaml
services:
  dashy:
    image: lissy93/dashy:latest
    container_name: dashy
    restart: unless-stopped
    ports:
      - "8080:80"
    volumes:
      - ./conf.yml:/app/user-data/conf.yml
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

## Konfigurasi

### Via UI (Mudah)

1. Buka `http://<IP_SERVER>:8080`
2. Klik ikon settings (gear) pojok kanan atas
3. Pilih **Switch to Edit Mode**
4. Mulai tambah section & item

### Via File (conf.yml)

Buka menu Settings → **Config Editor** → paste:

```yaml
pageInfo:
  title: Home Lab
  description: Self-Hosted Services
  logo: https://i.ibb.co/4T7B4bD/home.png

appConfig:
  theme: material
  language: id

sections:
  - name: Infrastruktur
    items:
      - title: CasaOS
        description: Server Manager
        url: https://casaos.example.com
        icon: https://cdn.jsdelivr.net/npm/simple-icons@v5/icons/casaos.svg
        statusCheck: true

      - title: Portainer
        description: Docker UI
        url: https://portainer.example.com
        icon: https://cdn.jsdelivr.net/npm/simple-icons@v5/icons/portainer.svg
        statusCheck: true

      - title: Uptime Kuma
        description: Monitoring
        url: https://uptime.example.com
        icon: https://cdn.jsdelivr.net/npm/simple-icons@v5/icons/uptimekuma.svg
        statusCheck: true

  - name: Jaringan
    items:
      - title: Pi-hole
        description: DNS Adblock
        url: https://pihole.example.com/admin
        icon: https://cdn.jsdelivr.net/npm/simple-icons@v5/icons/pihole.svg
        statusCheck: true
```

Ganti `example.com` dengan domain kamu. Simpan.

> **Catatan**: statusCheck otomatis nge-ping service dan kasih indikator hijau/merah — jadi langsung keliatan mana yang mati.

## Hasil Akhir

- [ ] Dashy berjalan — bisa diakses via `http://<IP>:8080`
- [ ] Semua service terdaftar di dashboard
- [ ] Status check aktif (hijau/merah)

## Berikutnya

[Modul 4 — Subdomain & Tunnel Integration](04_TUNNEL_INTEGRATION.md)
