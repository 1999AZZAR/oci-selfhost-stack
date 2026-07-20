# Self-Host Starter Pack — CasaOS, Dashy, Uptime Kuma & Portainer

Pasang CasaOS di OCI Always Free dan ubah server jadi dashboard visual: file manager, app store, monitoring, dan central startpage — semua lewat browser, tanpa perlu SSH tiap saat.

## Arsitektur

```
Browser → Cloudflare Tunnel → cloudflared (OCI)
                                ├── casaos.example.com   → CasaOS :81
                                ├── dashy.example.com    → Dashy :80
                                ├── uptime.example.com   → Uptime Kuma :3001
                                └── portainer.example.com → Portainer :9000
```

Semua subdomain via tunnel — tidak ada port terbuka, semua HTTPS otomatis.

## Prasyarat

Kursus ini melanjutkan dari [oci-domain-cloudflare](https://github.com/1999AZZAR/oci-domain-cloudflare). Kamu harus sudah punya:

- Domain + Cloudflare Tunnel berjalan di OCI
- Tunnel status **Healthy**
- Paham cara nambah ingress (Public Hostname) di dashboard tunnel

Belum punya? Selesaikan [kursus domain + tunnel](https://github.com/1999AZZAR/oci-domain-cloudflare) dulu.

## Daftar Modul

### Fase 1: Foundation
1. [Install CasaOS](modules/01_CASAOS_INSTALL.md) — web-based server manager

### Fase 2: Monitoring
2. [Uptime Kuma](modules/02_UPTIME_KUMA.md) — pantau semua service 24/7

### Fase 3: Central Dashboard
3. [Dashy Dashboard](modules/03_DASHY_DASHBOARD.md) — startpage untuk semua service

### Fase 4: Integrasi Tunnel
4. [Subdomain & Tunnel](modules/04_TUNNEL_INTEGRATION.md) — akses semua service via tunnel

### Fase 5: Advanced (Opsional)
5. [Portainer](modules/05_PORTAINER_OPTIONAL.md) — Docker management UI

---

**Catatan**: Resource semua service ini ringan — CasaOS ~128 MB, Dashy ~64 MB, Uptime Kuma ~128 MB. Total masih muat di OCI 12-24 GB RAM.

Tidak familiar dengan istilah teknis tertentu? Lihat [Glosarium](GLOSARIUM.md).
