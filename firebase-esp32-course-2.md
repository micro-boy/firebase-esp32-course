# 🔥 Part 2: Mengorganisasi Database dan Aturan Database

<div align="center">
  <img src="../images/banners/part2-banner.png" alt="Firebase Database Organization" width="600">
</div>

Pada bagian ini, Anda akan belajar cara mengorganisasi data dalam Realtime Database untuk memudahkan pengaturan aturan database. Anda akan membuat node database untuk menyimpan data dan menerapkan aturan database untuk membatasi akses. Kami juga akan membagikan aturan database yang berguna yang mungkin ingin Anda gunakan dalam proyek-proyek mendatang.

## 📚 Apa yang Akan Anda Pelajari

Dalam bagian ini, kita akan membahas:

1. Memahami struktur Realtime Database Firebase
2. Mengorganisasi data dalam format JSON
3. Membuat node database untuk proyek IoT kita
4. Menerapkan aturan keamanan database
5. Contoh aturan database lainnya untuk referensi masa depan

Mari kita mulai!

## 2.1 - Mengorganisasi Database Anda

Untuk lebih memahami apa itu Firebase Realtime Database, berikut adalah kutipan dari dokumentasi Firebase:

> "Firebase Realtime Database adalah database yang di-hosting di cloud. Data disimpan sebagai JSON dan disinkronkan secara realtime ke setiap klien yang terhubung. Ketika Anda membangun aplikasi lintas platform dengan SDK iOS, Android, dan JavaScript kami, semua klien Anda berbagi satu instance Realtime Database dan secara otomatis menerima pembaruan dengan data terbaru."

Semua data yang disimpan dalam Firebase Realtime Database disimpan sebagai objek JSON. Jadi, Anda dapat menganggap database sebagai struktur JSON berbasis cloud. Ketika Anda menambahkan data ke struktur JSON, data tersebut menjadi sebuah node dengan kunci terkait dalam struktur JSON yang ada.

### Pengenalan Format JSON

Tidak familiar dengan JSON? JSON (JavaScript Object Notation) adalah format pertukaran data ringan yang mudah dibaca dan ditulis oleh manusia, serta mudah dianalisis dan dibuat oleh mesin. JSON dibangun di atas dua struktur:

- Kumpulan pasangan nama/nilai
- Daftar nilai yang berurutan

Dalam JSON, data diwakili sebagai pasangan kunci-nilai, di mana kunci adalah string dan nilai bisa berupa string, angka, objek, array, boolean, atau null.

### Merencanakan Struktur Database

Cara terbaik untuk mengorganisasi data Anda akan bergantung pada fitur proyek Anda dan bagaimana pengguna mengakses data.

Mari kita tinjau proyek kita untuk memahami bagaimana kita harus mengelola data:

- Hanya ada satu pengguna yang dapat mengakses data—pengguna yang Anda buat di bagian sebelumnya: Mengatur Metode Autentikasi;
- Kita perlu menyimpan data berikut dalam database:
  - Pembacaan sensor: suhu, kelembaban, dan tekanan;
  - Dua GPIO dan status masing-masing: 0 (mati) atau 1 (hidup)—kita akan menggunakan GPIO 2 dan GPIO 12;
  - Dua GPIO dan nilai duty cycle PWM masing-masing: nilai antara 0 dan 100—kita akan menggunakan GPIO 13 dan 14;
  - Pesan yang akan ditampilkan pada layar OLED.

Dengan mempertimbangkan data yang ingin kita simpan dan bahwa kita ingin membatasi akses hanya pada satu pengguna yang berwenang, berikut adalah bagaimana kita akan menyusun database:

```
• UsersData:
  ○ <user_uid>*:
    ▪ outputs:
      • digital:
        ○ 2: 0
        ○ 12: 1
      • pwm: 
        ○ 13: 20
        ○ 14: 80
      • message: "I love this app"
    ▪ sensor:
      • temperature: 25.00
      • humidity: 52.50
      • pressure: 1006.13
```

*\*Ini adalah UID Pengguna unik dari pengguna yang masuk dan diberi otorisasi.*

