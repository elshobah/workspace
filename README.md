# Paperclip — Self-Hosted Stack (Dokploy)

Stack deployment [Paperclip](https://paperclip.ing) self-hosted via [Dokploy](https://dokploy.com) menggunakan Docker Compose.

## Stack

| Komponen | Image |
|---|---|
| Paperclip | `ghcr.io/paperclipai/paperclip:latest` |
| PostgreSQL | `postgres:17-alpine` |
| Redis | `redis:7-alpine` |

## Cara Deploy di Dokploy

1. **Buat project baru** → pilih **Docker Compose**
2. **Source** → paste isi `Docker-compose.yml` atau hubungkan ke Git repo ini
3. **Tab Environment** → paste semua variable dari `.env.example`, isi nilainya
4. **Tab Domains** → tambah domain → arahkan ke port `3100`
   - Dokploy + Traefik handle SSL otomatis via Let's Encrypt
5. **Deploy**

## Environment Variables

Salin `.env.example` menjadi `.env`, lalu isi semua nilai yang wajib.

| Variable | Keterangan |
|---|---|
| `PAPERCLIP_PUBLIC_URL` | URL publik aplikasi, contoh: `https://workspace.elshobah.com` |
| `PAPERCLIP_ALLOWED_HOSTNAMES` | Hostname yang diizinkan, tanpa `https://` |
| `BETTER_AUTH_SECRET` | Secret untuk auth — generate: `openssl rand -hex 32` |
| `POSTGRES_PASSWORD` | Password PostgreSQL — generate: `openssl rand -hex 16` |
| `REDIS_PASSWORD` | Password Redis — generate: `openssl rand -hex 16` |
| `ANTHROPIC_API_KEY` | API key dari [console.anthropic.com](https://console.anthropic.com) |
| `OPENAI_API_KEY` | Opsional — jika ada agent yang pakai model OpenAI |
| `LOG_LEVEL` | Opsional — `error` / `warn` / `info` / `debug` (default: `info`) |

## Arsitektur

```
Internet
   │
   ▼
Traefik (Dokploy built-in) ← SSL otomatis
   │
   ▼ port 3100
paperclip (app)
   ├── db (PostgreSQL 17) — internal network
   └── redis (Redis 7)    — internal network
```

- PostgreSQL dan Redis **tidak** terekspos ke host, hanya bisa diakses dari internal network compose.
- Semua data disimpan di **named volumes** — kompatibel dengan fitur Dokploy Volume Backups ke S3.

## Backup

Dokploy mendukung backup otomatis named volumes ke S3. Volume yang tersedia:

| Volume | Isi |
|---|---|
| `elshobah_paperclip_data` | Data aplikasi Paperclip |
| `elshobah_postgres_data` | Database PostgreSQL |
| `elshobah_redis_data` | Persistensi Redis (AOF) |

Aktifkan di Dokploy → **Volumes** → pilih volume → **Backup**.
