# Praktikum Minggu 5 — CRUD Laravel API

## 1. Tujuan Praktikum

Praktikum ini bertujuan untuk mengembangkan API CRUD (Create, Read, Update, Delete) menggunakan Laravel. API dibuat untuk mengelola data buku dengan menerapkan validasi request, format respons menggunakan API Resource, penanganan data yang tidak ditemukan, pengujian melalui Postman, dokumentasi, dan pengelolaan perubahan kode menggunakan Git.

## 2. Endpoint API

Endpoint yang digunakan untuk mengelola data buku adalah sebagai berikut.

| No. | Method | Endpoint | Fungsi | Respons yang Diharapkan |
|---:|---|---|---|---|
| 1 | GET | `/api/books` | Menampilkan daftar buku | `200 OK` |
| 2 | POST | `/api/books` | Menambahkan buku baru | `201 Created` |
| 3 | GET | `/api/books/{id}` | Menampilkan detail buku berdasarkan ID | `200 OK`, atau `404 Not Found` jika tidak ada |
| 4 | PUT/PATCH | `/api/books/{id}` | Memperbarui data buku | `200 OK`, atau `404 Not Found` jika tidak ada |
| 5 | DELETE | `/api/books/{id}` | Menghapus buku berdasarkan ID | `204 No Content`, atau `404 Not Found` jika tidak ada |

Route diverifikasi menggunakan perintah:

```bash
php artisan route:list --path=api/books
```

Daftar route yang diharapkan memuat operasi GET daftar buku, POST, GET detail, PUT/PATCH, dan DELETE.

## 3. Validasi Request

Validasi input dikelola menggunakan Form Request agar aturan validasi terpisah dari controller.

### 3.1 StoreBookRequest — Create

Data buku yang digunakan dalam pengujian mengikuti kontrak praktikum, yaitu `title`, `isbn`, dan `available`.

Contoh payload:

```json
{
  "title": "Refactoring",
  "isbn": "9780134757599",
  "available": true
}
```

Aturan validasi create harus mengikuti implementasi aktual di `app/Http/Requests/StoreBookRequest.php`. Secara umum, aturan dapat mencakup:
- `title`: wajib diisi dan harus berupa teks.
- `isbn`: wajib diisi, berupa teks, serta mengikuti aturan keunikan atau format yang ditentukan aplikasi.
- `available`: wajib diisi jika kontrak mewajibkannya dan harus berupa boolean.

> Catatan: Sesuaikan uraian aturan di atas dengan rules yang benar-benar ada di file `StoreBookRequest.php`.

### 3.2 UpdateBookRequest — Update

Update menggunakan `app/Http/Requests/UpdateBookRequest.php`. Karena endpoint mendukung pembaruan sebagian melalui PATCH, hanya field yang dikirim pengguna yang perlu diperbarui. Field yang tidak dikirim harus tetap mempertahankan nilai sebelumnya.

Aturan update yang sebenarnya perlu dicatat sesuai isi `UpdateBookRequest.php`. Pastikan aturan untuk `isbn` mempertimbangkan data buku yang sedang diperbarui jika ISBN harus unik.

## 4. Format Respons

Respons sukses untuk data buku menggunakan `BookResource` pada `app/Http/Resources/BookResource.php`. Resource bertujuan menjaga struktur JSON API tetap konsisten dan hanya menampilkan field yang ditentukan.

Contoh payload create:

```json
{
  "title": "Refactoring",
  "isbn": "9780134757599",
  "available": true
}
```

Struktur respons aktual dapat memiliki pembungkus seperti `data`, bergantung pada implementasi `BookResource` dan konfigurasi Laravel. Karena itu, hasil akhir harus dibandingkan dengan respons yang benar-benar dihasilkan aplikasi.

Untuk data yang tidak ditemukan, kontrak praktikum mengharapkan respons `404 Not Found` seperti berikut:

```json
{
  "message": "Book not found",
  "errors": null
}
```

Untuk operasi DELETE yang berhasil, API harus mengembalikan status `204 No Content` tanpa body respons.

## 5. Pengujian API dengan Postman

Seluruh request API menggunakan header berikut:

```http
Accept: application/json
Content-Type: application/json
```

Header `Content-Type` digunakan saat request mengirim body JSON. Request GET umumnya tidak memerlukan body.

### 5.1 Create Berhasil — 201 Created

- Method: `POST`
- URL: `/api/books`
- Body: JSON dengan field `title`, `isbn`, dan `available`.
- Hasil yang diharapkan: `201 Created`.