Dalam format JSON, beginilah tampilannya:

```json
{
 "UsersData" : {
   "JXhbigMqPsOGEg9T99WXXXXXXXX" : {
     "outputs" : {
       "digital" : {
         "2" : 0,
         "12" : 1
       },
       "message" : "I love this app",
       "pwm" : {
         "13" : 20,
         "14" : 80
       }
     },
     "sensor" : {
       "humidity" : 52.50,
       "pressure" : 1006.13,
       "temperature" : 25.00
     }
   }
 }
}
```

Struktur ini memungkinkan:
- Organisasi data yang jelas dan terstruktur
- Pemisahan data berdasarkan fungsinya (outputs vs sensor)
- Akses berbasis pengguna yang aman melalui node UID
- Kemudahan dalam menerapkan aturan database (yang akan kita lihat nanti)

### Membuat Node Database

Sekarang mari kita buat node database dalam database kita. Anda dapat membuat node secara manual dengan menulis node di konsol Firebase, di aplikasi web, atau melalui ESP32/ESP8266. Kita akan membuatnya secara manual agar lebih mudah diikuti.

Kita akan menyimpan semua data pengguna di dalam sebuah node dengan UID pengguna yang berwenang. Jadi, dapatkan UID pengguna Anda dengan mengikuti langkah-langkah berikut:

#### Langkah-langkah:

1. Di konsol Firebase Anda, pilih Proyek Anda, buka tab **Authentication**, pilih **Users**, dan salin UID pengguna dari tabel.

   <div align="center">
     <img src="../images/screenshots/part2/get-user-uid.png" alt="Get User UID" width="700">
   </div>

2. Klik pada **Realtime Database** agar kita mulai membuat node.

   <div align="center">
     <img src="../images/screenshots/part2/open-rtdb.png" alt="Open Realtime Database" width="700">
   </div>

3. Anda dapat membuat node database secara manual dengan menggunakan ikon (+) pada database. Namun, untuk mencegah kesalahan pengetikan, kami menyediakan file JSON yang dapat Anda unggah untuk membuat node yang sama dengan milik kami.

   > **Catatan**: Pastikan Anda mengunduh file JSON dari repositori kursus ini. File tersebut akan berisi template yang akan kita gunakan.

4. Buka file JSON yang diunduh. Anda dapat menggunakan editor teks apa pun seperti Notepad atau VS Code. Ganti teks `REPLACE_WITH_YOUR_USER_UID` dengan UID pengguna yang Anda dapatkan dari langkah nomor 1. Kemudian, simpan file JSON Anda.

   <div align="center">
     <img src="../images/screenshots/part2/edit-json.png" alt="Edit JSON File" width="700">
   </div>

   File JSON Anda akan terlihat seperti berikut tetapi dengan UID pengguna yang berwenang milik Anda:

   <div align="center">
     <img src="../images/screenshots/part2/json-with-uid.png" alt="JSON with UID" width="700">
   </div>

5. Sekarang, kembali ke database Anda di konsol Firebase. Klik ikon tiga titik dan pilih **Import JSON**.

   <div align="center">
     <img src="../images/screenshots/part2/import-json.png" alt="Import JSON" width="700">
   </div>

6. Pilih file JSON yang baru saja Anda edit.

7. Database Anda akan terlihat seperti gambar di bawah ini tetapi dengan UID unik Anda. Anda dapat memperluas node untuk melihat semua data.

   <div align="center">
     <img src="../images/screenshots/part2/database-structure.png" alt="Database Structure" width="700">
   </div>

Semua node database untuk proyek ini telah dibuat. Nilai untuk setiap node hanyalah nilai sembarang agar kita dapat menguji database. Nanti Anda akan memasukkan data aktual ke dalam node.

## 2.2 - Menyiapkan Aturan Database

Saat ini, siapa pun dengan referensi ke database Anda dapat membaca, menulis, dan menghapus data. Anda tidak ingin ini terjadi. Anda hanya ingin pengguna yang berwenang memiliki akses ke data.

