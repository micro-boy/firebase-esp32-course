# 🔥 Part 3.4: Streaming Database - Mendeteksi Perubahan di Firebase Realtime Database

<div align="center">
  <img src="../images/banners/part3-4-banner.png" alt="Streaming Data from Firebase" width="600">
  
  ### *Komunikasi Dua Arah Antara ESP dan Firebase*
</div>

Pada bagian ini, Anda akan mempelajari cara memantau perubahan pada jalur database tertentu, yang memungkinkan ESP32/ESP8266 mengetahui kapan pun terjadi perubahan di database. Fitur ini sangat penting untuk mengembangkan aplikasi IoT interaktif yang dapat dikontrol dari jarak jauh.

## 📋 Daftar Isi

- [Konsep Streaming Database](#konsep-streaming-database)
- [Mengimplementasikan Streaming Database](#mengimplementasikan-streaming-database)
- [Penjelasan Kode](#penjelasan-kode)
- [Demonstrasi](#demonstrasi)
- [Streaming Database dan Memperbarui Status Perangkat](#streaming-database-dan-memperbarui-status-perangkat)
- [Menginstal Library yang Dibutuhkan](#menginstal-library-yang-dibutuhkan)
- [Skema Rangkaian](#skema-rangkaian)
- [Langkah Selanjutnya](#langkah-selanjutnya)

## 💡 Konsep Streaming Database

Firebase Realtime Database memiliki fitur streaming yang memungkinkan aplikasi Anda mengetahui secara real-time ketika data berubah di database. Pada ESP32/ESP8266, kita dapat menggunakan fitur ini untuk:

- Mendeteksi perubahan status GPIO → mengubah status GPIO sesuai dengan perubahan
- Mendeteksi perubahan nilai PWM → menyesuaikan kecerahan LED
- Mendeteksi perubahan nilai pesan → memperbarui pesan pada layar OLED

Dibandingkan dengan polling (memeriksa database secara berkala), streaming memiliki beberapa keunggulan:

1. **Kecepatan** - Perubahan terdeteksi hampir secara instan
2. **Efisiensi bandwidth** - Hanya data yang berubah yang dikirimkan
3. **Penggunaan daya lebih rendah** - Tidak perlu melakukan permintaan berulang ke server

## 🔄 Mengimplementasikan Streaming Database

Pada bagian ini, kita akan menambahkan kode untuk mendengarkan perubahan pada jalur `UsersData/<user_uid>/outputs` untuk mendeteksi kapan pun terjadi perubahan pada status GPIO, nilai PWM, atau pesan.

Berikut adalah kode untuk mengimplementasikan streaming database. Perhatikan bahwa bagian kode baru ditandai dengan warna oranye terang:

```cpp
#include <Arduino.h>
#if defined(ESP32)
  #include <WiFi.h>
#elif defined(ESP8266)
  #include <ESP8266WiFi.h>
#endif
#include <Firebase_ESP_Client.h>
#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BME280.h>

// Menyediakan informasi proses pembuatan token
#include "addons/TokenHelper.h"
// Menyediakan info printing payload RTDB dan fungsi pembantu lainnya
#include "addons/RTDBHelper.h"

// Masukkan kredensial jaringan Anda
#define WIFI_SSID "REPLACE_WITH_YOUR_SSID"
#define WIFI_PASSWORD "REPLACE_WITH_YOUR_PASSWORD"

// Masukkan API Key proyek Firebase
#define API_KEY "REPLACE_WITH_YOUR_PROJECT_API_KEY"

// Masukkan Email dan Password yang telah diotorisasi
#define USER_EMAIL "REPLACE_WITH_THE_USER_EMAIL"
#define USER_PASSWORD "REPLACE_WITH_THE_USER_PASSWORD"

// Masukkan URL RTDB
#define DATABASE_URL "REPLACE_WITH_YOUR_DATABASE_URL"

// Mendefinisikan objek Firebase
FirebaseData stream;
FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;

// Variabel untuk menyimpan UID PENGGUNA
String uid;

// Variabel untuk menyimpan jalur database
String databasePath;
String tempPath;
String humPath;
String presPath;
String listenerPath;

// Sensor BME280
Adafruit_BME280 bme; // I2C
float temperature;
float humidity;
float pressure;

// Variabel timer (mengirim data baru setiap dua menit)
unsigned long sendDataPrevMillis = 0;
unsigned long timerDelay = 120000;

// Inisialisasi BME280
void initBME(){
  if (!bme.begin(0x76)) {
    Serial.println("Tidak dapat menemukan sensor BME280 yang valid, periksa pengkabelan!");
    while (1);
  }
}

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

// Mengirim nilai float ke database
void sendFloat(String path, float value){
  if (Firebase.RTDB.setFloat(&fbdo, path.c_str(), value)){
    Serial.print("Menulis nilai: ");
    Serial.print(value);
    Serial.print(" pada jalur berikut: ");
    Serial.println(path);
    Serial.println("BERHASIL");
    Serial.println("JALUR: " + fbdo.dataPath());
    Serial.println("TIPE: " + fbdo.dataType());
  }
  else {
    Serial.println("GAGAL");
    Serial.println("ALASAN: " + fbdo.errorReason());
  }
}

// Fungsi callback yang dijalankan saat database berubah
void streamCallback(FirebaseStream data){
  Serial.printf("jalur stream, %s\njalur event, %s\ntipe data, %s\ntipe event, %s\n\n",
                data.streamPath().c_str(),
                data.dataPath().c_str(),
                data.dataType().c_str(),
                data.eventType().c_str());
  printResult(data); //lihat addons/RTDBHelper.h
  Serial.println();
  
  // Ini adalah ukuran payload stream yang diterima (nilai saat ini dan maksimum)
  // Ukuran payload maksimum adalah ukuran payload di bawah jalur stream sejak stream terhubung
  // dan dibaca sekali dan tidak akan diperbarui sampai reconnection stream terjadi.
  // Nilai maksimum ini akan nol jika tidak ada payload yang diterima dalam kasus ESP8266 yang
  // ukuran buffer Rx BearSSL yang dipesan kurang dari payload stream aktual.
  Serial.printf("Ukuran payload stream yang diterima: %d (Maks. %d)\n\n", 
                data.payloadLength(), data.maxPayloadLength());
}

void streamTimeoutCallback(bool timeout){
  if (timeout)
    Serial.println("stream timeout, melanjutkan...\n");
  if (!stream.httpConnected())
    Serial.printf("kode error: %d, alasan: %s\n\n", stream.httpCode(), 
                  stream.errorReason().c_str());
}

void setup(){
  Serial.begin(115200);
  
  // Init sensor BME, OLED, dan WiFi
  initBME();
  initWiFi();
  
  // Menetapkan api key (diperlukan)
  config.api_key = API_KEY;

  // Menetapkan kredensial masuk pengguna
  auth.user.email = USER_EMAIL;
  auth.user.password = USER_PASSWORD;

  // Menetapkan URL RTDB (diperlukan)
  config.database_url = DATABASE_URL;

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

  // Perbarui jalur database dengan UID pengguna
  databasePath = "/UsersData/" + uid;
  
  // Tentukan jalur database untuk pembacaan sensor
  // --> UsersData/<user_uid>/sensor/temperature
  tempPath = databasePath + "/sensor/temperature";
  // --> UsersData/<user_uid>/sensor/humidity 
  humPath = databasePath + "/sensor/humidity";
  // --> UsersData/<user_uid>/sensor/pressure
  presPath = databasePath + "/sensor/pressure";

  // Perbarui jalur database untuk mendengarkan
  listenerPath = databasePath + "/outputs/";
  
  // Streaming (kapan pun data berubah pada jalur)
  // Mulai stream pada jalur database --> UsersData/<user_uid>/outputs
  if (!Firebase.RTDB.beginStream(&stream, listenerPath.c_str()))
    Serial.printf("stream begin error, %s\n\n", stream.errorReason().c_str());
  
  // Tetapkan fungsi callback untuk dijalankan ketika mendeteksi perubahan pada database
  Firebase.RTDB.setStreamCallback(&stream, streamCallback, streamTimeoutCallback);
  
  delay(2000);
}

void loop(){
  // Kirim pembacaan baru ke database
  if (Firebase.ready() && (millis() - sendDataPrevMillis > timerDelay || sendDataPrevMillis == 0)){
    sendDataPrevMillis = millis();
    
    // Dapatkan pembacaan sensor terbaru
    temperature = bme.readTemperature();
    humidity = bme.readHumidity();
    pressure = bme.readPressure()/100.0F;
    
    // Kirim pembacaan ke database:
    sendFloat(tempPath, temperature);
    sendFloat(humPath, humidity);
    sendFloat(presPath, pressure);
  }
}
```

## 📘 Penjelasan Kode

### 1️⃣ Objek Stream Data Firebase

Pertama, kita perlu membuat objek FirebaseData baru bernama `stream` untuk menangani data ketika ada perubahan pada jalur database tertentu.

```cpp
FirebaseData stream;
```

Objek ini berbeda dari objek `fbdo` yang digunakan untuk operasi database umum. Objek `stream` secara khusus digunakan untuk mendengarkan perubahan real-time.

### 2️⃣ Variabel Jalur Pendengar (Listener Path)

Kita membuat variabel baru untuk jalur pendengar bernama `listenerPath` — ini adalah jalur database yang akan kita pantau perubahannya.

```cpp
String listenerPath;
```

Kita akan memperbarui jalur database nanti dalam kode setelah mendapatkan UID pengguna. Jalur database yang ingin kita dengarkan adalah:

```
UsersData/<user_uid>/outputs
```

### 3️⃣ Fungsi Callback Stream

Bagian penting dari implementasi streaming adalah fungsi callback yang akan dipanggil setiap kali ada perubahan pada jalur database tertentu:

```cpp
void streamCallback(FirebaseStream data){
  Serial.printf("jalur stream, %s\njalur event, %s\ntipe data, %s\ntipe event, %s\n\n",
                data.streamPath().c_str(),
                data.dataPath().c_str(),
                data.dataType().c_str(),
                data.eventType().c_str());
  printResult(data); //lihat addons/RTDBHelper.h
  Serial.println();
  
  Serial.printf("Ukuran payload stream yang diterima: %d (Maks. %d)\n\n", 
                data.payloadLength(), data.maxPayloadLength());
}
```

Ketika fungsi `streamCallback()` dipicu, sebuah objek bernama `data` dari tipe `FirebaseStream` secara otomatis diberikan sebagai argumen ke fungsi tersebut. Dari objek tersebut, kita dapat memperoleh:

- **streamPath()**: Jalur lengkap yang sedang kita pantau
- **dataPath()**: Jalur relatif terhadap streamPath di mana perubahan terjadi
- **dataType()**: Tipe data dari nilai yang berubah (string, int, float, dll)
- **eventType()**: Tipe event yang memicu stream (put, patch, dll)

Fungsi `printResult(data)` berasal dari `RTDBHelper.h` dan mencetak data dalam format JSON yang mudah dibaca.

### 4️⃣ Fungsi Callback Timeout

Kita juga mendefinisikan fungsi callback untuk timeout, yang akan dipanggil jika koneksi streaming mengalami timeout:

```cpp
void streamTimeoutCallback(bool timeout){
  if (timeout)
    Serial.println("stream timeout, melanjutkan...\n");
  if (!stream.httpConnected())
    Serial.printf("kode error: %d, alasan: %s\n\n", stream.httpCode(), 
                  stream.errorReason().c_str());
}
```

Fungsi ini membantu dalam menangani kesalahan koneksi dan memberikan informasi diagnostik jika streaming mengalami masalah.

### 5️⃣ Mengatur Streaming di setup()

Dalam fungsi `setup()`, setelah mendapatkan UID pengguna, kita memperbarui jalur pendengar:

```cpp
// Perbarui jalur database untuk mendengarkan
listenerPath = databasePath + "/outputs/";
```

Jadi, jalur pendengar menjadi:

```
UsersData/<user_uid>/outputs/
```

Kapan pun ada perubahan pada node database itu atau node anak berikutnya, kita akan memicu fungsi callback.

Baris kode berikut memulai stream untuk mendengarkan perubahan pada `listenerPath`:

```cpp
if (!Firebase.RTDB.beginStream(&stream, listenerPath.c_str()))
  Serial.printf("stream begin error, %s\n\n", stream.errorReason().c_str());
```

Kemudian, kita menetapkan fungsi callback yang akan dipicu setiap kali ada perubahan pada `listenerPath` — fungsi `streamCallback`:

```cpp
Firebase.RTDB.setStreamCallback(&stream, streamCallback, streamTimeoutCallback);
```

## 🚀 Demonstrasi

Setelah memasukkan semua kredensial yang diperlukan, unggah kode ke board Anda. Setelah mengunggah, buka Serial Monitor pada baud rate 115200 dan tekan tombol RST.

Anda akan melihat bahwa ketika ESP pertama kali berjalan dan terhubung ke Realtime Database, seolah-olah ia mendeteksi perubahan pada root (/) dari jalur pendengar:

<div align="center">
  <img src="../images/screenshots/part3/stream-initial-output.png" alt="Stream Initial Output" width="700">
</div>

Objek `data` berisi objek tipe JSON (lihat gambar di atas) yang mencakup semua kunci dan nilai dari node anak `listenerPath`. Ini memungkinkan kita mengetahui semua nilai yang disimpan pada jalur pendengar ketika ESP pertama kali berjalan. Nanti, ini berguna untuk memperbarui semua status menggunakan nilai-nilai tersebut ketika ESP pertama kali dimulai atau ketika direset.

Sekarang, buka konsol Firebase Anda dan pilih Realtime Database proyek Anda. Anda dapat mengubah nilai apa pun pada jalur `UsersData/<user_uid>/outputs` secara manual. ESP akan mendeteksi perubahan tersebut. Untuk mengubah nilai di database, arahkan mouse Anda ke nilai kunci, pilih bidangnya, dan masukkan nilai lain.

Misalnya, ubah teks pada bidang pesan:

<div align="center">
  <img src="../images/screenshots/part3/change-message-firebase.png" alt="Change Message in Firebase" width="700">
</div>

ESP akan mendeteksi perubahan tersebut ketika Anda mengirimkan nilai baru, seperti yang ditunjukkan di Serial Monitor:

<div align="center">
  <img src="../images/screenshots/part3/serial-message-change.png" alt="Serial Monitor Shows Change" width="700">
</div>

Serial Monitor mencetak jalur event, tipe data (String), tipe event, dan nilai baru yang dikirimkan.

Cobalah mengubah bidang lainnya dan periksa bahwa ESP32 atau ESP8266 dapat mendeteksi perubahannya:

<div align="center">
  <img src="../images/screenshots/part3/multiple-changes-detection.png" alt="Multiple Changes Detection" width="700">
</div>

## 🎮 Streaming Database dan Memperbarui Status Perangkat

Sekarang setelah Anda mengetahui cara mendeteksi perubahan pada Realtime Database, Anda dapat mengubah status GPIO, nilai PWM, dan pesan pada layar OLED sesuai dengan perubahan tersebut. Itulah yang akan kita lakukan di bagian berikutnya.

Pada bagian selanjutnya, ESP akan memperbarui status GPIO, nilai PWM, dan pesan pada layar OLED sesuai dengan data yang disimpan di database. Jadi, kita perlu mengatur:

- Mendengarkan perubahan status GPIO → mengubah status GPIO sesuai
- Mendengarkan perubahan nilai PWM → menyesuaikan kecerahan LED
- Mendengarkan perubahan nilai pesan → memperbarui pesan pada layar OLED

## 📚 Menginstal Library yang Dibutuhkan

Selain library yang diinstal di bagian sebelumnya, Anda juga perlu menginstal library berikut untuk antarmuka dengan layar OLED:

- [Adafruit SSD1306](https://github.com/adafruit/Adafruit_SSD1306)
- [Adafruit GFX](https://github.com/adafruit/Adafruit-GFX-Library)

### 💻 VS Code + PlatformIO

Jika Anda menggunakan VS Code, ikuti instruksi berikut:

1️⃣ Dengan folder proyek Anda terbuka, klik ikon Home untuk menuju ke PlatformIO Home. Pilih ikon Libraries pada sidebar kiri.

2️⃣ Cari library yang ingin Anda instal — "Adafruit SSD1306".

<div align="center">
  <img src="../images/screenshots/part3/platformio-ssd1306-search.png" alt="Search SSD1306 Library in PlatformIO" width="700">
</div>

3️⃣ Pilih library yang ingin Anda sertakan dalam proyek Anda. Kemudian, klik **Add to Project** dan pilih proyek yang ingin Anda gunakan.

<div align="center">
  <img src="../images/screenshots/part3/platformio-add-ssd1306.png" alt="Add SSD1306 Library to Project" width="700">
</div>

4️⃣ Library Adafruit SSD1306 juga membutuhkan library Adafruit GFX untuk bekerja. Ulangi langkah 2) dan 3) tetapi cari "adafruit GFX" dan instal library Adafruit GFX.

### 🔌 Arduino IDE

Jika Anda menggunakan Arduino IDE, ikuti instruksi berikut:

1️⃣ Di Arduino IDE, buka **Sketch > Include Library > Manage Libraries**. Pengelola Library akan terbuka.

2️⃣ Cari "adafruit ssd1306" pada kotak Search dan instal library tersebut.

<div align="center">
  <img src="../images/screenshots/part3/arduino-ssd1306-library.png" alt="Install SSD1306 Library in Arduino IDE" width="700">
</div>

3️⃣ Kemudian, cari "adafruit GFX" di kotak pencarian. Gulir ke bawah untuk menemukan library dan instal.

<div align="center">
  <img src="../images/screenshots/part3/arduino-gfx-library.png" alt="Install GFX Library in Arduino IDE" width="700">
</div>

4️⃣ Setelah menginstal library, restart Arduino IDE Anda.

## 🔌 Skema Rangkaian

Hubungkan empat LED dan layar OLED ke rangkaian dengan BME280 yang sudah Anda miliki dari unit sebelumnya. Tabel berikut menunjukkan semua koneksi:

| Komponen | ESP32 | ESP8266 |
|----------|-------|---------|
| BME280 | SDA (GPIO21), SCL (GPIO22) | SDA (GPIO4), SCL (GPIO5) |
| OLED Display | SDA (GPIO21), SCL (GPIO22) | SDA (GPIO4), SCL (GPIO5) |
| LED 1 | GPIO2 | GPIO2 (D4) |
| LED 2 | GPIO12 | GPIO12 (D6) |
| LED 3 | GPIO13 | GPIO13 (D7) |
| LED 4 | GPIO14 | GPIO14 (D5) |

Anda juga dapat mengikuti diagram skematik di bawah ini.

### ESP32

<div align="center">
  <img src="../images/schematics/esp32-full-circuit.png" alt="ESP32 Full Circuit" width="700">
</div>

### ESP8266

<div align="center">
  <img src="../images/schematics/esp8266-full-circuit.png" alt="ESP8266 Full Circuit" width="700">
</div>

## 🔜 Langkah Selanjutnya

Pada bagian selanjutnya, kita akan mengimplementasikan kode lengkap yang tidak hanya mendeteksi perubahan pada database tetapi juga mengubah status GPIO, menyesuaikan nilai PWM LED, dan memperbarui pesan pada layar OLED sesuai dengan data yang disimpan di database.

Kode untuk ESP32 dan ESP8266 sedikit berbeda ketika berkaitan dengan sinyal output PWM. Selain perubahan tersebut, sketsanya identik.

[➡️ Lanjut ke Part 3 (4): Streaming Database (2)](https://github.com/micro-boy/firebase-esp32-course/blob/main/firebase-esp32-course-3-4.md)
---

<div align="Left">
  <p>Dibuat dengan ❤️ oleh <a href="https://github.com/micro-boy">Micro Boy</a></p>
  <p>Referensi: Firebase Web App with ESP32 and ESP8266 oleh Rui Santos dan Sara Santos</p>
</div>
