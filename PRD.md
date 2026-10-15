# Product Requirements Document (PRD)

## Roomie — Smart Classroom Reservation & Monitoring System

**Versi:** 1.0  
**Status:** Draft untuk MVP  
**Platform:** Web responsif (desktop dan mobile)  
**Tech Stack:** Laravel, MySQL, Blade, Tailwind CSS  
**Target Pengguna:** Mahasiswa dan administrator  
**Ruang Lingkup:** Dashboard, monitoring ruangan, reservasi, QR scanning, checkout, dan fondasi integrasi IoT.

---

## 1. Product Overview

### 1.1 Latar Belakang

Penggunaan ruang kelas di lingkungan kampus membutuhkan informasi ketersediaan yang mudah diakses. Pengguna sering kali harus memeriksa ruangan secara langsung untuk mengetahui apakah ruangan sedang digunakan atau tersedia.

Roomie dikembangkan sebagai sistem berbasis web yang membantu pengguna melihat status ruang kelas, melakukan reservasi, serta mencatat penggunaan aktual ruangan melalui mekanisme check-in dan checkout menggunakan QR code.

Pada tahap selanjutnya, Roomie akan terintegrasi dengan ESP32 yang menggunakan sensor PIR dan magnetic reed switch untuk memperoleh informasi aktivitas dan kondisi pintu ruangan.

### 1.2 Problem Statement

1. Pengguna tidak memiliki satu tempat untuk melihat ketersediaan ruang kelas.
2. Proses pemesanan ruangan berpotensi menimbulkan konflik jadwal.
3. Tidak terdapat mekanisme sederhana untuk mencatat waktu penggunaan aktual.
4. Status reservasi tidak selalu mencerminkan kondisi fisik ruangan.
5. Sistem membutuhkan fondasi web yang dapat diintegrasikan dengan perangkat IoT pada fase berikutnya.

### 1.3 Product Vision

Menjadi platform sederhana untuk membantu mahasiswa memeriksa ketersediaan ruang kelas, melakukan reservasi, dan mencatat penggunaan ruangan secara transparan melalui satu aplikasi web.

### 1.4 MVP Goals

- Menyediakan dashboard informasi ketersediaan ruang kelas.
- Menyediakan halaman monitoring untuk setiap ruangan.
- Memungkinkan pengguna melakukan reservasi berdasarkan waktu.
- Memungkinkan pengguna check-in dan checkout melalui QR code.
- Mencegah konflik reservasi pada ruangan dan rentang waktu yang sama.
- Menyediakan data dan struktur sistem yang siap diintegrasikan dengan ESP32.

### 1.5 Non-Goals

Fitur berikut tidak termasuk dalam MVP awal:

- Integrasi langsung dengan sistem akademik kampus.
- Pembayaran reservasi.
- Notifikasi WhatsApp, SMS, atau push notification.
- Computer vision atau penghitungan jumlah orang.
- Sistem akses pintu fisik untuk gedung sungguhan.
- Analitik penggunaan ruangan tingkat lanjut.
- Aplikasi mobile native.
- Multi-campus dan multi-tenant.

---

## 2. Target Users dan Roles

### 2.1 Mahasiswa

Pengguna utama yang ingin mengetahui ketersediaan kelas, membuat reservasi, serta mencatat penggunaan ruangan.

**Kebutuhan:**

- Melihat daftar ruangan dan statusnya.
- Memindai QR code ruangan.
- Membuat reservasi dengan mengisi informasi perkuliahan.
- Melihat reservasi miliknya.
- Melakukan check-in dan checkout.
- Melihat riwayat penggunaan ruangan.

### 2.2 Administrator

Pengelola data ruang kelas dan operasional aplikasi.

**Kebutuhan:**

- Mengelola data ruangan.
- Melihat seluruh reservasi.
- Melihat sesi penggunaan yang aktif.
- Memantau status perangkat IoT jika sudah terintegrasi.
- Membatalkan reservasi atau menangani sesi yang bermasalah.

### 2.3 ESP32 (Future Integration)