### Langkah-langkah:

1. Di konsol Firebase Anda, buka **Realtime Database** dan pilih tab **Rules**.

   <div align="center">
     <img src="../images/screenshots/part2/database-rules-tab.png" alt="Database Rules Tab" width="700">
   </div>

   Saat ini, aturan database Anda harus mirip dengan milik saya tetapi dengan tanggal jatuh tempo yang berbeda. Anda ingin melindungi data sehingga hanya pengguna yang berwenang yang dapat membaca dan menulis ke database di bawah node dengan UID-nya.

2. Untuk itu, salin aturan berikut ke aturan database Anda:

   ```json
   // Aturan ini memberikan akses ke node yang cocok dengan ID pengguna terotentikasi
   // dari token otentikasi Firebase
   {
     "rules": {
       "UsersData": {
         "$uid": {
           ".read": "$uid === auth.uid",
           ".write": "$uid === auth.uid"
         }
       }
     }
   }
   ```

   Aturan ini menentukan bahwa:
   - Hanya pengguna dengan UID yang cocok dengan node `$uid` yang dapat membaca data dari node tersebut
   - Hanya pengguna dengan UID yang cocok dengan node `$uid` yang dapat menulis data ke node tersebut

   <div align="center">
     <img src="../images/screenshots/part2/edit-rules.png" alt="Edit Database Rules" width="700">
   </div>

3. Kemudian, klik **Publish**.

   <div align="center">
     <img src="../images/screenshots/part2/publish-rules.png" alt="Publish Rules" width="700">
   </div>

Dan selesai! Database dan aturan database Anda sudah siap! Lanjutkan ke bagian berikutnya untuk mulai berinteraksi dengan database menggunakan board ESP32 atau ESP8266.

## 🛠️ Aturan Database Lain yang Berguna

Anda dapat mengubah aturan database kapan saja nanti. Berikut adalah beberapa aturan database yang mungkin berguna untuk proyek di masa depan atau jika Anda ingin mengubah proyek ini.

### Tanpa Keamanan

Siapa pun dapat membaca atau menulis ke database Anda. Ini berguna untuk tujuan pengujian.

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

### Keamanan Penuh

Tidak ada yang dapat mengakses database. Anda hanya dapat membaca/menulis menggunakan konsol Firebase.

```json
{
  "rules": {
    ".read": false,
    ".write": false
  }
}
```

### Hanya Pengguna Terotentikasi yang Dapat Mengakses Data

Tidak masalah siapa penggunanya. Selama terotentikasi, ia dapat mengakses semua node database.

```json
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null"
  }
}
```

## 📚 Bacaan Lebih Lanjut

Untuk mempelajari lebih lanjut tentang aturan database, periksa [dokumentasi Firebase](https://firebase.google.com/docs/database/security).

## 🎉 Apa yang Sudah Kita Pelajari

Pada bagian ini, kita telah:

- ✅ Memahami struktur data JSON dalam Firebase Realtime Database
- ✅ Merancang struktur database yang terorganisir untuk proyek IoT kita
- ✅ Membuat node database untuk menyimpan data sensor dan kontrol
- ✅ Menerapkan aturan keamanan untuk membatasi akses hanya ke pengguna yang berwenang
- ✅ Mempelajari contoh aturan database lainnya untuk berbagai skenario

## 🔜 Langkah Selanjutnya

Di bagian berikutnya, kita akan mulai menjelajahi cara ESP32/ESP8266 dapat berinteraksi dengan Realtime Database untuk membaca dan menulis data.

[➡️ Lanjut ke Part 3: ESP32/ESP8266: Berinteraksi dengan Realtime Database (1)](https://github.com/micro-boy/firebase-esp32-course/blob/main/firebase-esp32-course-3-1.md)
---

<div align="Left">
  <p>Dibuat dengan ❤️ oleh <a href="https://github.com/micro-boy">Micro Boy</a></p>
  <p>Referensi: Firebase Web App with ESP32 and ESP8266 oleh Rui Santos dan Sara Santos</p>
</div>
