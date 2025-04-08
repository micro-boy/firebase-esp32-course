# 🔥 Pengenalan Firebase dan Gambaran Proyek

<div align="center">
  <img src="../images/banners/firebase-logo.png" alt="Firebase Logo" width="300">
</div>

## Apa Itu Firebase?

Firebase adalah platform pengembangan aplikasi mobile besutan Google yang membantu Anda membangun, meningkatkan, dan mengembangkan aplikasi. Platform ini menyediakan banyak layanan untuk mengelola data dari aplikasi Android, iOS, atau web.

Kutipan berikut menjelaskan dengan sangat baik keunggulan menggunakan Firebase:

> "Firebase adalah kumpulan alat untuk 'membangun, meningkatkan, dan mengembangkan aplikasi Anda', dan alat yang disediakan mencakup sebagian besar layanan yang biasanya harus dibangun sendiri oleh developer tetapi mereka tidak benar-benar ingin membangunnya karena lebih memilih fokus pada pengalaman aplikasi itu sendiri. Ini mencakup hal-hal seperti analitik, autentikasi, database, konfigurasi, penyimpanan file, push messaging, dan masih banyak lagi. Layanan-layanan ini di-hosting di cloud dan dapat diskalakan dengan sedikit atau tanpa usaha dari pihak developer."[^1]

