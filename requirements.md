# Spesifikasi Kebutuhan Perangkat Lunak (Software Requirements Specification)
**Sistem Pencatatan Hasil Tangkap Nelayan (Varian NIM: 2440304016 - Reza Randika Putra)**

## 1. Analisis Masalah, Model Proses, dan Stakeholder

### A1. Analisis Masalah
Proses pencatatan hasil tangkapan ikan di Tempat Pelelangan Ikan (TPI) dan pelabuhan pendaratan lokal saat ini masih menggunakan buku log kertas atau lembar rekapitulasi manual yang tidak terintegrasi. Ketika puluhan kapal nelayan bersandar secara bersamaan pada jam sibuk pendaratan pagi hari, Petugas Pencatat TPI, Nelayan/Nakhoda, Pengelola TPI, serta Dinas Perikanan Daerah mengalami kendala operasional yang serius. Akar permasalahan utama terletak pada ketiadaan sistem basis data terpusat yang mampu memvalidasi input jenis ikan, volume tangkapan (kg), titik lokasi Wilayah Pengelolaan Perikanan (WPP), dan waktu pendaratan secara waktu nyata (*real-time*), ditambah penyusunan rekapitulasi harian yang masih dihitung ulang secara manual. Apabila masalah ini tidak diselesaikan, antrean penimbangan dan pencatatan di dermaga akan terus menghambat kesegaran mutu ikan, meningkatkan risiko selisih atau kehilangan data transaksi nelayan, serta memperlambat pelaporan kuota tangkapan ke instansi perikanan. Selain itu, pada kondisi jam sibuk dengan asumsi beban **P = 116 pengguna aktif secara bersamaan**, proses pencatatan yang lambat (melebihi **3 detik** per transaksi) akan memicu penumpukan antrean input di lapangan.

### A2. Pemilihan Model Proses Perangkat Lunak
- **Model yang Dipilih:** **Incremental Model** (Pengembangan Inkremental Bertahap).
  1. **Prioritas Fungsi Inti & Constraint 3 Detik:** Modul pencatatan transaksi hasil tangkap, jenis/volume, dan lokasi/waktu (Inkremental 1) dapat dibangun dan diuji coba lebih awal untuk memastikan target waktu respons maksimal 3 detik pada beban 116 pengguna aktif tercapai sebelum menambahkan modul rekapitulasi analitik (Inkremental 2).
  2. **Adaptif terhadap Operasional TPI:** Petugas pencatat lapangan dapat langsung menggunakan fitur pencatatan dasar sambil memberikan umpan balik terkait ergonomi form input di perangkat *mobile*.
  3. **Isolasi Risiko Teknis:** Memudahkan pengujian performa basis data (*indexing* dan *connection pooling*) pada tiap kenaikan fitur tanpa menunggu seluruh sistem rampung.
- **Model Alternatif yang Kurang Sesuai:** **Waterfall Model**. Model sekuensial kaku ini mengharuskan seluruh tahap analisis, desain, dan pengkodean selesai 100% sebelum pengujian dilakukan, sehingga potensi kegagalan memenuhi *constraint* waktu respons <= 3 detik pada beban 116 pengguna baru akan terdeteksi di fase akhir dan sulit diperbaiki.

### A3. Analisis Stakeholder
| Stakeholder | Kepentingan | Kebutuhan | Pengaruh (T/S/R) | Risiko jika Diabaikan |
| :--- | :--- | :--- | :--- | :--- |
| **Petugas Pencatat TPI** | Mencatat jenis, volume, lokasi, dan waktu tangkap dengan cepat di lapangan | Formulir input ringkas dengan respons <= 3 detik saat jam sibuk | **Tinggi (T)** — Eksekutor utama input data harian di dermaga TPI | Sistem ditolak pengguna lapangan karena lambat dan menghambat antrean pendaratan ikan |
| **Nelayan / Nakhoda Kapal** | Memperoleh bukti pencatatan hasil tangkap yang akurat dan transparan | Bukti catat digital berisi rincian jenis/volume ikan dan waktu setor | **Sedang (S)** — Subjek penghasil data tangkapan namun jarang mengonfigurasi sistem | Timbul sengketa selisih timbangan/volume tangkapan dan ketidakpercayaan nelayan terhadap TPI |
| **Pengelola / Kepala TPI** | Memantau total volume pendaratan ikan harian dan kinerja petugas pencatat | Dashboard rekapitulasi harian/bulanan yang akurat dan instan | **Tinggi (T)** — Pemegang wewenang operasional dan kebijakan di pelabuhan/TPI | Evaluasi operasional pelabuhan terhambat dan rekapitulasi produksi harian tidak valid |
| **Dinas Perikanan Daerah** | Memantau stok perikanan tangkap dan kepatuhan zona lokasi tangkap (WPP) | Laporan rekapitulasi periodik per spesies dan lokasi tangkap | **Tinggi (T)** — Regulator yang menetapkan standar pendataan (seperti e-Logbook KKP) | Sistem tidak memenuhi standar pelaporan pemerintah daerah dan data kuota tangkap meleset |
| **Admin Sistem / Teknisi TI** | Menjaga ketersediaan server, keamanan hak akses, dan performa database | Panel manajemen akun pencatat dan pemantauan performa query <= 3 detik | **Sedang (S)** — Pengelola teknis infrastruktur pendukung operasional | Server mengalami *bottleneck* saat diakses 116 pengguna aktif bersamaan dan data rentan rusak |

