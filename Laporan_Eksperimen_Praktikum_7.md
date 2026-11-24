# Laporan Hasil & Eksperimen Praktikum 7: Dokumentasi OpenAPI dan Automated Testing

* **Nama Mahasiswa**: Fardan Yudhistira
* **Stambuk**: F5512520080
* **Mata Kuliah**: Rekayasa API (F09252013)
* **Judul Praktikum**: Dokumentasi OpenAPI dan Pengujian API dengan Postman

---

## 1. Hasil 4 Eksperimen Pengembangan (Tugas D.2)

### Eksperimen 1: Melengkapi Dokumentasi OpenAPI v2
* Menambahkan operasi `PUT /v2/destinations/{id}`, `GET /v2/regions`, dan `GET /v2/regions/{id}`.
* Menambahkan skema komponen `Region` pada `components.schemas` dengan atribut `id`, `name`, `latitude`, dan `longitude`.
* Seluruh operasi memetakan status `200`, `401`, dan `404` secara lengkap.

### Eksperimen 2: Mendokumentasikan Versi Lama (v1) Secara Penuh
* Melengkapi seluruh metode pada `/v1/destinations` (`GET`, `POST`) dan `/v1/destinations/{id}` (`GET`, `PUT`, `DELETE`).
* Seluruh operasi diberi penanda `deprecated: true` selaras dengan masa transisi deprecation.
* Membuat skema `DestinationV1` dan `DestinationV1Input` yang mempertahankan representasi string `region` (bukan `regionId`).

### Eksperimen 3: Menyajikan Spesifikasi OpenAPI Langsung dari Server
* **Implementasi Kode pada `server.js`**:
  ```javascript
  // Disisipkan di atas pemeriksaan periksaApiKey agar endpoint dapat diakses publik
  if (segmen[0] === 'openapi.yaml' && segmen.length === 1) {
    if (req.method !== 'GET') {
      res.setHeader('Allow', 'GET');
      return kirimError(res, 405, 'Endpoint /openapi.yaml hanya menerima method GET');
    }
    const isiYaml = await fs.readFile(OPENAPI_FILE, 'utf8');
    res.writeHead(200, {
      'Content-Type': 'text/yaml; charset=utf-8',
      'Content-Length': Buffer.byteLength(isiYaml)
    });
    return res.end(isiYaml);
  }
  ```

### Eksperimen 4: Penambahan 5 Test Case Otomatis Baru (TC9 s.d. TC13)
* **TC9 (Payload Tidak Valid 400)**: Memastikan penambahan destinasi tanpa kategori ditolak dengan status 400.
* **TC10 (PUT Berhasil 200)**: Memperbarui nama destinasi id 1 menjadi "Danau Poso Eksotis" dan memverifikasi perubahan datanya.
* **TC11 (Header Sunset v1)**: Memanggil `GET /v1/destinations/1` dan memeriksa keberadaan header `Sunset: Wed, 30 Jun 2027 GMT` serta atribut `region`.
* **TC12 (GET Regions v2)**: Memeriksa endpoint koleksi daerah mengembalikan 5 data kabupaten/kota beserta koordinatnya.
* **TC13 (GET /openapi.yaml)**: Memeriksa server mengembalikan dokumen spesifikasi dengan Content-Type `text/yaml`.

---

## 2. Tabel Kesesuaian Dokumentasi OpenAPI dan Implementasi Pengujian