Contoh body:

```json
{
  "title": "Refactoring",
  "isbn": "9780134757599",
  "available": true
}
```

### 5.2 Validasi Gagal — 422 Unprocessable Content

- Method: `POST`
- URL: `/api/books`
- Body: data yang tidak memenuhi aturan validasi, misalnya menghilangkan field wajib.
- Hasil yang diharapkan: `422 Unprocessable Content` dengan informasi kesalahan validasi dalam format JSON.

Contoh body untuk menguji validasi gagal:

```json
{
  "title": "",
  "isbn": "",
  "available": "bukan-boolean"
}
```

Hasil aktual bergantung pada rules yang ditetapkan aplikasi.

### 5.3 Update Sebagian — 200 OK

- Method: `PATCH`
- URL: `/api/books/{id}`
- Body: hanya field yang ingin diubah.

Contoh untuk mengubah ketersediaan buku:

```json
{
  "available": false
}
```

Hasil yang diharapkan: `200 OK`. Field lain yang tidak dikirim harus tetap sama.

### 5.4 Not Found — 404 Not Found

- Method: `GET`
- URL: `/api/books/999999`
- Hasil yang diharapkan: `404 Not Found`, dengan respons JSON:

```json
{
  "message": "Book not found",
  "errors": null
}
```

ID `999999` digunakan sebagai ID yang diperkirakan tidak ada. Pastikan ID tersebut memang tidak terdapat di database saat pengujian.

### 5.5 Delete — 204 No Content

- Method: `DELETE`
- URL: `/api/books/{id}`
- Gunakan ID buku yang benar-benar ada.
- Hasil yang diharapkan: `204 No Content` tanpa body.

Setelah DELETE berhasil, kirim request:

```http
GET /api/books/{id}
```

Hasil yang diharapkan adalah `404 Not Found`, untuk memastikan buku sudah tidak tersedia.dan saya juga sudah mencobannya dan hasilnnya 404 not found setelah buku dihapuskan

## 6. Perbandingan Contract dan Actual Response

Contract adalah hasil yang ditetapkan dalam instruksi praktikum, sedangkan actual response adalah hasil yang benar-benar dikembalikan oleh aplikasi ketika diuji.

| Skenario | Contract yang Diharapkan | Actual Response | Status |
|---|---|---|---|
| Create valid | `201 Created` | Isi sesuai hasil Postman | Isi setelah pengujian |
| Create invalid | `422 Unprocessable Content` | Isi sesuai hasil Postman | Isi setelah pengujian |
| Update valid | `200 OK` | Isi sesuai hasil Postman | Isi setelah pengujian |
| Buku tidak ditemukan | `404 Not Found` | Isi sesuai hasil Postman | Isi setelah pengujian |
| Delete berhasil | `204 No Content`, body kosong | Isi sesuai hasil Postman | Isi setelah pengujian |

Perbandingan akhir harus dilakukan berdasarkan status, struktur JSON, field yang dikembalikan, dan isi body. Jangan menandai skenario berhasil sebelum actual response diperiksa di Postman.

## 7. Penggunaan AI

Bagian ini mencatat penggunaan AI selama praktikum. Sesuaikan dengan penggunaan yang benar-benar dilakukan.

| Aspek | Catatan |
|---|---|
| Alat | ChatGPT |
| Tujuan penggunaan | Memahami pesan Git, menyusun langkah pengujian API di Postman, dan membantu menyusun dokumentasi praktikum |
| Contoh prompt | “Bagaimana cara menguji endpoint POST, PATCH, GET not found, dan DELETE di Postman?” |
| Hasil yang digunakan | Penjelasan alur request, header JSON, payload pengujian, status HTTP yang diharapkan, dan kerangka laporan |
| Cara memeriksa | Memeriksa route dengan `php artisan route:list --path=api/books`, menguji request di Postman, serta mencocokkan respons aktual dengan kontrak praktikum dan kode Laravel |
| Verifikasi mandiri | Memeriksa `BookController`, `StoreBookRequest`, `UpdateBookRequest`, `BookResource`, dan `routes/api.php` |

AI digunakan sebagai alat bantu pemahaman dan penyusunan dokumentasi. Kebenaran implementasi dan hasil pengujian tetap diverifikasi melalui kode aplikasi dan respons aktual. Jangan mencantumkan credential, password, token, API key, cookie, atau secret lain dalam laporan.
