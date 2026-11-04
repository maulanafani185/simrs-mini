# Jawaban Tes Debugging SIMRS Mini

Nama:
Tanggal:

Salin blok di bawah untuk setiap bug yang Anda temukan.

## Bug 1

- Gejala: `SQLSTATE[HY000] [1049] Unknown database 'simrs_mini'` saat membuka `GET /` (route `dashboard`). Error muncul di `StartSession` karena driver session `database` query tabel `sessions`.
- Lokasi (file:baris): `.env` baris `DB_DATABASE=simrs_mini` (config `config/database.php` -> `connections.mysql.database`), dipakai oleh `SESSION_DRIVER=database` -> `Illuminate\Session\DatabaseSessionHandler.php:117`.
- Penyebab: Database `simrs_mini` belum dibuat di MySQL server `127.0.0.1:3306` dan migrasi belum dijalankan, jadi tabel `sessions` tidak ada. Aplikasi langsung query tabel session sebelum request diproses.
- Perbaikan: Buat database lalu jalankan migrasi:
  ```
  & "C:\laragon\bin\mysql\mysql-8.4.3-winx64\bin\mysql.exe" -u root -e "CREATE DATABASE IF NOT EXISTS simrs_mini CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
  php artisan migrate
  ```
  Setelah `php artisan migrate` sukses (tabel `sessions`, `users`, `cache`, `jobs`, `dokter`, `pasien` dibuat), error hilang dan `GET /` kembali normal.
