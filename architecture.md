# Rancangan Arsitektur dan Desain Perangkat Lunak
**Sistem Pencatatan Hasil Tangkap Nelayan (Varian NIM: 2440304016 - Reza Randika Putra)**

## C1. Arsitektur Sistem (Layered Client-Server Architecture)
Sistem dirancang menggunakan **Layered Architecture (Arsitektur Berlapis)** berbasis Client-Server yang memisahkan tanggung jawab ke dalam 4 lapisan logis:
1. **Client / Presentation Layer (`User / Client`):** Antarmuka Web/Mobile responsif yang digunakan oleh Petugas Pencatat TPI, Nelayan, dan Pengelola TPI. Mengirimkan *payload* JSON ringan melalui HTTPS.
2. **Application / Controller Layer (`Application`):** Terdiri atas `AuthController`, `CatchController`, dan `ReportController` yang menangani routing REST API, validasi skema input awal, kompresi respons (Gzip), dan otorisasi token JWT.
3. **Business Logic / Service Layer (`Business Logic`):** Terdiri atas `CatchRecordService`, `RekapLaporanService`, dan `AuthService` yang mengeksekusi aturan bisnis perhitungan total volume tangkapan (kg), validasi zona lokasi WPP, dan agregasi rekapitulasi.
4. **Data Access & Storage Layer (`Data / Database`):** Terdiri atas `CatchRecordRepository`, *In-Memory Cache* (Redis/Local Memory Cache), dan basis data relasional (PostgreSQL/MySQL).

## C2. Design Pattern (Repository Pattern)
- **Masalah Desain:** Jika logika kueri SQL dan logika manajemen *cache* ditulis langsung di dalam kelas layanan bisnis (`CatchRecordService` dan `RekapLaporanService`), maka terjadi *tight coupling* dan duplikasi kode kueri yang menyulitkan pengujian serta pemeliharaan.
- **Pattern yang Dipilih:** **Repository Pattern** (`ICatchRepository` dan `SqlCachedCatchRepository`).
- **Bagian Sistem Tempat Diterapkan:** Lapisan antara *Business Logic Layer* (`CatchRecordService` & `RekapLaporanService`) dan *Data/Database Layer*.
- **Alasan Lebih Sesuai Dibanding Alternatif Lain:** Dibandingkan *Active Record Pattern* atau penulisan SQL langsung di *Service Layer*, *Repository Pattern* mengisolasi logika pengaksesan basis data dan mekanisme *cache invalidation* di satu tempat, sehingga perubahan skema tabel atau optimasi kueri untuk mengejar waktu respons <= 3 detik tidak merusak kode logika bisnis.

## C3. Trade-off Keputusan Desain
Saya memilih **Layered Monolithic Architecture yang dikombinasikan dengan In-Memory Caching (Redis) dan Database Indexing** karena mampu menjamin waktu respons fungsi utama di bawah 3 detik untuk 116 pengguna aktif bersamaan tanpa overhead komunikasi jaringan antar-layanan, tetapi konsekuensinya adalah peningkatan konsumsi memori RAM server untuk menyimpan *cache* rekapitulasi serta keharusan mengelola *cache invalidation* setiap kali ada input tangkapan baru. Pilihan ini tetap digunakan karena skala beban $P = 116$ pengguna aktif konkuren sangat ideal ditangani oleh satu *deployment unit* monolitik berlapis yang cepat, mudah dipelihara, dan jauh lebih efisien dibanding memecahnya menjadi *Microservices* yang kompleks.

## C4. Strategi Pemenuhan Constraint Waktu Respons <= 3 Detik pada P = 116 Pengguna Aktif (`feature-4016`)
Untuk memastikan waktu respons fungsi utama selalu berada di bawah **3 detik** ketika diakses oleh **P = 116 pengguna aktif bersamaan**:
1. **Database Connection Pooling:** Menggunakan *connection pool* (kapasitas 150 koneksi aktif) agar 116 request konkuren dari petugas pencatat TPI tidak perlu membuat *handshake* koneksi TCP/SQL baru dari nol (menghemat ~200–400 ms per transaksi).
2. **Composite B-Tree Indexing:** Menerapkan indeks komposit pada kolom `(waktu_tangkap, id_pengguna, kode_spesies)` sehingga kueri filter rekapitulasi berjalan dalam orde `< 50 ms` tanpa melakukan *full table scan*.
3. **In-Memory Caching (TTL 60 Detik) dengan Write-Through Invalidation:** Data rekapitulasi harian disimpan pada memori *cache*; pembacaan dashboard oleh Pengelola TPI/Dinas Perikanan dilayani langsung dari memori (`< 20 ms`), dan *cache* otomatis diperbarui ketika `CatchRecordRepository.save()` menyimpan tangkapan baru.
4. **Estimasi Anggaran Waktu Respons (Latency Budget):** Transmisi JSON ringan (`~150 ms`) + Validasi Controller & Kalkulasi Service (`~200 ms`) + Eksekusi SQL Terindeks & Update Cache (`~350 ms`) = **Total ~0,70 detik** (jauh di bawah batas maksimal **3 detik**).
