# 🔥 Part 3: ESP32/ESP8266 - Berinteraksi dengan Firebase Realtime Database

<div align="center">
  <img src="../images/banners/part3-banner.png" alt="ESP32/ESP8266 Interacting with Realtime Database" width="600">
  
  ### *Menghubungkan Perangkat IoT Anda dengan Cloud Database*
</div>

> 💡 **Tujuan Pembelajaran**: Pada bagian ini, Anda akan menguasai teknik berinteraksi dengan Firebase Realtime Database menggunakan board ESP32 atau ESP8266. Anda akan belajar cara autentikasi, membaca data, menulis data, dan mendeteksi perubahan database secara real-time.

---

## 📋 Daftar Isi

- [3.1 - Menginstal Firebase Library](#31---menginstal-firebase-library)
- [3.2 - Autentikasi dengan Email dan Password](#32---autentikasi-dengan-email-dan-password)
- [Apa yang Sudah Kita Pelajari](#-apa-yang-sudah-kita-pelajari)
- [Langkah Selanjutnya](#-langkah-selanjutnya)

---

## 3.1 - Menginstal Firebase Library

<div align="center">
  <img src="../images/icons/library.png" alt="Firebase Library" width="100">
</div>

Sekarang setelah Realtime Database sudah dibuat, kita akan belajar cara menghubungkan board ESP32 dan ESP8266 dengan database tersebut.

Kita akan menggunakan **Firebase-ESP-Client Library** oleh Mobizt untuk berinteraksi antara mikrokontroler ESP32/ESP8266 dan project Firebase.

### 🛠️ Prasyarat

Sebelum melanjutkan, Anda harus mengetahui cara memprogram board ESP32 dan ESP8266. Anda dapat menggunakan salah satu lingkungan pengembangan berikut:

| IDE | Keunggulan |
|-----|------------|
| Arduino IDE | Mudah digunakan, familiar bagi pemula |
| VS Code + PlatformIO | Fitur lengkap, manajemen library otomatis |

Pastikan Anda telah menginstal add-on untuk ESP32 atau ESP8266 sesuai dengan board yang Anda gunakan. Berikut adalah tutorial yang dapat Anda ikuti:

* [Menginstal ESP32 Board di Arduino IDE](https://randomnerdtutorials.com/installing-the-esp32-board-in-arduino-ide-windows-instructions/)
* [Menginstal ESP8266 Board di Arduino IDE](https://randomnerdtutorials.com/how-to-install-esp8266-board-arduino-ide/)
* [Memulai dengan VS Code dan PlatformIO IDE untuk ESP32 dan ESP8266](https://randomnerdtutorials.com/vs-code-platformio-ide-esp32-esp8266-arduino/)

### 📚 Firebase-ESP-Client Library

[Firebase-ESP-Client library](https://github.com/mobizt/Firebase-ESP-Client) menyediakan banyak contoh untuk menggunakan Firebase dengan board ESP32 dan ESP8266. Library ini memiliki fitur-fitur:

- Interaksi dengan Realtime Database
- Autentikasi berbasis Token
- Operasi CRUD (Create, Read, Update, Delete)
- Mendeteksi perubahan nilai secara real-time
- Manajemen aturan keamanan

> 🔍 **Info Mendalam**: Anda dapat memeriksa semua contoh library [di sini](https://github.com/mobizt/Firebase-ESP-Client/tree/main/examples). Library ini juga memiliki dokumentasi terperinci yang menjelaskan cara penggunaannya.

### ⚙️ Cara Instalasi

#### 💻 Instalasi pada VS Code + PlatformIO

Jika menggunakan VS Code + PlatformIO, Anda akan menginstal library Firebase ESP Client nanti ketika membuat folder project.

#### 🔌 Instalasi pada Arduino IDE

Jika Anda menggunakan Arduino IDE, ikuti langkah-langkah berikut:

1️⃣ Buka menu **Sketch > Include Library > Manage Libraries**
   
2️⃣ Pada kotak pencarian, ketik **Firebase ESP Client**

3️⃣ Cari dan instal **Firebase Arduino Client Library for ESP8266 and ESP32** oleh Mobitz

   <div align="center">
     <img src="../images/screenshots/part3/firebase-library-arduino.png" alt="Install Firebase Library in Arduino IDE" width="700">
   </div>

> ⚠️ **Penting**: Pastikan untuk menginstal versi terbaru dari library ini untuk mendapatkan semua fitur dan perbaikan bug terkini.

---

## 3.2 - Autentikasi dengan Email dan Password

<div align="center">
  <img src="../images/icons/authentication.png" alt="Authentication Icon" width="100">
</div>

Di bagian ini, Anda akan belajar cara mengautentikasi ESP32/ESP8266 sebagai pengguna dengan email dan password sehingga dapat berinteraksi dengan project Firebase.

### 🆕 Membuat Project Baru (VS Code)

Jika Anda baru menggunakan VS Code, ikuti panduan ini untuk membuat project baru. Jika sudah familiar, Anda dapat langsung melompat ke bagian kode.

#### Langkah-Langkah:

1️⃣ Di VS Code, klik ikon Home di bilah bawah berwarna biru. Kemudian, klik **+New Project**

   <div align="center">
     <img src="../images/screenshots/part3/platformio-new-project.png" alt="New PlatformIO Project" width="700">
   </div>

2️⃣ Beri nama project Anda (misalnya, **Authentication**)

3️⃣ Pilih board yang Anda gunakan—misalnya, DOIT ESP32 DEVKIT V1

4️⃣ Pilih Framework "Arduino" untuk menggunakan Arduino core

5️⃣ Pilih lokasi penyimpanan project (default atau kustom)

6️⃣ Klik **Finish**

   <div align="center">
     <img src="../images/screenshots/part3/platformio-project-wizard.png" alt="PlatformIO Project Wizard" width="700">
   </div>

> 💡 **Tips**: Setelah membuat project, Anda hanya perlu fokus pada file **main.cpp** (untuk kode program) dan **platformio.ini** (untuk konfigurasi).

### 📦 Menginstal ESP Firebase Client Library (VS Code)

1️⃣ Klik ikon PIO Home dan pilih tab **Libraries**

2️⃣ Cari "Firebase ESP Client"

3️⃣ Pilih **Firebase Arduino Client Library for ESP8266 and ESP32**

   <div align="center">
     <img src="../images/screenshots/part3/platformio-firebase-library.png" alt="Firebase Library in PlatformIO" width="700">
   </div>

4️⃣ Klik **Add to Project** dan pilih project yang sedang Anda kerjakan

   <div align="center">
     <img src="../images/screenshots/part3/platformio-add-to-project.png" alt="Add Firebase Library to Project" width="700">
   </div>

### 📂 Membuka Folder Project yang Sudah Ada (VS Code)

Jika Anda memiliki project yang sudah ada:

1️⃣ Ekstrak folder project dari file .ZIP (jika perlu)

2️⃣ Buka VS Code, klik PlatformIO Home, dan pilih **Open Project**

3️⃣ Telusuri dan pilih folder project Anda

   <div align="center">
     <img src="../images/screenshots/part3/platformio-open-project.png" alt="Open Existing Project in PlatformIO" width="700">
   </div>

### 💻 Kode - Autentikasi

Buat project baru untuk board ESP32 atau ESP8266 Anda menggunakan Arduino IDE atau VS Code.

> ⚠️ **Penting**: Jika menggunakan VS Code, tambahkan baris `monitor_speed = 115200` ke file platformio.ini untuk mengatur baud rate Serial Monitor.

Salin kode di bawah ini ke Arduino IDE atau ke file main.cpp jika menggunakan VS Code:

```cpp
#include <Arduino.h>
#if defined(ESP32)
  #include <WiFi.h>
#elif defined(ESP8266)
  #include <ESP8266WiFi.h>
#endif
#include <Firebase_ESP_Client.h>

// Menyediakan informasi proses pembuatan token
#include "addons/TokenHelper.h"
// Menyediakan info printing payload RTDB dan fungsi pembantu lainnya
#include "addons/RTDBHelper.h"

// Masukkan kredensial jaringan Anda
#define WIFI_SSID "REPLACE_WITH_YOUR_SSID"
#define WIFI_PASSWORD "REPLACE_WITH_YOUR_PASSWORD"

// Masukkan API Key project Firebase
#define API_KEY "REPLACE_WITH_YOUR_PROJECT_API_KEY"

// Masukkan Email dan Password yang telah diotorisasi
#define USER_EMAIL "REPLACE_WITH_THE_USER_EMAIL"
#define USER_PASSWORD "REPLACE_WITH_THE_USER_PASSWORD"

// Mendefinisikan objek Firebase
FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;

// Variabel untuk menyimpan UID PENGGUNA
String uid;

// Inisialisasi WiFi
void initWiFi() {
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Menghubungkan ke WiFi ..");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print('.');
    delay(1000);
  }
  Serial.println(WiFi.localIP());
  Serial.println();
}

void setup(){
  Serial.begin(115200);
  
  // Inisialisasi WiFi
  initWiFi();
  
  // Menetapkan api key (diperlukan)
  config.api_key = API_KEY;

  // Menetapkan kredensial masuk pengguna
  auth.user.email = USER_EMAIL;
  auth.user.password = USER_PASSWORD;

  Firebase.reconnectWiFi(true);
  fbdo.setResponseSize(4096);

  // Menetapkan callback function untuk tugas pembuatan token jangka panjang
  config.token_status_callback = tokenStatusCallback; // lihat addons/TokenHelper.h
  
  // Menetapkan percobaan ulang maksimum pembuatan token
  config.max_token_generation_retry = 5;

  // Inisialisasi library dengan autentikasi dan konfigurasi Firebase
  Firebase.begin(&config, &auth);

  // Mendapatkan UID pengguna mungkin memerlukan beberapa detik
  Serial.println("Mendapatkan User UID");
  while ((auth.token.uid) == "") {
    Serial.print('.');
    delay(1000);
  }
  
  // Cetak UID pengguna
  uid = auth.token.uid.c_str();
  Serial.print("User UID: ");
  Serial.println(uid);
}

void loop(){
  // Kode untuk loop akan ditambahkan di bagian selanjutnya
}
```

> 🔄 **Kustomisasi**: Ganti nilai-nilai pada `#define` dengan informasi Anda sendiri:
> - WIFI_SSID & WIFI_PASSWORD: Kredensial WiFi Anda
> - API_KEY: Firebase Project API Key
> - USER_EMAIL & USER_PASSWORD: Email dan password yang telah Anda tambahkan ke Authentication Users

### 📘 Penjelasan Kode

#### 1️⃣ Menyertakan Library

```cpp
#include <Arduino.h>
#if defined(ESP32)
  #include <WiFi.h>
#elif defined(ESP8266)
  #include <ESP8266WiFi.h>
#endif
#include <Firebase_ESP_Client.h>

// Menyediakan informasi proses pembuatan token
#include "addons/TokenHelper.h"
// Menyediakan info printing payload RTDB dan fungsi pembantu lainnya
#include "addons/RTDBHelper.h"
```

Bagian ini menyertakan library yang diperlukan:
- `WiFi.h` untuk ESP32 atau `ESP8266WiFi.h` untuk ESP8266
- `Firebase_ESP_Client.h` untuk berinteraksi dengan Firebase
- Add-on untuk membantu pembuatan token dan fungsi pembantu lainnya

#### 2️⃣ Mendefinisikan Konstanta

```cpp
// Masukkan kredensial jaringan Anda
#define WIFI_SSID "REPLACE_WITH_YOUR_SSID"
#define WIFI_PASSWORD "REPLACE_WITH_YOUR_PASSWORD"

// Masukkan API Key project Firebase
#define API_KEY "REPLACE_WITH_YOUR_PROJECT_API_KEY"

// Masukkan Email dan Password yang telah diotorisasi
#define USER_EMAIL "REPLACE_WITH_THE_USER_EMAIL"
#define USER_PASSWORD "REPLACE_WITH_THE_USER_PASSWORD"
```

Mendefinisikan konstanta untuk:
- Kredensial jaringan WiFi
- API Key project Firebase
- Email dan password pengguna untuk autentikasi

#### 3️⃣ Mendefinisikan Objek Firebase dan Variabel

```cpp
// Mendefinisikan objek Firebase
FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;

// Variabel untuk menyimpan UID PENGGUNA
String uid;
```

- `FirebaseData fbdo`: Objek untuk menangani data dari/ke Firebase
- `FirebaseAuth auth`: Objek untuk autentikasi
- `FirebaseConfig config`: Objek untuk konfigurasi
- `String uid`: Variabel untuk menyimpan UID pengguna

#### 4️⃣ Fungsi Inisialisasi WiFi

```cpp
// Inisialisasi WiFi
void initWiFi() {
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Menghubungkan ke WiFi ..");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print('.');
    delay(1000);
  }
  Serial.println(WiFi.localIP());
  Serial.println();
}
```

Fungsi ini:
- Menginisialisasi koneksi WiFi dengan kredensial yang diberikan
- Menampilkan status koneksi di Serial Monitor
- Menampilkan IP lokal setelah terhubung

#### 5️⃣ Fungsi Setup

```cpp
void setup(){
  Serial.begin(115200);
  
  // Inisialisasi WiFi
  initWiFi();
  
  // Menetapkan api key (diperlukan)
  config.api_key = API_KEY;

  // Menetapkan kredensial masuk pengguna
  auth.user.email = USER_EMAIL;
  auth.user.password = USER_PASSWORD;

  Firebase.reconnectWiFi(true);
  fbdo.setResponseSize(4096);

  // Menetapkan callback function untuk tugas pembuatan token jangka panjang
  config.token_status_callback = tokenStatusCallback; // lihat addons/TokenHelper.h
  
  // Menetapkan percobaan ulang maksimum pembuatan token
  config.max_token_generation_retry = 5;

  // Inisialisasi library dengan autentikasi dan konfigurasi Firebase
  Firebase.begin(&config, &auth);

  // Mendapatkan UID pengguna mungkin memerlukan beberapa detik
  Serial.println("Mendapatkan User UID");
  while ((auth.token.uid) == "") {
    Serial.print('.');
    delay(1000);
  }
  
  // Cetak UID pengguna
  uid = auth.token.uid.c_str();
  Serial.print("User UID: ");
  Serial.println(uid);
}
```

Fungsi ini melakukan:
1. Inisialisasi Serial Monitor dengan baud rate 115200
2. Memanggil fungsi initWiFi() untuk terhubung ke WiFi
3. Mengatur konfigurasi Firebase (API Key)
4. Menetapkan kredensial pengguna (email dan password)
5. Mengaktifkan fitur reconnect WiFi otomatis
6. Mengatur ukuran response data
7. Menetapkan callback untuk status token dan jumlah retry maksimum
8. Menginisialisasi Firebase dengan konfigurasi yang telah ditetapkan
9. Menunggu dan mendapatkan UID pengguna
10. Mencetak UID pengguna ke Serial Monitor

#### 6️⃣ Fungsi Loop

```cpp
void loop(){
  // Kode untuk loop akan ditambahkan di bagian selanjutnya
}
```

Fungsi loop saat ini kosong karena kita akan menambahkan kode untuk berinteraksi dengan database di bagian selanjutnya.

### 🧪 Demonstrasi

Setelah mengunggah kode, ikuti langkah-langkah berikut:

1️⃣ Buka Serial Monitor pada baud rate 115200
   > **Catatan**: Jika menggunakan VS Code, pastikan Anda telah menambahkan `monitor_speed = 115200` ke file platformio.ini

2️⃣ Reset board dengan menekan tombol EN/RST pada board

3️⃣ Amati output Serial Monitor

ESP32/ESP8266 akan:
- Terhubung ke jaringan WiFi
- Mengautentikasi ke Firebase
- Mendapatkan dan menampilkan UID pengguna

<div align="center">
  <img src="../images/screenshots/part3/esp-authentication-serial.png" alt="ESP Authentication Serial Output" width="700">
</div>

4️⃣ Verifikasi di konsol Firebase

Buka **Authentication > Users** di konsol Firebase untuk memeriksa status login.

<div align="center">
  <img src="../images/screenshots/part3/firebase-auth-signed-in.png" alt="Firebase Authentication Signed In" width="700">
</div>

Anda akan melihat:
- Tanggal saat ini di kolom **Signed In**
- UID yang cocok dengan yang ditampilkan di Serial Monitor

> 🎉 **Selamat!** Board ESP32/ESP8266 Anda telah berhasil mengautentikasi dengan Firebase.

---

## 🎓 Apa yang Sudah Kita Pelajari

Pada bagian ini, kita telah berhasil:

- ✅ Memahami cara menginstal dan menggunakan Firebase-ESP-Client Library
- ✅ Membuat project baru di Arduino IDE atau VS Code dengan PlatformIO
- ✅ Menulis kode untuk mengautentikasi ESP32/ESP8266 dengan Firebase
- ✅ Mendapatkan dan menggunakan UID pengguna untuk identifikasi
- ✅ Memverifikasi keberhasilan autentikasi melalui Serial Monitor dan konsol Firebase

## 🚀 Langkah Selanjutnya

Di bagian berikutnya (3.3), kita akan belajar cara:
- Menulis data sensor ke Firebase Realtime Database
- Membaca data pengaturan dari database
- Mendeteksi perubahan pada database secara real-time
- Mengimplementasikan kontrol perangkat melalui database

Dengan kemampuan ini, Anda akan dapat membuat aplikasi IoT yang dapat memantau dan mengontrol perangkat dari jarak jauh secara real-time.

---

<div align="Left">
  <p>Dibuat dengan ❤️ oleh <a href="https://github.com/micro-boy">Micro Boy</a></p>
  <p>Referensi: Firebase Web App with ESP32 and ESP8266 oleh Rui Santos dan Sara Santos</p>
</div>