[^1]: Sumber: [What is Firebase? The Complete Story, Abridged](https://medium.com/firebase-developers/what-is-firebase-the-complete-story-abridged-bcc730c5f2c0)

## 🧰 Produk Firebase untuk Membangun Aplikasi

Firebase adalah seperangkat alat untuk membangun, meningkatkan, dan mengembangkan aplikasi Anda. Dalam kursus ini, kita akan fokus pada proses pembangunan karena kita akan membuat aplikasi web untuk penggunaan pribadi. Jika Anda ingin membuat aplikasi web untuk penggunaan komersial, Firebase menawarkan alat seperti analitik, prediksi, pengujian A/B, pemantauan kinerja, dll., untuk meningkatkan dan mengembangkan aplikasi Anda.

Jadi, mari fokus pada proses pembangunan. Firebase menyediakan produk-produk berikut untuk membangun aplikasi Anda:

### Produk Utama Firebase

| Layanan | Deskripsi |
|---------|-----------|
| **Authentication** | Login/logout dan identitas pengguna |
| **Realtime Database** | Database NoSQL real-time berbasis cloud; data disimpan dalam struktur JSON |
| **Cloud Firestore** | Database NoSQL real-time berbasis cloud; data disimpan dalam "dokumen" |
| **Cloud Storage** | Penyimpanan file yang dapat diskalakan untuk mengupload dan mengunduh file |
| **Cloud Functions** | Backend serverless berbasis event |
| **Firebase Hosting** | Web hosting global dengan atau tanpa domain kustom |
| **ML Kit** | SDK untuk tugas-tugas Machine Learning umum |

Sepanjang proyek ini, kita akan menggunakan produk-produk berikut: **Authentication**, **Realtime Database**, **Cloud Functions**, dan **Firebase Hosting**.

## 📱 Gambaran Proyek Firebase

Dalam kursus ini, Anda akan membangun aplikasi web dengan Firebase untuk berinteraksi dengan board ESP32 dan/atau ESP8266 Anda—mengontrol output dan menampilkan pembacaan sensor dari mana saja. Firebase memudahkan pembuatan aplikasi web untuk berinteraksi dengan board ESP32 dan ESP8266 Anda dari mana saja di dunia. Firebase memungkinkan Anda untuk menyimpan semua data di satu tempat dan menyinkronkannya di semua perangkat Anda.

<div align="center">
  <img src="../images/screenshots/firebase-web-app-preview.png" alt="Firebase Web App Preview" width="700">
  <br>
  <em>Tampilan aplikasi web Firebase yang akan kita bangun</em>
</div>

Mari kita lihat gambaran cepat tentang proyek untuk memahami semua fitur proyek:

### Fitur-fitur Aplikasi Web

- ✅ **Autentikasi Aman**: Aplikasi web Firebase dilindungi dengan login menggunakan email dan password. Hanya pengguna yang berwenang yang dapat mengakses data. Selain itu, database Anda dilindungi oleh aturan database.

- ✅ **Kontrol GPIO**: Aplikasi web menampilkan dua tombol untuk mengontrol dua GPIO ON dan OFF. Status GPIO disimpan dan diperbarui di Realtime Database. ESP32 atau ESP8266 mendeteksi perubahan database dan memperbarui output sesuai.

- ✅ **Kontrol PWM**: Dua slider mengontrol dua output dengan PWM (LED, motor, atau periferal lain yang memerlukan PWM). Nilai slider disimpan dan diperbarui di Realtime Database. ESP32 atau ESP8266 mendeteksi perubahan database dan memperbarui duty cycle PWM output sesuai.

- ✅ **Tampilan Sensor**: Tabel menampilkan pembacaan sensor. Aplikasi web mendapatkan pembacaan baru dari node database yang dipublikasikan oleh ESP32. Kita akan menyimpan pembacaan sensor dari sensor BME280.

- ✅ **Pesan OLED**: Ada juga field input untuk memasukkan pesan yang akan ditampilkan di OLED. Pesan ini juga disimpan di Realtime Database.

- ✅ **Akses Global**: Anda dapat mengakses aplikasi web Firebase Anda dari mana saja di dunia untuk berinteraksi dengan ESP32/ESP8266 Anda menggunakan domain kustom atau nama domain default yang disediakan oleh Firebase.

- ✅ **Responsif**: Aplikasi web memiliki desain yang responsif untuk perangkat mobile.

### Fitur-fitur ESP32/ESP8266

- 🔐 **Autentikasi**: ESP32/ESP8266 mengautentikasi dengan username dan password. Username dan password tersebut harus milik pengguna yang berwenang—jika tidak, tidak akan dapat berinteraksi dengan Realtime Database.

- 📊 **Publikasi Data Sensor**: Board mempublikasikan pembacaan sensor BME280—suhu, kelembaban, dan tekanan—secara berkala ke Realtime Database.

- 👂 **Mendengarkan Perubahan**: ESP32/ESP8266 mendengarkan perubahan pada status GPIO, nilai slider, dan nilai pesan.

- 🔄 **Update Otomatis**: Setiap kali ada perubahan, board memperbarui status GPIO, nilai PWM, atau pesan yang ditampilkan di OLED.

- 🔄 **Status Persistensi**: Ketika ESP reboot atau mulai, secara otomatis mengumpulkan semua informasi dari database dan memperbarui GPIO dan tampilan OLED sesuai.

### Fitur Datalogging (Part 6)

Kami menambahkan bagian tambahan, Part 6, di mana kami menambahkan fitur datalogging dengan timestamps sehingga Anda dapat mengakses riwayat data Anda dalam tabel dan grafik. Di bagian itu, Anda akan belajar:

- 📝 Cara mencatat data dengan timestamps menggunakan ESP32/ESP8266 ke Firebase Realtime Database.
- 🔍 Query dan mengambil data untuk menampilkannya di grafik dan tabel.

<div align="center">
  <img src="../images/screenshots/datalogging-preview.png" alt="Datalogging Preview" width="700">
  <br>
  <em>Fitur datalogging dengan timestamps, grafik, dan tabel</em>
</div>

## 📚 Bacaan yang Direkomendasikan

Sebelum memulai, Anda mungkin tertarik untuk membaca beberapa referensi berikut:

- [Firebase Terms](https://firebase.google.com/terms/)
- [Firebase Billing](https://firebase.google.com/docs/projects/billing/firebase-pricing-plans)

Untuk aplikasi kita, kita akan menggunakan **Spark Pricing Plan** yang gratis.

> "Ketika Anda berada dalam tahap awal pengembangan aplikasi, mulailah dengan paket harga Spark. Anda tidak perlu memberikan informasi pembayaran apa pun untuk mulai menggunakan sebagian besar fitur Firebase dengan segera!"[^2]

[^2]: Sumber: [Firebase Pricing Plans](https://firebase.google.com/docs/projects/billing/firebase-pricing-plans#spark-pricing-plan)

---

Sekarang setelah Anda mengetahui tentang apa yang akan dibahas dalam proyek ini, silakan lanjutkan ke unit berikutnya untuk memulai.

[➡️ Lanjut ke Part 1: Membuat Proyek Firebase](../part1-firebase-project/README.md)