ESP32 bukan pengguna manusia, tetapi perangkat eksternal yang nantinya mengirimkan telemetri sensor dan menerima perintah tertentu melalui API yang terautentikasi.

Integrasi ini berada di luar implementasi fungsional MVP web, tetapi kontrak datanya perlu dipersiapkan.

---

## 3. Product Scope

### In Scope

1. Authentication dan authorization.
2. Dashboard.
3. Daftar dan detail ruangan.
4. Monitoring status ruangan.
5. QR code scanning.
6. Formulir reservasi.
7. Validasi konflik jadwal.
8. Check-in dan checkout.
9. Riwayat reservasi dan sesi penggunaan.
10. Admin room management.
11. Data dummy untuk status sensor sebelum integrasi ESP32.

### Out of Scope

1. Integrasi hardware yang sesungguhnya.
2. QR dinamis berbasis waktu.
3. Persetujuan dosen atau biro administrasi.
4. Jadwal akademik otomatis.
5. Prediksi ketersediaan menggunakan AI.
6. Pembukaan kunci fisik menggunakan servo sungguhan.

---

## 4. Functional Requirements

### FR-01 — Authentication

Sistem harus menyediakan mekanisme login dan logout.

- Pengguna yang belum login hanya dapat mengakses halaman publik yang ditentukan.
- Pengguna yang login dapat membuat dan mengelola reservasinya sendiri.
- Administrator memiliki hak akses untuk mengelola data ruangan dan seluruh reservasi.
- Sistem harus membatasi akses berdasarkan role.
- Password disimpan dalam bentuk hash, bukan teks biasa.

**Acceptance Criteria:**

- Pengguna dapat login dengan kredensial yang valid.
- Login dengan kredensial yang salah ditolak.
- Pengguna biasa tidak dapat mengakses endpoint admin.
- Pengguna tidak dapat mengubah atau mengakhiri sesi milik pengguna lain tanpa otorisasi.

### FR-02 — Dashboard

Dashboard menampilkan empat indikator utama:

1. Total ruangan.
2. Jumlah ruangan tersedia.
3. Jumlah ruangan sedang terpakai.
4. Jumlah ruangan yang statusnya tidak diketahui atau perlu diperiksa.

Dashboard juga menampilkan daftar monitoring ruangan dan tombol utama untuk melakukan scan QR.

**Acceptance Criteria:**

- Statistik dihitung dari data database, bukan angka hardcoded.
- Daftar ruangan menampilkan nama, status ketersediaan, dan kondisi monitoring.
- Pengguna dapat membuka detail ruangan.
- Status tidak diketahui tidak dihitung sebagai ruangan tersedia.
- Dashboard dapat digunakan pada layar desktop maupun smartphone.

### FR-03 — Room Management

Setiap ruangan mempunyai identitas unik.

Data minimal:

- ID ruangan.
- Nama atau kode ruangan.
- Gedung dan lantai (opsional untuk MVP).
- Kapasitas (opsional).
- Status aktif atau nonaktif.
- QR token unik.
- Waktu pembaruan terakhir status monitoring.

Administrator dapat membuat, melihat, memperbarui, dan menonaktifkan ruangan.

**Acceptance Criteria:**

- QR token mengidentifikasi tepat satu ruangan.
- Ruangan nonaktif tidak dapat menerima reservasi baru.
- Ruangan yang sudah memiliki riwayat penggunaan tidak dihapus secara permanen melalui operasi biasa.
- Administrator dapat melihat atau mencetak QR code untuk setiap ruangan.

### FR-04 — Room Monitoring

Halaman monitoring menampilkan status ruangan berdasarkan reservasi, sesi aktif, dan telemetri perangkat jika tersedia.

Sistem harus membedakan:

- **Availability status:** apakah ruangan dapat dipesan pada rentang waktu yang diminta.
- **Usage status:** apakah terdapat sesi penggunaan aktif.
- **Physical status:** indikasi kondisi ruangan berdasarkan sensor.
- **Device status:** apakah perangkat IoT sedang online.

Sebelum ESP32 diintegrasikan, data fisik dan perangkat boleh ditampilkan sebagai `UNKNOWN` atau menggunakan data simulasi yang diberi label jelas.

