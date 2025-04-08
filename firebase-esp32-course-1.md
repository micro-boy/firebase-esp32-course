# 🔥 Part 1: Membuat Proyek Firebase

<div align="center">
  <img src="../images/banners/part1-banner.png" alt="Firebase Project Setup" width="600">
</div>

Pada bagian ini, Anda akan belajar cara membuat proyek Firebase, menyiapkan metode autentikasi, dan membuat Realtime Database. Langkah-langkah ini merupakan fondasi penting untuk seluruh proyek Anda. Anda akan mengonfigurasi semua layanan yang diperlukan untuk membangun aplikasi web yang dapat berinteraksi dengan perangkat ESP32 dan ESP8266 Anda.

> ⚠️ **Penting**: Pastikan Anda mengikuti semua langkah yang dijelaskan dan tidak melewatkan instruksi apa pun.

## 📋 Gambaran Bagian Ini

Pada bagian ini, kita akan melakukan beberapa langkah penting:

1. Membuat proyek Firebase baru
2. Mengatur metode autentikasi dengan email dan password
3. Membuat Realtime Database
4. Mendapatkan API Key proyek

Mari kita mulai!

## 1️⃣ Membuat Proyek Baru

Langkah pertama adalah membuat proyek Firebase baru yang akan menjadi dasar untuk aplikasi IoT kita.

### Langkah-langkah:

