# Firebase Web App dengan ESP32 dan ESP8266

<div align="center">
  <img src="https://raw.githubusercontent.com/assets/firebase-esp-header.png" alt="Firebase dengan ESP32 dan ESP8266" width="600"/>
</div>

## 🚀 Memulai Perjalanan

Selamat datang di modul pembelajaran "**Firebase Web App dengan ESP32 dan ESP8266**". Materi ini akan memandu Anda langkah demi langkah untuk membangun aplikasi web berbasis Firebase yang dapat memantau dan mengendalikan perangkat ESP32 dan ESP8266 dari mana saja di dunia.

### 📱 Fitur Utama Aplikasi

Aplikasi web yang akan kita bangun memiliki kemampuan:

- Mengendalikan GPIO ESP menggunakan tombol dan slider
- Mengirim data input ke perangkat ESP
- Menampilkan pembacaan sensor dari perangkat ESP32/ESP8266
- Menyimpan semua data pada Firebase Realtime Database
- Dilengkapi sistem login menggunakan email dan password
- Database yang terlindungi dengan aturan keamanan (database rules)

## 💰 Apakah Firebase Gratis?

Ya, untuk aplikasi yang akan kita bangun, kita akan menggunakan "Spark Pricing Plan" yang gratis. Layanan Firebase yang akan kita gunakan untuk membangun dan meng-host aplikasi web tersedia dalam paket gratis.

## 📚 Cara Mengikuti Materi Pembelajaran

Materi ini disusun dengan pendekatan langkah-demi-langkah. Oleh karena itu, Anda harus mengikuti materi ini secara linier. Artinya, Anda perlu mengikuti semua langkah yang disebutkan dan sebaiknya tidak melewatkan bagian-bagian tertentu kecuali jika ada catatan khusus.

## 📦 Mengunduh Source Code dan Resources

Setiap bagian berisi kode dan resources yang Anda butuhkan untuk menyelesaikan setiap langkah dan melanjutkan ke bagian berikutnya. Anda dapat mengunduh semua resources untuk contoh tertentu pada Unit yang sesuai.

Alternatifnya, Anda dapat mengunduh seluruh repository proyek dan langsung mendapatkan semua resources:

- [Unduh Semua Resources Materi](https://github.com/YourUsername/Firebase-ESP32-ESP8266-Course/archive/refs/heads/main.zip)

## 📋 Gambaran Umum Materi

Materi pembelajaran ini dibagi menjadi enam bagian:

### Part 1: Membuat Proyek Firebase
- Mempelajari cara membuat proyek Firebase
- Menyiapkan layanan yang diperlukan (Authentication dan Firebase Realtime Database)

### Part 2: Mengorganisir Database dan Aturan Database
- Memahami cara mengorganisir database dengan efisien
- Mengatur aturan database untuk melindungi data Anda

### Part 3: ESP32/ESP8266: Berinteraksi dengan Realtime Database
- Berinteraksi dengan Firebase Realtime Database menggunakan board ESP32 dan ESP8266
- Mempelajari cara membaca dan mengirim data ke database

### Part 4: Membuat Aplikasi Web Firebase
- Menyiapkan file HTML dan JavaScript untuk membuat aplikasi web
- Mengimplementasikan kontrol dan pemantauan untuk board Anda

### Part 5: Hosting Aplikasi Web (Nama Domain Kustom)
- Meng-host aplikasi web Anda
- Mengatur nama domain kustom untuk mengakses aplikasi web dari mana saja

### Part 6: ESP32/ESP8266: Pencatatan Data ke Realtime Database
- Mempelajari cara mencatat data dengan timestamp
- Menampilkan riwayat data pada grafik dan tabel

## 🎓 Apa yang Akan Anda Pelajari

Dengan mengikuti proyek ini, Anda akan mempelajari hal-hal berikut:

### Firebase Project:
- Membuat proyek Firebase
- Menambahkan autentikasi ke proyek Firebase (email dan password)
- Menambahkan Realtime Database (RTDB) ke proyek untuk menyimpan data dalam format JSON
- Mengorganisir RTDB Anda
- Melindungi RTDB menggunakan aturan database
- Menambahkan aplikasi web ke proyek Firebase untuk mengendalikan dan memantau board ESP32 dan ESP8266
- Meng-host aplikasi web di server Firebase
- Menambahkan domain kustom ke aplikasi web — ini memerlukan pembelian nama domain (langkah ini opsional)

### ESP32/ESP8266:
- Mengautentikasi board ESP32 atau ESP8266 sebagai pengguna resmi dengan email dan password
- Menulis data ke Realtime Database
- Memantau database — mendeteksi perubahan database
- Menjalankan tugas berdasarkan perubahan yang terdeteksi
- Mencatat data dengan timestamp

### Aplikasi Web Firebase:
- Membuat aplikasi web dan menghubungkannya ke proyek Firebase
- Membangun halaman web untuk aplikasi menggunakan HTML dasar dan bootstrap (framework CSS): modal login/logout, tombol, slider, tabel untuk pembacaan sensor, grafik, dan kolom input
- Memantau perubahan database untuk memperbarui data aplikasi web menggunakan JavaScript
- Menulis ke database untuk mengendalikan output ESP32 atau ESP8266 menggunakan JavaScript
- Mengambil dan mengelola data untuk ditampilkan dalam berbagai format

## 🧩 Prasyarat

Untuk mendapatkan pengalaman terbaik dalam mengikuti proyek ini, kami merekomendasikan Anda memiliki pengetahuan dasar tentang:

### Pemrograman ESP32/ESP8266 dengan Arduino Core
Memahami dasar-dasar seperti mengendalikan output dan membaca sensor. Untuk membiasakan diri dengan board ini, Anda dapat mengikuti tutorial gratis atau eBook premium kami:
- [Tutorial ESP32 Gratis (Cooming Soon)](https://yourwebsite.com/free-esp32-tutorials)
- [Tutorial ESP8266 Gratis (Cooming Soon)](https://yourwebsite.com/free-esp8266-tutorials)
- [Belajar ESP32 dengan Arduino IDE (Cooming Soon)](https://yourwebsite.com/learn-esp32-arduino-ide)
- [Home Automation menggunakan ESP8266 (Cooming Soon)](https://yourwebsite.com/home-automation-esp8266)

### Familiar dengan VS Code dan PlatformIO
Memahami penggunaan editor VS Code dan ekstensi PlatformIO. Tutorial berikut dapat membantu Anda:
- [Memulai dengan VS Code dan PlatformIO IDE untuk ESP32 dan ESP8266 (Cooming Soon)](https://yourwebsite.com/vscode-platformio-getting-started)
- [VS Code Workspaces dengan Proyek ESP32 dan ESP8266 (Cooming Soon)](https://yourwebsite.com/vscode-workspaces-esp)

### Pengetahuan Dasar HTML, CSS, dan JavaScript
Kami berasumsi Anda memiliki pengetahuan dasar tentang HTML (tag HTML dasar), CSS, dan JavaScript (variabel, fungsi, dan event). Situs web [W3Schools](https://www.w3schools.com/) adalah tempat yang baik untuk memulai dengan cepat.

> 💡 **Catatan**: Jika Anda tidak memiliki pengetahuan sebelumnya tentang topik-topik ini, Anda tetap dapat mengikuti panduan langkah demi langkah. Namun, mungkin akan lebih menantang untuk memahami beberapa langkah dan memodifikasi proyek sesuai kebutuhan spesifik Anda. Meskipun demikian, kami yakin Anda akan dapat membuat aplikasi berfungsi.

## 🛠️ Komponen yang Dibutuhkan

Untuk mengikuti proyek ini, Anda hanya membutuhkan beberapa komponen elektronik:

<div align="center">
  <img src="https://raw.githubusercontent.com/assets/components-setup.jpg" alt="Komponen yang Dibutuhkan" width="600"/>
</div>

- [ESP32](https://yourwebsite.com/esp32) atau [ESP8266](https://yourwebsite.com/esp8266)
- [BME280](https://yourwebsite.com/bme280) atau sensor lain yang Anda kenal
- 4x [LED](https://yourwebsite.com/leds)
- 4x [Resistor 220 Ohm](https://yourwebsite.com/resistors)
- [Display OLED SSD1306 0.96 inch](https://yourwebsite.com/ssd1306)
- [Breadboard](https://yourwebsite.com/breadboard)
- [Kabel jumper](https://yourwebsite.com/jumper-wires)

## 🔍 Dukungan dan Umpan Balik

Selama mengikuti materi, Anda mungkin akan menghadapi kesulitan atau masalah teknis. Kami sangat mendorong Anda untuk mencoba memperbaiki masalah teknis sendiri terlebih dahulu. Memperbaiki masalah teknis sendiri adalah cara yang sangat baik untuk mempelajari subjek baru.

Jika Anda sudah berusaha semaksimal mungkin tetapi tidak bisa menemukan solusi untuk masalah Anda, atau jika Anda ingin memberikan umpan balik tentang materi pembelajaran ini, Anda dapat bergabung dengan komunitas kami di:

- [Grup Telegram Koding Indonesia](https://t.me/kodingindonesia)

Di grup Telegram ini, Anda dapat:
- Mengajukan pertanyaan tentang masalah yang Anda hadapi
- Berbagi pengalaman Anda dalam mengikuti proyek
- Memberikan umpan balik untuk perbaikan materi
- Berinteraksi dengan peserta lain yang sedang mengerjakan proyek serupa

Umpan balik Anda sangat berharga bagi kami untuk terus meningkatkan kualitas materi pembelajaran. Saran, koreksi, dan pendapat Anda akan membantu kami membuat materi yang lebih baik untuk semua orang.

---

<div align="center">
  <p>Dibuat dengan ❤️ oleh <a href="https://github.com/YourUsername">Your Name</a></p>
  <p>Referensi: Firebase Web App with ESP32 and ESP8266 oleh Rui Santos dan Sara Santos</p>
</div>