**Acceptance Criteria:**

- Status reservasi tidak secara otomatis dianggap sebagai bukti kondisi fisik ruangan.
- Perangkat offline tidak membuat ruangan otomatis dianggap kosong.
- Waktu pembaruan sensor ditampilkan jika data tersedia.
- Pengguna dapat melihat alasan status tidak diketahui jika informasi tersebut tersedia.

### FR-05 — QR Scanning

Sistem menyediakan tombol Scan QR yang membuka kamera smartphone melalui browser.

QR code berisi token identifikasi ruangan, bukan data sensitif pengguna.

Alur:

1. Pengguna membuka scanner.
2. Browser meminta izin kamera.
3. Sistem membaca QR code.
4. Frontend mengirim token ke backend untuk validasi.
5. Backend mengembalikan detail ruangan.
6. Sistem menampilkan detail ruangan dan tindakan yang tersedia.

Jika kamera tidak tersedia atau izin ditolak, sistem menampilkan pesan yang jelas dan menyediakan input token manual sebagai alternatif.

**Acceptance Criteria:**

- QR yang valid menampilkan ruangan yang benar.
- QR tidak dikenal atau ruangan nonaktif ditolak.
- Kamera dapat digunakan pada perangkat yang kompatibel.
- Akses kamera pada deployment web harus menggunakan HTTPS, kecuali lingkungan localhost yang diizinkan browser.
- Token QR tidak boleh memberikan hak akses administrator atau mengizinkan checkout sesi orang lain.

### FR-06 — Reservation

Pengguna dapat membuat reservasi melalui formulir setelah memilih ruangan.

**Field formulir:**

- Ruangan: diambil dari hasil scan dan tidak dapat diubah secara bebas pada formulir.
- Nama mata kuliah: wajib.
- Total SKS: wajib, bilangan bulat sesuai rentang yang ditentukan.
- Waktu mulai: wajib.
- Waktu selesai: wajib.
- Identitas pemesan: berasal dari akun login.

Status reservasi:

- `CONFIRMED`
- `CANCELLED`
- `COMPLETED`

Status tambahan seperti `PENDING` hanya diperlukan jika proses persetujuan diterapkan.

**Business Rules:**

- Waktu selesai harus lebih besar daripada waktu mulai.
- Durasi reservasi tidak boleh melampaui batas yang dikonfigurasi.
- Reservasi tidak boleh bertabrakan dengan reservasi aktif lain pada ruangan yang sama.
- Pemeriksaan konflik dilakukan kembali di backend ketika permintaan disimpan.
- Operasi pembuatan reservasi harus aman terhadap request bersamaan.
- Reservasi harus terikat pada pengguna yang sedang login.
- Aturan waktu lampau, keterlambatan, dan pemesanan berulang harus ditentukan secara eksplisit.

**Acceptance Criteria:**

- Formulir valid menghasilkan reservasi yang tersimpan.
- Konflik waktu menghasilkan pesan kesalahan yang jelas.
- Pengguna tidak dapat memanipulasi `user_id` melalui request untuk membuat reservasi atas nama orang lain.
- Jika dua pengguna mencoba memesan slot yang sama secara bersamaan, maksimal satu reservasi yang bertentangan berhasil dikonfirmasi.

### FR-07 — My Reservations

Pengguna dapat melihat daftar reservasi miliknya.

Informasi yang ditampilkan:

- Ruangan.
- Nama mata kuliah.
- Total SKS.
- Waktu mulai dan selesai.
- Status reservasi.
- Status sesi penggunaan jika sudah dimulai.

Pengguna dapat membatalkan reservasi sesuai aturan pembatalan.

**Acceptance Criteria:**

- Pengguna hanya melihat reservasi miliknya pada halaman personal.
- Reservasi yang sedang memiliki sesi aktif tidak dapat dibatalkan melalui mekanisme pembatalan biasa.
- Pembatalan mengubah status reservasi tanpa menghapus riwayat.

### FR-08 — Check-in

Check-in mencatat bahwa pengguna mulai menggunakan ruangan yang telah dipesan.

Alur:

1. Pengguna memindai QR ruangan.
2. Backend mengidentifikasi ruangan.
3. Backend memeriksa reservasi pengguna dan rentang waktu yang diizinkan.
4. Pengguna mengonfirmasi check-in.
5. Backend membuat sesi penggunaan aktif.
6. Dashboard diperbarui.

**Business Rules:**

- Pengguna hanya dapat check-in ke reservasi yang memenuhi aturan waktu.
- Satu reservasi hanya boleh mempunyai satu sesi aktif.
- Pengguna tidak dapat membuat sesi aktif duplikat dengan mengirim request berulang.
- Keterlambatan dan toleransi check-in harus dikonfigurasi.

**Acceptance Criteria:**

- Check-in yang valid membuat sesi penggunaan.
- Check-in tanpa reservasi yang memenuhi syarat ditolak.
- Request duplikat tidak menghasilkan dua sesi aktif.
- Waktu check-in aktual dicatat oleh server.

### FR-09 — Checkout

Checkout mengakhiri sesi penggunaan yang aktif.

Alur:

1. Pengguna memindai QR ruangan.
2. Backend memeriksa sesi aktif pada ruangan tersebut.
3. Backend memastikan sesi tersebut milik pengguna atau pengguna mempunyai hak yang sesuai.
4. Pengguna menekan tombol Checkout.
5. Backend mencatat waktu selesai aktual.
6. Status sesi dan dashboard diperbarui.

**Business Rules:**

- QR ruangan saja tidak cukup untuk mengakhiri sesi.
- Hanya pemilik sesi atau role yang diizinkan yang dapat melakukan checkout.
- Checkout harus idempotent: request berulang tidak membuat waktu checkout berubah-ubah.
- Checkout tidak membuktikan bahwa ruangan secara fisik sudah kosong.

**Acceptance Criteria:**

- Checkout yang valid menutup sesi aktif.
- Checkout oleh pengguna yang tidak berhak ditolak.
- Checkout tanpa sesi aktif menampilkan pesan yang sesuai.
- Durasi aktual dapat dihitung dari waktu check-in dan checkout.

### FR-10 — Usage History

Sistem menyimpan riwayat reservasi dan penggunaan aktual.

Informasi minimal:

- Pemesan.
- Ruangan.
- Mata kuliah.
- SKS.
- Jadwal yang direncanakan.
- Waktu check-in aktual.
- Waktu checkout aktual.
- Status akhir.

Administrator dapat melihat riwayat lintas pengguna, sedangkan pengguna biasa hanya dapat melihat riwayatnya sendiri.

### FR-11 — Admin Operations

Administrator dapat:

- Mengelola data ruangan.
- Membuat atau menonaktifkan QR token.
- Melihat seluruh reservasi.
- Melihat sesi aktif.
- Menangani sesi yang tidak ditutup dengan benar.
- Melihat status perangkat jika integrasi IoT telah tersedia.

Tindakan administratif yang memengaruhi reservasi atau sesi harus dicatat untuk keperluan audit sederhana.

---

## 5. Non-Functional Requirements

### NFR-01 — Security

- Gunakan Laravel authentication dan authorization melalui middleware/policy.
- Gunakan CSRF protection untuk formulir web berbasis session.
- Validasi seluruh input pada backend.
- Gunakan hash password bawaan Laravel.
- Batasi endpoint admin berdasarkan role.
- Jangan menyimpan kredensial atau token rahasia dalam QR code.
- Terapkan rate limiting pada endpoint sensitif.
- Jangan mempercayai `user_id`, status reservasi, atau waktu aktual yang dikirim oleh frontend.

### NFR-02 — Data Integrity

- Gunakan foreign key dan indeks yang relevan.
- Gunakan database transaction untuk proses booking, check-in, dan checkout.
- Cegah reservasi yang saling bertabrakan.
- Cegah pembuatan sesi aktif ganda.
- Gunakan waktu server sebagai sumber waktu resmi.
- Tentukan timezone aplikasi, misalnya `Asia/Jakarta`, dan konsisten dalam penyimpanan serta tampilan.

### NFR-03 — Performance

