# Aplikasi Web Firebase dengan ESP32 dan ESP8266

<div align="center">
  <img src="images/firebase-banner.png" alt="Firebase dengan ESP32/ESP8266" width="600">
</div>

## 🔥 Selamat Datang di Kursus Ini

Selamat datang di tutorial lengkap tentang membangun Aplikasi Web Firebase dengan mikrokontroler ESP32 dan ESP8266. Sepanjang kursus ini, Anda akan belajar cara membuat sistem IoT lengkap yang memungkinkan Anda:

- Mengendalikan GPIO ESP32/ESP8266 dari jarak jauh menggunakan buttons dan sliders
- Memantau pembacaan sensor secara real-time dari mana saja di dunia
- Mengimplementasikan authentication yang aman menggunakan email dan password
- Mengatur database rules untuk keamanan
- Mencatat data dengan timestamps dan menampilkan data historis pada charts

Semua data disimpan dalam Firebase Realtime Database, menciptakan aliran komunikasi yang mulus antara mikrokontroler dan web interface Anda.

<div align="center">
  <img src="images/project-demo.png" alt="Demo Proyek" width="600">
</div>

## 📚 Struktur Repository

Repository ini disusun dalam modul-modul yang mengikuti struktur yang sama dengan kursus asli:

```
/firebase-esp32-course/
├── README.md                   # Pengenalan kursus utama (Anda sedang berada di sini)
├── images/                     # Gambar yang digunakan dalam dokumentasi
├── resources/                  # Resource dan download tambahan
├── part-0-getting-started/     # Pengenalan Firebase dan gambaran umum proyek
├── part-1-firebase-project/    # Membuat dan mengkonfigurasi proyek Firebase
├── part-2-database-setup/      # Mengorganisir database dan menyiapkan rules
├── part-3-esp-integration/     # Interaksi ESP32/ESP8266 dengan Firebase
├── part-4-web-app-creation/    # Membangun web interface
├── part-5-hosting/             # Hosting aplikasi dengan custom domain
└── part-6-datalogging/         # Mengimplementasikan fungsi datalogging
```

Setiap modul berisi:
- README terperinci dengan instruksi langkah demi langkah
- Source code lengkap untuk modul tersebut
- Resource tambahan dan diagram jika diperlukan

## 🛠️ Apa yang Akan Anda Pelajari

Dengan menyelesaikan kursus ini, Anda akan mendapatkan pengalaman langsung dengan:

### Platform Firebase
- Membuat dan mengkonfigurasi proyek Firebase
- Menyiapkan authentication dengan email dan password
- Bekerja dengan Realtime Database
- Mengorganisir data secara efisien
- Mengimplementasikan database rules
- Hosting web apps
- Mengatur custom domain (opsional)

### Pemrograman ESP32/ESP8266
- Mengautentikasi board sebagai pengguna yang berwenang
- Menulis dan membaca data dari Realtime Database
- Streaming perubahan database secara real-time
- Menjalankan tasks berdasarkan perubahan database
- Logging data sensor dengan timestamps

### Pengembangan Web
- Membuat responsive web interface dengan HTML dan Bootstrap
- Mengimplementasikan authentication flows
- Real-time data synchronization dengan JavaScript
- Membangun interactive controls (buttons, sliders, inputs)
- Memvisualisasikan data dengan charts dan tables

## 🧩 Prerequisites

Untuk mendapatkan hasil maksimal dari kursus ini, pengetahuan berikut direkomendasikan (meskipun tidak wajib):

- Basic programming ESP32/ESP8266 dengan Arduino Core
- Familiaritas dengan VS Code dan PlatformIO
- Pemahaman dasar tentang HTML, CSS, dan JavaScript

Jika Anda belum memiliki keterampilan ini, jangan khawatir! Instruksinya cukup detail untuk pemula, meskipun memiliki background akan membantu Anda memahami konsep dengan lebih baik dan menyesuaikan proyek untuk kebutuhan Anda.

## 📋 Komponen yang Diperlukan

Untuk membangun proyek lengkap, Anda akan membutuhkan:

- Board ESP32 atau ESP8266
- Sensor BME280 atau sensor lain yang Anda kenal
- 4× LED
- 4× Resistor 220 Ohm
- Display OLED SSD1306 0,96 inci
- Breadboard
- Jumper wires

## 🚀 Memulai

Untuk mulai bekerja dengan repository ini:

1. Clone repository ini:
   ```
   git clone https://github.com/namapengguna/firebase-esp32-course.git
   ```

2. Navigasi ke modul pertama:
   ```
   cd firebase-esp32-course/part-0-getting-started
   ```

3. Ikuti instruksi README di setiap modul secara berurutan

## 💾 Mengunduh Resources

Semua resources untuk kursus ini tersedia dalam dua cara:

1. **Modul per Modul**: Setiap modul berisi resources-nya sendiri di direktori masing-masing

2. **Paket Lengkap**: Anda dapat mengunduh semua resources kursus sekaligus dari:
   [Download Complete Resources](https://github.com/namapengguna/firebase-esp32-course/archive/refs/heads/main.zip)

## ❓ Mendapatkan Bantuan

Jika Anda mengalami kesulitan:

1. Cobalah untuk menyelesaikan masalah dengan menganalisis error messages dan dokumentasi
2. Periksa tab [Issues](https://github.com/namapengguna/firebase-esp32-course/issues) untuk melihat apakah orang lain menghadapi masalah serupa
3. Buka issue baru jika Anda tidak dapat menemukan solusi untuk masalah Anda
4. Berpartisipasilah dalam [Discussions](https://github.com/namapengguna/firebase-esp32-course/discussions) untuk berinteraksi dengan pembelajar lain

## 📖 Gambaran Kursus Terperinci

Mari mulai dengan pengenalan Firebase dan gambaran umum tentang apa yang akan Anda bangun!

[Lanjut ke Part 0: Getting Started →](./part-0-getting-started/)

## 📜 License

Materi kursus ini dilisensikan di bawah MIT License - lihat file [LICENSE](LICENSE) untuk detailnya.

---

<div align="center">
  <p>Dibuat dengan ❤️ untuk para penggemar IoT di seluruh dunia</p>
</div>