- **Stakeholder Utama:** **Petugas Pencatat TPI**, karena mereka berinteraksi langsung dengan fungsi utama sistem pada jam puncak pendaratan kapal; apabila alur input tidak ergonomis atau melampaui batas respons 3 detik pada beban 116 pengguna aktif, seluruh rantai pengumpulan data perikanan akan lumpuh sejak titik awal.

## 2. Kebutuhan Fungsional (Functional Requirements)

```text
FR-01 - Autentikasi dan Otorisasi Pengguna Pencatat
Aktor: Petugas Pencatat TPI, Pengelola TPI, Admin Sistem
Deskripsi: Sistem memvalidasi kredensial login pengguna dan menerapkan hak akses berbasis peran (Role-Based Access Control) sehingga hanya petugas resmi yang dapat menginput atau memverifikasi catatan hasil tangkap.
Prioritas: High
Alasan: Menjamin akuntabilitas identitas pengguna pencatat pada setiap transaksi hasil tangkapan.
```

```text
FR-02 - Pencatatan Transaksi Hasil Tangkap Baru
Aktor: Petugas Pencatat TPI, Nelayan / Nakhoda Kapal
Deskripsi: Sistem menyediakan formulir pembuatan transaksi hasil tangkap baru yang mengaitkan identitas kapal/nelayan, ID pengguna pencatat, serta nomor bukti pencatatan unik secara otomatis.
Prioritas: High
Alasan: Merupakan fungsi inti untuk mendigitalisasi pencatatan pendaratan ikan di TPI.
```

```text
FR-03 - Input Rincian Jenis dan Volume Ikan
Aktor: Petugas Pencatat TPI
Deskripsi: Sistem memungkinkan petugas menambahkan satu atau lebih jenis/spesies ikan (misalnya Tuna, Tongkol, Cakalang) beserta volume beratnya dalam satuan kilogram (kg) dan menghitung total volume otomatis.
Prioritas: High
Alasan: Memenuhi ruang lingkup wajib pencatatan jenis dan volume tangkapan tanpa kesalahan hitung manual.
```

```text
FR-04 - Pencatatan Lokasi Tangkap dan Waktu Pendaratan
Aktor: Petugas Pencatat TPI
Deskripsi: Sistem mencatat kode Wilayah Pengelolaan Perikanan (WPP)/titik lokasi penangkapan, nama dermaga TPI, serta cap waktu (timestamp) penangkapan dan pendaratan secara akurat.
Prioritas: High
Alasan: Memenuhi ruang lingkup wajib pencatatan lokasi/waktu guna ketertelusuran asal ikan.
```

```text
FR-05 - Verifikasi dan Pembaruan Status Catatan Tangkap
Aktor: Petugas Pencatat TPI, Pengelola TPI
Deskripsi: Sistem menyediakan fitur verifikasi dan koreksi terotorisasi atas data tangkapan sebelum dikunci menjadi catatan final dalam rekapitulasi harian.
Prioritas: Medium
Alasan: Mencegah kesalahan ketik volume atau jenis ikan masuk ke laporan resmi.
```

```text
FR-06 - Penyusunan dan Pengunduhan Rekapitulasi Hasil Tangkap
Aktor: Pengelola TPI, Dinas Perikanan Daerah, Petugas Pencatat TPI
Deskripsi: Sistem menyajikan ringkasan dan rekapitulasi hasil tangkap berdasarkan rentang waktu, jenis ikan, lokasi tangkap, serta pengguna pencatat, lengkap dengan fitur ekspor laporan (CSV/PDF).
Prioritas: High
Alasan: Memenuhi ruang lingkup wajib rekapitulasi data untuk evaluasi operasional dan pelaporan regulasi.
```

## 3. User Story dan Acceptance Criteria