- Dashboard harus mengambil ringkasan agregat dari backend, bukan menghitung semua status di browser.
- Hindari query N+1 pada daftar ruangan dan reservasi.
- Gunakan pagination untuk tabel riwayat.
- Perbarui dashboard melalui polling sederhana pada MVP; WebSocket tidak diwajibkan.

### NFR-04 — Usability

- Antarmuka harus responsif untuk smartphone dan desktop.
- Status ruangan tidak hanya dibedakan berdasarkan warna, tetapi juga teks.
- Formulir menampilkan pesan validasi yang mudah dipahami.
- Pengguna harus mendapat konfirmasi sebelum checkout.
- Hasil scan harus memberikan tindakan yang sesuai dengan status reservasi dan sesi.

### NFR-05 — Maintainability

- Pisahkan controller, model, validation request, policy, dan business logic.
- Gunakan migration dan seeder.
- Buat automated test untuk aturan reservasi dan otorisasi.
- Sediakan konfigurasi batas durasi dan toleransi waktu agar tidak tersebar sebagai magic number.

---

## 6. Data Model

Entitas utama MVP:

### User

- `id`
- `name`
- `email`
- `password`
- `role`
- `created_at`
- `updated_at`

### Room

- `id`
- `code`
- `name`
- `building`
- `floor`
- `capacity`
- `qr_token`
- `is_active`
- `created_at`
- `updated_at`

### Reservation

- `id`
- `user_id`
- `room_id`
- `course_name`
- `sks`
- `start_at`
- `end_at`
- `status`
- `created_at`
- `updated_at`

### RoomSession

- `id`
- `reservation_id`
- `room_id`
- `user_id`
- `checked_in_at`
- `checked_out_at`
- `status`
- `created_at`
- `updated_at`

### RoomDevice (untuk fondasi IoT)

- `id`
- `room_id`
- `device_identifier`
- `last_seen_at`
- `connection_status`
- `created_at`
- `updated_at`

### RoomTelemetry (untuk integrasi berikutnya)

- `id`
- `room_id`
- `device_id`
- `door_open`
- `motion_detected`
- `recorded_at`

`RoomDevice` dan `RoomTelemetry` dapat ditunda implementasinya jika MVP web belum terhubung dengan ESP32. Untuk tahap awal, status fisik dapat direpresentasikan sebagai `UNKNOWN`, bukan data sensor palsu.

---

## 7. Availability dan Status Rules

Sistem perlu menggunakan definisi status yang konsisten.

### Availability

Ketersediaan dihitung untuk rentang waktu tertentu, bukan sebagai properti permanen ruangan.

Ruangan dapat dipesan jika:

1. Ruangan aktif.
2. Tidak ada reservasi yang bertabrakan pada rentang waktu yang diminta.
3. Tidak ada aturan operasional yang melarang penggunaan.
4. Kebijakan sistem mengizinkan pemesanan pada waktu tersebut.

### Usage

- `NOT_STARTED`: belum ada sesi aktif.
- `ACTIVE`: sesi sedang berjalan.
- `COMPLETED`: sesi telah berakhir.
- `EXPIRED`: sesi ditutup oleh mekanisme kedaluwarsa atau tindakan administratif.

### Physical Monitoring

- `UNOCCUPIED_INDICATED`
- `OCCUPIED_INDICATED`
- `UNKNOWN`

Status fisik dari sensor merupakan indikasi, bukan bukti absolut bahwa ruangan kosong atau penuh.

---

## 8. QR Code Specification

Untuk MVP, QR code bersifat statis dan mengidentifikasi ruangan.

QR dapat berisi URL seperti:

`https://roomie.example/scan/<opaque-room-token>`

Aturan:

- Token harus unik dan sulit ditebak.
- QR tidak memuat password, data pribadi, atau kredensial akses.
- Backend memvalidasi token dan status ruangan.
- Token QR tidak memberikan hak istimewa.
- Token dapat dirotasi jika QR disalahgunakan.
- Hak booking, check-in, dan checkout tetap diperiksa melalui akun pengguna serta aturan bisnis.