1. Kunjungi [Firebase](https://firebase.google.com), masuk menggunakan Akun Google, dan klik **Get Started**.

   <div align="center">
     <img src="../images/screenshots/part1/firebase-homepage.png" alt="Firebase Homepage" width="700">
   </div>

2. Klik **Add project** untuk membuat proyek baru.

   <div align="center">
     <img src="../images/screenshots/part1/add-project.png" alt="Add Project" width="700">
   </div>

3. Beri nama proyek Anda, misalnya "ESP IoT App", dan tekan **Continue**.

   <div align="center">
     <img src="../images/screenshots/part1/project-name.png" alt="Project Name" width="700">
   </div>

4. Nonaktifkan opsi **Enable Google Analytics for this project** karena tidak diperlukan, dan klik **Create project**.

   <div align="center">
     <img src="../images/screenshots/part1/disable-analytics.png" alt="Disable Analytics" width="700">
   </div>

5. Tunggu beberapa saat sampai proyek Anda disiapkan. Klik **Continue** ketika sudah siap.

   <div align="center">
     <img src="../images/screenshots/part1/project-ready.png" alt="Project Ready" width="700">
   </div>

6. Anda akan diarahkan ke halaman konsol Proyek Anda.

   <div align="center">
     <img src="../images/screenshots/part1/project-console.png" alt="Project Console" width="700">
   </div>

Proyek Anda sekarang telah dibuat! Mari mengaktifkan dan mengonfigurasi layanan yang diperlukan.

## 2️⃣ Mengatur Metode Autentikasi

Anda perlu menyiapkan metode autentikasi untuk aplikasi Anda. Autentikasi sangat penting karena:

- Memungkinkan aplikasi mengetahui identitas pengguna
- Memungkinkan penyimpanan data pengguna dengan aman di cloud
- Memberikan pengalaman yang sama di semua perangkat pengguna
- Menangani proses login dan identifikasi pengguna (termasuk ESP32/ESP8266 untuk mengakses database)
- Memungkinkan kita menyiapkan aturan database berdasarkan pengguna yang masuk

### Langkah-langkah:

1. Pada sidebar kiri, klik **Authentication** dan kemudian klik **Get started**.

   <div align="center">
     <img src="../images/screenshots/part1/auth-getstarted.png" alt="Authentication Get Started" width="700">
   </div>

2. Pilih opsi **Email/Password**.

   <div align="center">
     <img src="../images/screenshots/part1/auth-email-password.png" alt="Email/Password Option" width="700">
   </div>

3. Aktifkan opsi untuk memungkinkan pengguna mendaftar menggunakan alamat email dan password mereka. Kemudian, klik **Save**.

   <div align="center">
     <img src="../images/screenshots/part1/enable-email-auth.png" alt="Enable Email Authentication" width="700">
   </div>

4. Sekarang Anda perlu menambahkan pengguna yang diotorisasi. Aplikasi kita hanya akan memiliki satu pengguna yang diotorisasi. Masih di tab **Authentication**, klik tab **Users** di bagian atas. Kemudian, klik **Add user**.

   <div align="center">
     <img src="../images/screenshots/part1/add-user.png" alt="Add User" width="700">
   </div>

5. Tambahkan alamat email untuk pengguna yang diotorisasi. Ini bisa berupa akun Google Anda atau email lainnya. Anda juga bisa membuat email khusus untuk proyek ini. Tambahkan password yang akan memungkinkan Anda masuk ke aplikasi dan mengakses database. Jangan lupa menyimpan password di tempat yang aman karena Anda akan membutuhkannya nanti. Setelah selesai, klik **Add user**.

   <div align="center">
     <img src="../images/screenshots/part1/create-user.png" alt="Create User" width="700">
   </div>

6. Pengguna baru berhasil dibuat dan ditambahkan ke tabel **Users**.

   <div align="center">
     <img src="../images/screenshots/part1/user-created.png" alt="User Created" width="700">
   </div>

> **Catatan**: Firebase membuat UID unik untuk setiap pengguna terdaftar. UID pengguna memungkinkan kita mengidentifikasi pengguna dan melacak pengguna untuk memberikan atau menolak akses ke database. Simpan UID ini karena akan dibutuhkan nanti.

## 3️⃣ Membuat Realtime Database

Firebase menyediakan dua opsi database NoSQL yang berbeda: Realtime Database dan Firestore Database.

- **Realtime Database** menyimpan data dalam format JSON
- **Firestore Database** menyimpan data dalam dokumen yang diorganisir ke dalam koleksi

Dalam proyek kita, kita akan menggunakan **Realtime Database (RTDB)**. Ikuti instruksi berikut untuk menambahkan Realtime Database ke proyek Anda.

### Langkah-langkah:

1. Pada sidebar kiri, klik **Realtime Database** dan kemudian klik **Create Database**.

   <div align="center">
     <img src="../images/screenshots/part1/create-rtdb.png" alt="Create Realtime Database" width="700">
   </div>

2. Pilih lokasi database Anda. Sebaiknya pilih yang terdekat dengan lokasi Anda. Kemudian, klik **Next**.

   <div align="center">
     <img src="../images/screenshots/part1/select-location.png" alt="Select Database Location" width="700">
   </div>

3. Anda akan diminta untuk menyiapkan aturan keamanan untuk database Anda. Pilih **Start in test mode**. Dengan aturan database ini, siapa pun dengan referensi database Anda dapat melihat, mengedit, dan menghapus semua data selama 30 hari ke depan. Namun jangan khawatir. Di bagian berikutnya, kita akan menyiapkan aturan database untuk melindungi data Anda. Klik **Enable** untuk menyelesaikan penyiapan Realtime Database.

   <div align="center">
     <img src="../images/screenshots/part1/test-mode.png" alt="Start in Test Mode" width="700">
   </div>

4. Database Anda sekarang telah dibuat. Database ini diberi URL unik, seperti yang disorot pada gambar berikut. Anda akan membutuhkan URL database Anda nanti agar ESP32 atau ESP8266 dapat berinteraksi dengan database.

   <div align="center">
     <img src="../images/screenshots/part1/database-url.png" alt="Database URL" width="700">
   </div>

> **Tip**: Anda dapat membuat file .txt di komputer Anda untuk menyimpan semua pengaturan seperti UID pengguna, URL database, dan kunci API proyek. Atau, Anda selalu dapat kembali ke konsol Firebase untuk mendapatkannya.

## 4️⃣ Mendapatkan API Key Proyek

Untuk berinteraksi dengan proyek Firebase Anda menggunakan board ESP32 atau ESP8266, Anda perlu mendapatkan kunci API proyek Anda. Ikuti langkah-langkah berikut untuk mendapatkan kunci API proyek Anda.

### Langkah-langkah:

1. Pada sidebar kiri, klik **Project Settings**.

   <div align="center">
     <img src="../images/screenshots/part1/project-settings.png" alt="Project Settings" width="700">
   </div>

2. Salin **Web API Key** ke tempat yang aman karena Anda akan membutuhkannya nanti.

   <div align="center">
     <img src="../images/screenshots/part1/web-api-key.png" alt="Web API Key" width="700">
   </div>

## 🎉 Selamat!

Proyek Firebase Anda sekarang sudah siap! Kita telah berhasil:

- ✅ Membuat proyek Firebase baru
- ✅ Mengatur autentikasi dengan email dan password
- ✅ Membuat pengguna autentikasi
- ✅ Membuat Realtime Database
- ✅ Mendapatkan API Key proyek

## 📝 Catatan Penting

Pastikan Anda telah menyimpan informasi penting berikut di tempat yang aman:

1. **Email dan Password** yang Anda gunakan untuk autentikasi
2. **UID Pengguna** yang dihasilkan Firebase
3. **URL Database** Realtime Database Anda
4. **Web API Key** proyek Anda

Semua informasi ini akan diperlukan di bagian-bagian selanjutnya saat kita menghubungkan ESP32/ESP8266 ke Firebase dan membuat aplikasi web.

## 🔜 Langkah Selanjutnya

Di bagian berikutnya, kita akan melihat cara mengatur struktur database dan melindungi data Anda menggunakan aturan database.

[➡️ Lanjut ke Part 2: Mengorganisasi Database dan Aturan Database](https://github.com/micro-boy/firebase-esp32-course/blob/main/firebase-esp32-course-2.md)

---

<div align="Left">
  <p>Dibuat dengan ❤️ oleh <a href="https://github.com/micro-boy">Micro Boy</a></p>
  <p>Referensi: Firebase Web App with ESP32 and ESP8266 oleh Rui Santos dan Sara Santos</p>
</div>
