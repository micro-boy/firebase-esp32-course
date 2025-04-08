# 🔥 Part 3.3: Mengirim Data Sensor ke Firebase Realtime Database

<div align="center">
  <img src="../images/banners/part3-3-banner.png" alt="Sending Sensor Data to Firebase" width="600">
  
  ### *Memantau Data Sensor Secara Real-time dari Mana Saja*
</div>

> 💡 **Tujuan Pembelajaran**: Pada bagian ini, Anda akan belajar cara mengirim data dari sensor BME280 (suhu, kelembaban, dan tekanan) ke Firebase Realtime Database menggunakan ESP32 atau ESP8266.

---

## 📋 Daftar Isi

- [Gambaran Proyek](#gambaran-proyek)
- [Skema Rangkaian](#skema-rangkaian)
- [Menginstal Library yang Dibutuhkan](#menginstal-library-yang-dibutuhkan)
- [Kode Program](#kode-program)
- [Penjelasan Kode](#penjelasan-kode)
- [Demonstrasi](#demonstrasi)
- [Langkah Selanjutnya](#-langkah-selanjutnya)

---

## 🔍 Gambaran Proyek

Pada bagian ini, Anda akan mengirim pembacaan sensor BME280 (suhu, kelembaban, dan tekanan) ke Firebase Realtime Database pada node-node berikut:

| Data Sensor | Node Database |
|-------------|---------------|
| Suhu | `UsersData/<user_uid>/sensor/temperature` |
| Kelembaban | `UsersData/<user_uid>/sensor/humidity` |
| Tekanan | `UsersData/<user_uid>/sensor/pressure` |

> **Catatan**: `<user_uid>` adalah UID unik dari pengguna yang telah login dan terautentikasi.

## 🔌 Skema Rangkaian

Dalam proyek ini, kita akan mengirim pembacaan sensor BME280 ke Firebase Realtime Database. Hubungkan sensor BME280 ke board ESP32 atau ESP8266 pada pin I2C:

| Board | SDA | SCL |
|-------|-----|-----|
| ESP32 | GPIO 21 | GPIO 22 |
| ESP8266 | GPIO 4 (D2) | GPIO 5 (D1) |

### 📡 ESP32

Hubungkan sensor BME280 ke board ESP32 seperti yang ditunjukkan pada diagram skematik berikut:

<div align="center">
  <img src="../images/schematics/esp32-bme280.png" alt="ESP32 with BME280 Schematic" width="600">
</div>

### 📡 ESP8266

Jika Anda menggunakan ESP8266, ikuti diagram berikut:

<div align="center">
  <img src="../images/schematics/esp8266-bme280.png" alt="ESP8266 with BME280 Schematic" width="600">
</div>

## 📚 Menginstal Library yang Dibutuhkan

Ada beberapa cara untuk mendapatkan data dari sensor BME280. Kita akan menggunakan library Adafruit BME280. Ikuti langkah-langkah berikut untuk menginstal library.

### 💻 VS Code + PlatformIO

Jika Anda menggunakan VS Code, ikuti instruksi berikut untuk menginstal library BME280:

1️⃣ Dengan folder proyek Anda terbuka, klik ikon Home untuk menuju ke PlatformIO Home. Pilih ikon Libraries pada menu kiri.

2️⃣ Cari library yang ingin Anda instal—"BME280".

<div align="center">
  <img src="../images/screenshots/part3/platformio-bme280-search.png" alt="Search BME280 Library in PlatformIO" width="700">
</div>

3️⃣ Pilih "Adafruit BME280 Library". Kemudian, klik **Add to Project** dan pilih proyek yang Anda kerjakan.

<div align="center">
  <img src="../images/screenshots/part3/platformio-add-bme280.png" alt="Add BME280 Library to Project" width="700">
</div>

4️⃣ Library Adafruit BME280 juga membutuhkan library Adafruit Unified Sensor untuk bekerja. Ulangi langkah 2) dan 3) tetapi cari "adafruit sensor" dan instal library Adafruit Unified Sensor.

### 🔌 Arduino IDE

Jika Anda menggunakan Arduino IDE, ikuti instruksi berikut:

1️⃣ Di Arduino IDE, buka **Sketch > Include Library > Manage Libraries**. Pengelola Library akan terbuka.

2️⃣ Cari "adafruit bme280" pada kotak Search dan instal library tersebut.

<div align="center">
  <img src="../images/screenshots/part3/arduino-bme280-library.png" alt="Install BME280 Library in Arduino IDE" width="700">
</div>

3️⃣ Kemudian, cari "Adafruit Unified Sensor" di kotak pencarian. Gulir ke bawah untuk menemukan library dan instal.

<div align="center">
  <img src="../images/screenshots/part3/arduino-unified-sensor.png" alt="Install Unified Sensor Library in Arduino IDE" width="700">
</div>

4️⃣ Setelah menginstal library, restart Arduino IDE Anda.

## 💻 Kode Program

Berikut adalah kode yang mengirim pembacaan sensor BME280 ke database setiap tiga menit. Setelah menguji kode dan memastikan bahwa semuanya berfungsi sebagaimana mestinya, kami merekomendasikan untuk meningkatkan waktu tunda.

> ⚠️ **Penting**: Anda perlu memasukkan kredensial jaringan (SSID dan password), API key proyek Firebase, URL database, serta email dan password pengguna yang memiliki otorisasi.

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

// Sensor BME280
Adafruit_BME280 bme; // I2C
float temperature;
float humidity;
float pressure;

// Variabel timer (mengirim data baru setiap tiga menit)
unsigned long sendDataPrevMillis = 0;
unsigned long timerDelay = 180000;

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

void setup(){
  Serial.begin(115200);
  
  // Inisialisasi sensor BME280
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

  // Perbarui jalur database
  databasePath = "/UsersData/" + uid;
  
  // Tentukan jalur database untuk pembacaan sensor
  // --> UsersData/<user_uid>/sensor/temperature
  tempPath = databasePath + "/sensor/temperature";
  // --> UsersData/<user_uid>/sensor/humidity
  humPath = databasePath + "/sensor/humidity";
  // --> UsersData/<user_uid>/sensor/pressure
  presPath = databasePath + "/sensor/pressure";
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

### 1️⃣ URL Database

Untuk berinteraksi dengan database, Anda perlu memasukkan URL database Anda.

```cpp
// Masukkan URL RTDB
#define DATABASE_URL "REPLACE_WITH_YOUR_DATABASE_URL"
```

Anda dapat mendapatkan URL database dengan membuka konsol Firebase, memilih tab Realtime Database, dan menyalin URL seperti yang disorot di bawah ini:

<div align="center">
  <img src="../images/screenshots/part3/firebase-database-url.png" alt="Get Firebase Database URL" width="700">
</div>

### 2️⃣ Deklarasi Variabel

Variabel berikut akan digunakan untuk menyimpan node tempat kita akan mengirim pembacaan sensor. Kita akan memperbarui variabel ini nanti dalam kode saat kita mendapatkan UID pengguna.

```cpp
// Variabel untuk menyimpan jalur database
String databasePath;
String tempPath;
String humPath;
String presPath;
```

Kemudian, kita membuat objek `Adafruit_BME280` bernama `bme`. Ini secara otomatis membuat objek sensor pada pin I2C default ESP32 atau ESP8266.

```cpp
// Sensor BME280
Adafruit_BME280 bme; // I2C
```

Variabel berikut akan menyimpan pembacaan suhu, kelembaban, dan tekanan dari sensor.

```cpp
float temperature;
float humidity;
float pressure;
```

### 3️⃣ Pengaturan Waktu Tunda

Variabel `sendDataPrevMillis` dan `timerDelay` digunakan untuk memeriksa waktu tunda antara setiap pengiriman. Dalam contoh ini, kita menetapkan waktu tunda menjadi 3 menit (180000 milidetik). Setelah Anda menguji proyek ini dan memeriksa bahwa semuanya berfungsi sebagaimana mestinya, kami merekomendasikan untuk meningkatkan waktu tunda untuk mengurangi jumlah data yang dikirim.

```cpp
// Variabel timer (mengirim data baru setiap tiga menit)
unsigned long sendDataPrevMillis = 0;
unsigned long timerDelay = 180000;
```

### 4️⃣ Fungsi initBME()

Fungsi `initBME()` menginisialisasi library BME280 menggunakan objek `bme` yang dibuat sebelumnya. Kemudian, Anda harus memanggil fungsi ini di `setup()`.

```cpp
// Inisialisasi BME280
void initBME(){
  if (!bme.begin(0x76)) {
    Serial.println("Tidak dapat menemukan sensor BME280 yang valid, periksa pengkabelan!");
    while (1);
  }
}
```

> 📝 **Catatan**: Alamat I2C default untuk BME280 adalah `0x76`. Jika sensor Anda menggunakan alamat yang berbeda (misalnya `0x77`), Anda perlu mengubah parameter dalam fungsi `begin()`.

### 5️⃣ Fungsi sendFloat()

Untuk menyimpan data pada node tertentu di Firebase Realtime Database, Anda dapat menggunakan fungsi-fungsi berikut: `set`, `setInt`, `setFloat`, `setDouble`, `setString`, `setJSON`, `setArray`, `setBlob`, dan `setFile`.

Fungsi-fungsi ini mengembalikan nilai boolean yang menunjukkan keberhasilan operasi, yang akan bernilai `true` jika semua kondisi berikut terpenuhi:
1. Server mengembalikan kode status HTTP 200
2. Tipe data cocok antara permintaan dan respons

Dalam contoh kita, kita akan mengirim variabel float, jadi kita perlu menggunakan fungsi `setFloat()` sebagai berikut:

```cpp
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
```

### 6️⃣ Fungsi setup()

Dalam `setup()`, kita melakukan beberapa inisialisasi penting:

```cpp
void setup(){
  Serial.begin(115200);
  
  // Inisialisasi sensor BME280
  initBME();
  initWiFi();
```

Pertama, kita menginisialisasi Serial Monitor dengan baud rate 115200, kemudian memanggil fungsi `initBME()` untuk menginisialisasi sensor BME280 dan `initWiFi()` untuk menghubungkan ke jaringan WiFi.

```cpp
  // Menetapkan URL RTDB (diperlukan)
  config.database_url = DATABASE_URL;
```

Kita menetapkan URL database ke objek konfigurasi Firebase.

Setelah mendapatkan UID pengguna, kita dapat memperbarui jalur node database untuk menyertakan UID pengguna. Itulah yang kita lakukan di baris-baris berikut.

```cpp
  // Perbarui jalur database
  databasePath = "/UsersData/" + uid;
  
  // Tentukan jalur database untuk pembacaan sensor
  // --> UsersData/<user_uid>/sensor/temperature
  tempPath = databasePath + "/sensor/temperature";
  // --> UsersData/<user_uid>/sensor/humidity
  humPath = databasePath + "/sensor/humidity";
  // --> UsersData/<user_uid>/sensor/pressure
  presPath = databasePath + "/sensor/pressure";
```

### 7️⃣ Fungsi loop()

Dalam `loop()`, kita memeriksa apakah sudah waktunya untuk mengirim pembacaan baru:

```cpp
void loop(){
  // Kirim pembacaan baru ke database
  if (Firebase.ready() && (millis() - sendDataPrevMillis > timerDelay || sendDataPrevMillis == 0)){
    sendDataPrevMillis = millis();
```

Jika ya, dapatkan pembacaan sensor terbaru dari sensor BME280 dan simpan dalam variabel `temperature`, `humidity`, dan `pressure`.

```cpp
    // Dapatkan pembacaan sensor terbaru
    temperature = bme.readTemperature();
    humidity = bme.readHumidity();
    pressure = bme.readPressure()/100.0F; 
```

Akhirnya, panggil fungsi `sendFloat()` untuk mengirim pembacaan baru ke database. Berikan jalur node dan nilai sebagai argumen.

```cpp
    // Kirim pembacaan ke database:
    sendFloat(tempPath, temperature); 
    sendFloat(humPath, humidity); 
    sendFloat(presPath, pressure); 
  }
}
```

## 🚀 Demonstrasi

Setelah memasukkan URL database Anda pada kode, upload ke board Anda. Kemudian, buka Serial Monitor pada baud rate 115200 dan reset board. Anda akan mendapatkan pesan keberhasilan, dan pembacaan sensor akan dipublikasikan ke node yang sesuai.

<div align="center">
  <img src="../images/screenshots/part3/serial-output-sensor.png" alt="Serial Monitor Output" width="700">
</div>

Pada konsol Firebase, buka Realtime Database Anda. Data sensor baru akan dimasukkan setiap tiga menit. Kapan pun perubahan terjadi, data akan berkedip dengan warna oranye.

<div align="center">
  <img src="../images/screenshots/part3/firebase-sensor-data.png" alt="Firebase Database with Sensor Data" width="700">
</div>

> 🎉 **Selamat!** Anda telah berhasil mengirim data ke Realtime Database.

## 📊 Mengoptimalkan Pengiriman Data

Untuk aplikasi IoT dunia nyata, berikut beberapa tips untuk mengoptimalkan pengiriman data:

1. **Atur interval yang tepat** - Pilih interval pengiriman yang sesuai dengan kebutuhan aplikasi Anda. Data suhu ruangan mungkin hanya perlu diperbarui setiap 5-15 menit, sementara pemantauan industri mungkin memerlukan pembaruan lebih sering.

2. **Kirim data hanya saat berubah** - Untuk menghemat bandwidth dan kuota database, pertimbangkan untuk mengirim data hanya ketika nilainya berubah secara signifikan (misalnya, perubahan suhu > 0.5°C).

3. **Batasi presisi desimal** - Jika Anda tidak memerlukan presisi tinggi, batasi jumlah angka desimal yang disimpan. Misalnya: `temperature = round(bme.readTemperature() * 10) / 10;` untuk menyimpan hanya satu angka desimal.

4. **Gabungkan beberapa pembacaan** - Jika Anda perlu menyimpan riwayat data, pertimbangkan untuk menggabungkan beberapa pembacaan sebelum mengirimnya ke database.

## 🔜 Langkah Selanjutnya

Di bagian berikutnya, Anda akan belajar cara mengatur listener pada jalur tertentu sehingga Anda tahu kapan pun ada perubahan database pada status GPIO atau pada node pesan (pada jalur `UsersData/<user_uid>/outputs`).

[➡️ Lanjut ke Part 3 (3): Streaming Database](https://github.com/micro-boy/firebase-esp32-course/blob/main/firebase-esp32-course-3-3.md)
---

<div align="Left">
  <p>Dibuat dengan ❤️ oleh <a href="https://github.com/micro-boy">Micro Boy</a></p>
  <p>Referensi: Firebase Web App with ESP32 and ESP8266 oleh Rui Santos dan Sara Santos</p>
</div>