Untuk deployment lokal, gunakan URL yang dapat dijangkau smartphone. URL `localhost` pada QR akan merujuk ke perangkat pemindai itu sendiri, bukan otomatis ke komputer pengembang.

---

## 9. User Stories dan Priority

### P0 — Wajib untuk MVP

1. Sebagai mahasiswa, saya dapat melihat dashboard ketersediaan ruangan.
2. Sebagai mahasiswa, saya dapat memindai QR dan melihat detail ruangan.
3. Sebagai mahasiswa, saya dapat membuat reservasi untuk rentang waktu tertentu.
4. Sebagai mahasiswa, saya tidak dapat membuat reservasi yang bertabrakan.
5. Sebagai mahasiswa, saya dapat melihat reservasi milik saya.
6. Sebagai mahasiswa, saya dapat check-in dan checkout secara sah.
7. Sebagai administrator, saya dapat mengelola ruangan.
8. Sebagai pengguna, saya dapat melihat status fisik sebagai tidak diketahui apabila data sensor belum tersedia.

### P1 — Setelah fungsi utama stabil

1. Riwayat penggunaan aktual.
2. Halaman admin untuk sesi yang kedaluwarsa.
3. Status online/offline ESP32.
4. Polling dashboard untuk pembaruan status.
5. Pencetakan dan rotasi QR token.
6. Audit log untuk tindakan administratif.

### P2 — Pengembangan lanjutan

1. Integrasi ESP32 secara langsung.
2. MQTT atau mekanisme event-driven.
3. QR dinamis.
4. Notifikasi reservasi.
5. Integrasi jadwal akademik.
6. Laporan statistik penggunaan ruang.

---

## 10. Definition of Done

MVP dianggap selesai apabila:

- Dashboard menampilkan data aktual dari database.
- Pengguna dapat melakukan booking dengan rentang waktu yang valid.
- Konflik reservasi dicegah oleh backend/database.
- QR ruangan yang valid dapat dipindai melalui browser.
- QR tidak valid ditolak.
- Pengguna dapat melihat reservasi miliknya.
- Check-in dan checkout mencatat waktu aktual.
- Pengguna tidak dapat checkout sesi milik orang lain.
- Administrator dapat mengelola ruangan.
- Status sensor yang belum tersedia tidak ditampilkan sebagai fakta.
- Pengujian utama untuk booking, konflik jadwal, dan otorisasi berhasil.
- Antarmuka berfungsi pada smartphone dan desktop.

---

## 11. Assumptions dan Open Questions

Sebelum implementasi dimulai, keputusan berikut perlu disepakati:

1. Apakah pengguna harus login sebelum scan QR atau boleh scan terlebih dahulu? Harus login, khusus PJ kelas
2. Apakah semua mahasiswa boleh melakukan reservasi, atau hanya pengguna tertentu? Hanya PJ
3. Apakah reservasi langsung dikonfirmasi atau memerlukan persetujuan? Langsung konfirmasi
4. Apakah satu reservasi berlaku untuk satu pengguna atau satu kelompok? Untuk satu kelas (PJ yang reservasi)
5. Berapa batas minimum dan maksimum durasi booking? (Tergantung jumlah SKS yang diinputkan setelah scan QR, 1 SKS = 50 menit)
6. Berapa toleransi keterlambatan check-in? (Perlu didiskusikan)
7. Apa yang terjadi jika pengguna lupa checkout? Ketika sensor IoT tidak mendeteksi adanya gerakan selama periode waktu tertentu, maka status akan diubah menjadi tersedia
8. Apakah kelas yang tidak memiliki reservasi boleh digunakan spontan? Harus reservasi terlebih dahulu (?)
9. Apakah status tersedia harus mempertimbangkan sensor fisik atau hanya jadwal? Pertimbangkan sensor yang sudah disepakati
10. Apakah integrasi ESP32 akan menggunakan HTTP atau MQTT? HTTP

**Rekomendasi MVP:** gunakan login, reservasi langsung tanpa persetujuan, waktu mulai dan selesai eksplisit, QR statis untuk identifikasi ruangan, sesi penggunaan terpisah dari reservasi, dan status fisik `UNKNOWN` sampai ESP32 benar-benar terintegrasi.
