# Modul 3: Dashy Dashboard

Dashy adalah halaman startpage. Buka `dashy.example.com` dan semua service terdaftar rapi dengan icon — tinggal klik untuk akses.

## Cara Kerja

```
Browser → Dashy dashboard
           ├── icon CasaOS → https://casaos.example.com
           ├── icon Pi-hole → https://pihole.example.com
           ├── icon Uptime → https://uptime.example.com
           └── status check → hijau/merah (hidup/mati)
```

## Metode Install

---

## Opsi A: Via CasaOS App Store (Termudah)

1. Buka CasaOS dashboard di `http://<IP>:81`
2. **Apps** → **App Store**
3. Cari "Dashy"
4. Klik **Install**
5. Biarkan default — CasaOS handle semuanya

Dashy akan jalan setelah beberapa menit.

> **Catatan**: Pertama kali jalan, Dashy butuh 30-60 detik untuk build asset. Jangan panik kalau halaman putih. Tunggu.

---

## Opsi B: Manual via Docker (Advanced)

1. Buat folder:

```bash
mkdir -p ~/dashy && cd ~/dashy
```

2. Buat folder untuk konfigurasi:

```bash
mkdir -p user-data
```

3. Buat `compose.yaml`:

```bash
nano compose.yaml
```

```yaml
services:
  dashy:
    image: lissy93/dashy:latest
    container_name: dashy
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - ./user-data:/app/user-data
    environment:
      - NODE_ENV=production
    networks:
      - proxy-net

networks:
  proxy-net:
    external: true
```

> **PENTING**: Dashy pakai port internal **8080**, bukan 80. Jangan diganti.

4. Buat file konfigurasi awal `user-data/conf.yml`:

```bash
nano user-data/conf.yml
```

```yaml
pageInfo:
  title: My Homelab
  description: Self-Hosted Services
  navLinks: []

appConfig:
  theme: colorful
  statusCheck: true
  statusCheckInterval: 60
  layout: auto
  iconSize: medium
  language: id

sections:
  - name: Infrastruktur
    icon: fas fa-server
    items:
      - title: CasaOS
        description: Server Manager
        url: https://casaos.example.com
        icon: hl-casaos
        statusCheck: true

      - title: Uptime Kuma
        description: Monitoring
        url: https://uptime.example.com
        icon: hl-uptime-kuma
        statusCheck: true

      - title: Pi-hole
        description: DNS Adblock
        url: https://pihole.example.com/admin
        icon: hl-pihole
        statusCheck: true
```

Ganti `example.com` dengan domain kamu. Simpan.

5. Jalankan:

```bash
docker compose up -d
```

6. Verifikasi:

```bash
docker ps | grep dashy
```

Output:

```
<id>   lissy93/dashy:latest   ...   Up X minutes   0.0.0.0:8080->8080/tcp   dashy
```

---

## Setup Awal

1. Buka browser:

```
http://<IP_SERVER>:8080
```

2. Tunggu 30-60 detik — Dashy sedang build asset pertama kali
3. Dashboard akan muncul dengan service yang sudah kamu daftarkan di `conf.yml`

> **Kalau halaman putih terus**: Cek log: `docker logs dashy --tail 20`. Biasanya karena `conf.yml` ada error YAML (salah indentasi).

### Edit Konfigurasi Via UI

1. Klik icon **settings** (gear) di pojok kanan bawah
2. Pilih **Switch to Edit Mode**
3. Kamu bisa:
   - Klik item untuk edit URL, icon, nama
   - Klik **Add Item** untuk tambah service baru
   - Klik **Add Section** untuk bikin kategori baru

### Cara Dapat Icon

Dashy punya banyak icon built-in. Formatnya:

```
hl-pihole      → icon Pi-hole
hl-portainer   → icon Portainer
hl-casaos      → icon CasaOS
hl-uptime-kuma → icon Uptime Kuma
hl-nextcloud   → icon Nextcloud
```

Atau pakai icon dari [Font Awesome](https://fontawesome.com/icons):

```
fas fa-server    → icon server
fas fa-database  → icon database
fas fa-wifi      → icon wifi
fas fa-film      → icon media
fas fa-heartbeat → icon monitoring
```

Atau pakai URL gambar langsung:

```yaml
icon: https://i.ibb.co/logo-saya.png
```

## Verifikasi

```bash
# Cek container
docker ps | grep dashy

# Cek log (kalau ada error)
docker logs dashy --tail 20
```

Buka `http://<IP>:8080` — dashboard harus muncul dengan service-service yang terdaftar.

## Hasil Akhir

- [ ] Dashy berjalan di port 8080
- [ ] Semua service terdaftar di dashboard
- [ ] Status check aktif — icon hijau/merah
- [ ] Siap integrasi tunnel

## Berikutnya

[Modul 4 — Subdomain & Tunnel Integration](04_TUNNEL_INTEGRATION.md)
