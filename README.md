# Automation Stack Deployment

Repository ini berisi konfigurasi Docker Compose untuk mendeploy automation stack yang terdiri dari n8n, Evolution API, dan Redis. Stack ini dirancang agar mudah digunakan, di-maintain, dan dijalankan di server VPS (seperti Ubuntu 24.04).

## Penjelasan Singkat Service

1. **n8n**: Workflow automation tool berbasis node. Menggunakan database default (SQLite) untuk menyimpan data pada local volume (`./data/n8n`).
2. **Evolution API**: WhatsApp API provider yang memungkinkan interaksi via HTTP request. Dikonfigurasi untuk menggunakan Neon PostgreSQL sebagai database dan Redis container lokal sebagai cache.
3. **Redis**: In-memory data store yang digunakan oleh Evolution API untuk caching (tidak diekspos ke internet untuk alasan keamanan).

## Prasyarat Server
- OS: Ubuntu 24.04 (atau Linux distribution lainnya)
- Docker Engine & Docker Compose plugin sudah terinstall
- Akun dan database PostgreSQL yang sudah disiapkan di [Neon](https://neon.tech/)

---

## 1. Clone Repository
Clone repository ini ke server Anda:
```bash
git clone <URL_REPOSITORY_ANDA> automation-stack
cd automation-stack
```

## 2. Copy & Konfigurasi Environment Variables
Gandakan file environment example untuk masing-masing service:
```bash
cp evolution.env.example evolution.env
cp n8n.env.example n8n.env
```

Buka file konfigurasi dan sesuaikan isinya:
- **`n8n.env`**: Konfigurasikan timezone, URL, atau authentikasi tambahan.
- **`evolution.env`**: Anda *wajib* menyesuaikan API Key dan database connection string.

## 3. Konfigurasi Neon PostgreSQL
Untuk mengamankan data dan mempermudah deployment, Evolution API pada stack ini menggunakan database cloud dari Neon PostgreSQL.
1. Buat project dan database di Neon.
2. Dapatkan connection string (biasanya dimulai dengan `postgresql://` atau `postgres://`).
3. Tambahkan `?sslmode=require` pada akhir connection string.
4. Paste connection string tersebut ke variabel `DATABASE_CONNECTION_URI` di dalam file `evolution.env`.

*Contoh:*
```env
DATABASE_CONNECTION_URI=postgres://user:password@ep-cold-shadow-123456.us-east-2.aws.neon.tech/neondb?sslmode=require
```

## 4. Menjalankan Docker Compose
Setelah semua file environment sudah dikonfigurasi, jalankan stack dengan perintah:
```bash
docker compose up -d
```
Tunggu beberapa saat hingga seluruh service berhasil dijalankan (healthy). Anda dapat memantau log dengan:
```bash
docker compose logs -f
```

Stack akan dapat diakses di:
- **n8n**: `http://<IP_VPS_ANDA>:5678`
- **Evolution API**: `http://<IP_VPS_ANDA>:8080`

---

## Manajemen Stack

### Update Image
Untuk mengupdate seluruh service ke versi terbaru (sesuai tag di `docker-compose.yml`), jalankan:
```bash
docker compose pull
docker compose up -d --remove-orphans
```
*Note: Tidak perlu khawatir data hilang karena seluruh data tersimpan dengan aman di dalam folder `./data/`.*

### Backup Volume
Untuk mem-backup data, Anda hanya perlu membuat arsip dari direktori `data` (atau clone repository utuh jika dibutuhkan).
1. Hentikan container terlebih dahulu agar tidak ada proses penulisan ke database (khususnya SQLite di n8n):
```bash
docker compose stop
```
2. Buat backup (contoh dengan `tar`):
```bash
tar -czvf backup-automation-data-$(date +%F).tar.gz ./data/
```
3. Jalankan kembali container:
```bash
docker compose start
```

### Restore Volume
Jika Anda berpindah server, Anda dapat me-restore volume yang sudah dibackup.
1. Ekstrak file backup pada direktori repository baru:
```bash
tar -xzvf backup-automation-data-YYYY-MM-DD.tar.gz
```
2. Jalankan docker compose:
```bash
docker compose up -d
```
Seluruh data (seperti workflow n8n) akan otomatis dipulihkan.
