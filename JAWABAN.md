# Jawaban Tes Debugging SIMRS Mini

Nama:
Tanggal:

Salin blok di bawah untuk setiap bug yang Anda temukan.

## Bug 1

- Gejala: `SQLSTATE[HY000] [1049] Unknown database 'simrs_mini'` saat membuka `GET /` (route `dashboard`). Error muncul di `StartSession` karena driver session `database` query tabel `sessions`.
- Lokasi (file:baris): `.env` baris `DB_DATABASE=simrs_mini` (config `config/database.php` -> `connections.mysql.database`), dipakai oleh `SESSION_DRIVER=database` -> `Illuminate\Session\DatabaseSessionHandler.php:117`.
- Penyebab: Database `simrs_mini` belum dibuat di MySQL server `127.0.0.1:3306` dan migrasi belum dijalankan, jadi tabel `sessions` tidak ada. Aplikasi langsung query tabel session sebelum request diproses.
- Perbaikan: Buat database lalu jalankan migrasi

## Bug 2

- Gejala: `Call to a member function translatedFormat() on string` saat membuka `GET /pasien` (route `pasien.daftar`).
- Lokasi (file:baris): `app/Models/Pasien.php:44` (`casts()`), dipakai di `resources/views/pasien/daftar.blade.php:65`.
- Penyebab: Nama kolom pada cast salah: `'tgl_lahir' => 'date'`, padahal kolom tabel `pasien` adalah `tanggal_lahir`. Karena key cast tidak cocok dengan nama atribut, `$pasien->tanggal_lahir` tetap berupa string mentah dari DB (bukan `Carbon`), sehingga `translatedFormat()` gagal dipanggil pada string. Efek samping: accessor `umur()` juga error karena `$this->tanggal_lahir->age`.
- Perbaikan: Ubah key cast menjadi nama kolom yang benar:
  ```php
  'tanggal_lahir' => 'date',
  ```
  Setelah itu `$pasien->tanggal_lahir` menjadi instance `Carbon` dan `translatedFormat('d-M-Y')` serta `umur` berjalan normal.

## Bug 3

- Gejala: Hapus Pasien tidak bekerja
- Lokasi (file:baris): simrs-mini\resources\views\dokter\detail.blade.php Line 29 (<form method="POST" action="{{ route('dokter.hapus', $dokter) }}" class="d-inline" onsubmit="return confirm(@js('Hapus data '.$dokter->nama.'? Data yang dihapus tidak dapat dikembalikan.'))">)
- Penyebab: Terjadi salah sasaran route dan variabel pada form hapus pasien. dialam file, form justru memanggil rute untuk menghapus dokter (dokter.hapus) variabel $dokter.
- Perbaikan: 