| Operasi API | Status Code Tercatat | Test Case Pembuktian | Hasil Eksekusi |
|:---|:---:|:---:|:---:|
| `POST /login` | `200 OK` | TC3 (Editor), TC7 (Admin) | **PASSED** (Token tersimpan ke variable) |
| `POST /login` | `401 Unauthorized` | Pengujian Khusus Login Salah | **PASSED** (Username/password salah) |
| `POST /login` | `429 Too Many Requests`| Pengujian 3x Gagal Beruntun | **PASSED** (Cooldown 60 detik aktif) |
| `GET /me` | `200 OK` | Pengujian Profil Token | **PASSED** (Menampilkan user & sisa detik) |
| `GET /v2/destinations` | `200 OK` | TC1 Daftar destinasi | **PASSED** (Memuat 4 field sesuai skema) |
| `GET /v2/destinations` | `401 Unauthorized` | TC1 (tanpa API key) | **PASSED** (Ditolak tanpa header x-api-key) |
| `POST /v2/destinations` | `201 Created` | TC5 Tambah dengan token | **PASSED** (Header Location & id baru) |
| `POST /v2/destinations` | `400 Bad Request` | TC9 Payload tidak valid | **PASSED** (Pesan: Field wajib belum diisi) |
| `POST /v2/destinations` | `401 Unauthorized` | TC4 Tambah tanpa token | **PASSED** (Status 401 belum login) |
| `GET /v2/destinations/{id}`| `200 OK` | Pengujian Detail ID 1 | **PASSED** (Data ditemukan) |
| `GET /v2/destinations/{id}`| `404 Not Found` | TC2 Destinasi tidak ditemukan | **PASSED** (ID 9999 menghasilkan 404) |
| `PUT /v2/destinations/{id}`| `200 OK` | TC10 Perbarui destinasi | **PASSED** (Nama terupdate) |
| `DELETE /v2/destinations/{id}`| `204 No Content`| TC8 Hapus oleh admin | **PASSED** (Body respon kosong) |
| `DELETE /v2/destinations/{id}`| `403 Forbidden` | TC6 Hapus oleh editor | **PASSED** (Editor ditolak) |
| `GET /v2/regions` | `200 OK` | TC12 Ambil daftar daerah | **PASSED** (Array 5 daerah) |
| `GET /v1/destinations/{id}`| `200 OK` | TC11 Periksa Header Sunset | **PASSED** (Header Sunset terverifikasi) |
| `GET /openapi.yaml` | `200 OK` | TC13 Mengambil openapi.yaml | **PASSED** (text/yaml diterima) |

### Perilaku Server Tambahan yang Ditemukan (Belum Masuk Skema):
1. **Validasi Tipe Data Parameter ID**: Pemanggilan `GET /v2/destinations/abc` menghasilkan `400 Bad Request` (`id harus berupa bilangan bulat positif`).
2. **Method Not Allowed**: Pemanggilan `PATCH /v2/destinations` menghasilkan status `405 Method Not Allowed` dengan header `Allow: GET, POST`.

---

## 3. Analisis Kritis Dokumentasi dan Pengujian

1. **Risiko Ketidaksesuaian Dokumentasi dan Implementasi**:
   Jika dokumentasi menyatakan suatu field bernama `region` bertipe string, namun server mengirim `regionId` bertipe number, aplikasi client yang mengonsumsi API tersebut akan mengalami kegagalan parsing / crash. Dokumentasi adalah kontrak hukum teknis; dokumentasi yang salah lebih berbahaya daripada tidak ada dokumentasi sama sekali.
2. **Pentingnya Test Case Negatif**:
   Test case positif hanya membuktikan sistem bekerja ketika input ideal. Sebaliknya, test case negatif (seperti TC2, TC4, TC6, TC9) memastikan sistem memiliki ketahanan (*resilience*) dan pertahanan keamanan (*defensive programming*) untuk menolak input yang tidak sah, kredensial palsu, maupun pelanggaran batas otorisasi.
3. **Keunggulan Format OpenAPI Dibanding Dokumentasi Bebas**:
   OpenAPI berbasis standar mesin (*machine-readable*). Dari satu file YAML:
   - Dapat dirender otomatis menjadi portal dokumentasi interaktif (Swagger UI).
   - Dapat diimpor langsung ke Postman menjadi koleksi request pengujian otomatis.
   - Dapat dipakai oleh generator kode (*openapi-generator*) untuk menghasilkan SDK client secara otomatis dalam puluhan bahasa pemrograman.

---

## 4. Kesimpulan
Automated testing dengan Postman Collection Runner yang dipasangkan dengan spesifikasi baku OpenAPI 3.0.3 menjamin bahwa API Wisata Sulawesi Tengah tidak hanya terdokumentasi dengan rapi, namun setiap janji kontrak fungsional dan keamanannya telah teruji dan terbukti valid secara otomatis.
