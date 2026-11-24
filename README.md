# API Wisata Sulawesi Tengah (Versi 2.1.0)

Repositori resmi backend API Wisata Sulawesi Tengah dengan dukungan **URI & Header Versioning**, **GraphQL**, dan **Sistem Keamanan Berlapis (API Key & JWT RBAC)**.

* **Pengembang**: Fardan Yudhistira
* **Stambuk**: F5512520080
* **Mata Kuliah**: Rekayasa API (F09252013)

---

## 1. Persiapan Awal (Setup Environment)

Berkas kredensial dan secret sengaja dikecualikan dari repositori Git melalui `.gitignore` untuk mencegah kebocoran data. Pengguna yang mengkloning repositori ini wajib menyiapkan file berikut:

### a. Buat File `.env`
Buat file bernama `.env` di root folder dan tentukan secret signature JWT:
```env
TOKEN_SECRET=kunci-rahasia-jwt-rekayasa-api-sulteng-2026-fardan-tugas
```

### b. Daftarkan API Key Client
Pastikan file `data/api-keys.json` tersedia dengan daftar aplikasi yang diizinkan:
```json
[
  { "client": "Aplikasi Wisata Sulteng (Android)", "key": "sulteng-mobile-7f3a9c21" },
  { "client": "Website Dinas Pariwisata", "key": "sulteng-web-b84d1e06" }
]
```

### c. Daftarkan Pengguna (User Seeding)
Jalankan skrip `buat-user.js` untuk membuat akun administrator dan editor:
```bash
node buat-user.js admin tadulako2026 admin
node buat-user.js editor kaledo2026 editor
node buat-user.js F5512520080 kaledo2026 editor
node buat-user.js tamu tamu2026 viewer
```

---

## 2. Cara Menjalankan Server

Pastikan Node.js versi 20.6 ke atas telah terpasang, lalu jalankan:
```bash
# Jalankan server dengan membaca environment variable
node --watch --env-file=.env server.js
```
Server akan aktif di `http://localhost:3000`.

---

## 3. Daftar Versi API dan Siklus Hidup (Lifecycle)

API ini melayani dua cara pemanggilan versi:

### A. Versi 1 (`/v1`) — Deprecated
* **Bentuk Data**: Menggunakan representasi nama wilayah langsung (`"region": "Poso"`).
* **Status**: *Deprecated* (Ditandai dengan header response `Sunset: Wed, 30 Jun 2027 00:00:00 GMT`).
* **Tanggal Berakhir Layanan**: **30 Juni 2027**.

### B. Versi 2 (`/v2`) — Aktif & Direkomendasikan
* **Bentuk Data**: Menggunakan relasi id wilayah (`"regionId": 2`) dan mendukung resource `/v2/regions`.
* **Fitur Filter**: Mendukung query parameter filter wilayah: `GET /v2/destinations?regionId=3`.

### C. Header Versioning
* Client dapat memanggil endpoint bersih tanpa awalan URI (misal `GET /destinations/1`) dengan menyertakan header:
  `x-api-version: 1` atau `x-api-version: 2`.

### D. GraphQL (`/graphql`)
* Mengakses seluruh data relasi destinasi, wilayah, dan prakiraan cuaca Open-Meteo secara fleksibel melalui satu pintu.
* **Evolusi Skema**: Field `windSpeed` ditandai `@deprecated(reason: "Gunakan field windSpeedKmh")` dan digantikan oleh field `windSpeedKmh`.