- **User Story 1 (US-01):**  
  `Sebagai Petugas Pencatat TPI, saya ingin mencatat data hasil tangkap yang memuat jenis ikan, volume (kg), lokasi tangkap, dan waktu pendaratan dengan cepat, sehingga antrean nelayan di dermaga terlayani tanpa hambatan.`
  - **Acceptance Criteria US-01:**
    1. Sistem menolak penyimpanan dan menampilkan pesan peringatan jelas apabila kolom jenis ikan, volume (kg), atau lokasi tangkap dikosongkan atau diisi nilai volume `<= 0`.
    2. Sistem secara otomatis merekam ID pengguna pencatat yang sedang aktif dan cap waktu (*timestamp*) transaksi saat tombol simpan ditekan.
    3. Proses penyimpanan catatan tangkap baru hingga menampilkan nomor bukti transaksi selesai dalam waktu **maksimal 3 detik** pada simulasi beban **116 pengguna aktif bersamaan**.

- **User Story 2 (US-02):**  
  `Sebagai Pengelola TPI, saya ingin melihat dan mengunduh rekapitulasi hasil tangkap berdasarkan periode waktu dan jenis ikan, sehingga saya dapat mengevaluasi total produksi harian dan melaporkannya ke Dinas Perikanan secara akurat.`
  - **Acceptance Criteria US-02:**
    1. Sistem dapat memfilter tabel rekapitulasi berdasarkan rentang tanggal, jenis spesies ikan, lokasi tangkap, dan nama petugas pencatat.
    2. Sistem menampilkan akumulasi total volume tangkapan (kg) dan jumlah transaksi yang konsisten 100% dengan rincian data yang diinput.
    3. Halaman rekapitulasi utama berhasil dimuat dalam waktu **maksimal 3 detik** pada beban **116 pengguna aktif bersamaan** berkat dukungan *caching* dan pengindeksan tabel.

## 4. Requirements Traceability Matrix (Matriks Keterlacakan)

| Requirement | Stakeholder | Use Case | Class Terkait |
| :--- | :--- | :--- | :--- |
| **FR-01** | Petugas Pencatat TPI, Admin Sistem | UC-01: Login & Verifikasi Hak Akses Pencatat | `PenggunaPencatat` |
| **FR-02** | Petugas Pencatat TPI, Nelayan | UC-02: Mencatat Transaksi Hasil Tangkap Baru | `CatatanTangkap`, `CatchRecordRepository` |
| **FR-03** | Petugas Pencatat TPI, Nelayan | UC-03: Menginput Detail Jenis & Volume Ikan (kg) | `DetailJenisIkan`, `CatatanTangkap` |
| **FR-04** | Petugas Pencatat TPI, Dinas Perikanan | UC-04: Mencatat Lokasi Tangkap & Waktu | `LokasiTangkap`, `CatatanTangkap` |
| **FR-05** | Petugas Pencatat TPI, Pengelola TPI | UC-05: Memverifikasi & Memperbarui Status Catatan | `CatatanTangkap`, `CatchRecordRepository` |
| **FR-06** | Pengelola TPI, Dinas Perikanan Daerah | UC-06: Melihat & Mengunduh Rekapitulasi | `RekapLaporanService`, `CatchRecordRepository` |

## 5. Kebutuhan Nonfungsional Terukur & Constraint Wajib NIM (Branch `feature-4016`)
- **NFR-01 (Performance & Constraint Wajib Digit Kedua dari Belakang NIM = 1):** Sistem wajib memproses penyimpanan transaksi pencatatan hasil tangkap dan menampilkan halaman rekapitulasi utama dengan **waktu respons maksimal X = 3 detik** pada kondisi beban puncak **P = 116 pengguna aktif secara bersamaan** (*concurrent active users*). *Alasan:* Sesuai constraint wajib NIM `2440304016` (digit kedua dari belakang = 1 -> X = 3 detik) dan mencegah antrean petugas pencatat di TPI saat ratusan kapal bersandar serentak.
- **NFR-02 (Scalability & Capacity):** Lapisan basis data dan *application server* harus mampu melayani minimal **116 koneksi pengguna aktif secara bersamaan** (hingga ambang *pool* 150 koneksi) dengan tingkat keberhasilan transaksi (*throughput success rate*) 99,9% tanpa *connection timeout*.
- **NFR-03 (Security):** Seluruh transmisi data wajib dienkripsi menggunakan protokol HTTPS (TLS 1.3), kata sandi akun pencatat di-*hash* menggunakan algoritma bcrypt (cost factor >= 10), dan setiap endpoint API memvalidasi token otorisasi berbasis peran (RBAC).
- **NFR-04 (Usability):** Antarmuka pencatatan hasil tangkap harus responsif pada layar smartphone (lebar 360px–1920px) dan memungkinkan petugas menyelesaikan satu input pencatatan tangkapan maksimal dalam 4 langkah interaksi.
- **NFR-05 (Reliability & Availability):** Sistem wajib memiliki tingkat ketersediaan (*uptime*) minimal 99,5% selama jam operasional pendaratan ikan (04.00–18.00 WITA) dengan validasi transaksi atomik (ACID) agar tidak ada catatan volume tangkapan yang tercatat ganda atau hilang sebagian.
