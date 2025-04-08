# 🔥 Part 3.5: Streaming Database dan Pembaruan Status Perangkat

<div align="center">
  <img src="../images/banners/part3-5-banner.png" alt="Streaming Database and Updating Device Status" width="600">
  
  ### *Menerapkan Komunikasi Dua Arah Penuh antara Firebase dan ESP*
</div>

Pada bagian ini, kita akan mengimplementasikan kode lengkap untuk mendengarkan perubahan pada Firebase Realtime Database dan mengontrol komponen perangkat keras (LED dan OLED display) berdasarkan perubahan tersebut. Ini adalah langkah penting dalam membuat sistem IoT yang dapat dikontrol dari jarak jauh.

## 📋 Daftar Isi

- [Skema Rangkaian](#skema-rangkaian)
- [Menginstal Library yang Dibutuhkan](#menginstal-library-yang-dibutuhkan)
- [Kode ESP32](#kode-esp32)
- [Kode ESP8266](#kode-esp8266)
- [Penjelasan Kode](#penjelasan-kode)
- [Demonstrasi](#demonstrasi)
- [Langkah Selanjutnya](#langkah-selanjutnya)

## 🔌 Skema Rangkaian

Sebelum kita mulai dengan kode, mari pastikan kita memiliki semua komponen yang terhubung dengan benar. Pada proyek ini, kita akan menggunakan:

- Sensor BME280
- Layar OLED 128x64 pixels (I2C)
- 4 LED (2 untuk kontrol digital ON/OFF dan 2 untuk kontrol PWM)

Berikut adalah tabel koneksi yang menunjukkan cara menghubungkan semua komponen ke ESP32 atau ESP8266:

| Komponen | ESP32 | ESP8266 |
|----------|-------|---------|
| BME280 | SDA (GPIO21), SCL (GPIO22) | SDA (GPIO4), SCL (GPIO5) |
| OLED Display | SDA (GPIO21), SCL (GPIO22) | SDA (GPIO4), SCL (GPIO5) |
| LED 1 (Digital) | GPIO2 | GPIO2 (D4) |
| LED 2 (Digital) | GPIO12 | GPIO12 (D6) |
| LED 3 (PWM) | GPIO13 | GPIO13 (D7) |
| LED 4 (PWM) | GPIO14 | GPIO14 (D5) |

Ikuti diagram skematik di bawah ini untuk menghubungkan komponen dengan benar:

### ESP32

<div align="center">
  <img src="../images/schematics/esp32-full-circuit.png" alt="ESP32 Full Circuit" width="700">
</div>

### ESP8266

<div align="center">
  <img src="../images/schematics/esp8266-full-circuit.png" alt="ESP8266 Full Circuit" width="700">
</div>

## 📚 Menginstal Library yang Dibutuhkan

Untuk proyek ini, kita membutuhkan beberapa library tambahan selain Firebase-ESP-Client:

- **Adafruit BME280**: Untuk membaca data sensor BME280
- **Adafruit Unified Sensor**: Diperlukan oleh library BME280
- **Adafruit GFX**: Library grafis dasar untuk layar
- **Adafruit SSD1306**: Untuk mengontrol layar OLED

Jika Anda belum menginstal library ini, silakan kembali ke bagian 3.3 dan 3.4 untuk instruksi instalasi detail.

## 💻 Kode ESP32

Berikut adalah kode lengkap untuk ESP32. Anda perlu memasukkan kredensial jaringan (SSID dan password), API key proyek Firebase, URL database, serta email dan password pengguna yang diotorisasi agar kode berfungsi.

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <Firebase_ESP_Client.h>
#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BME280.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

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

//OLED Display
#define SCREEN_WIDTH 128 // Lebar layar OLED, dalam pixel
#define SCREEN_HEIGHT 64 // Tinggi layar OLED, dalam pixel
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// Variabel timer (mengirim data baru setiap dua menit)
unsigned long sendDataPrevMillis = 0;
unsigned long timerDelay = 120000;

// Deklarasi output
const int output1 = 2;
const int output2 = 12;
const int slider1 = 13;
const int slider2 = 14;

// Variabel untuk menyimpan pesan input
String message;

// Mengatur properti PWM
const int freq = 5000;
const int slider1Channel = 0;
const int slider2Channel = 1;
const int resolution = 8;

// Inisialisasi BME280
void initBME(){
  if (!bme.begin(0x76)) {
    Serial.println("Tidak dapat menemukan sensor BME280 yang valid, periksa pengkabelan!");
    while (1);
  }
}

// Inisialisasi OLED
void initOLED(){
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println(F("Alokasi SSD1306 gagal"));
    for(;;);
  }
  display.clearDisplay();
}

// Menampilkan pesan pada layar OLED
void displayMessage(String message){
  display.clearDisplay();
  display.setTextSize(2);
  display.setCursor(0,0);
  display.setTextColor(WHITE);
  display.print(message);
  display.display();
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
  
  // Dapatkan jalur yang memicu fungsi
  String streamPath = String(data.dataPath());
  
  /* Ketika pertama kali berjalan, fungsi dipicu pada jalur root (/) dan mengembalikan JSON dengan semua kunci
  dan nilai dari jalur tersebut. Jadi, kita dapat memperoleh semua nilai dari database dan memperbarui status GPIO,
  PWM, dan pesan pada OLED*/
  if (data.dataTypeEnum() == fb_esp_rtdb_data_type_json){
    FirebaseJson *json = data.to<FirebaseJson *>();
    FirebaseJsonData result;
    if (json->get(result, "/digital/" + String(output1), false)){
      bool state = result.to<bool>();
      digitalWrite(output1, state);
    }
    if (json->get(result, "/digital/" + String(output2), false)){
      bool state = result.to<bool>();
      digitalWrite(output2, state);
    }
    if (json->get(result, "/message", false)){
      String message = result.to<String>();
      displayMessage(message);
    }
    if (json->get(result, "/pwm/" + String(slider1), false)){
      int pwmValue = result.to<int>();
      ledcWrite(slider1Channel, map(pwmValue, 0, 100, 0, 255));
    }
    if (json->get(result, "/pwm/" + String(slider2), false)){
      int pwmValue = result.to<int>();
      ledcWrite(slider2Channel, map(pwmValue, 0, 100, 0, 255));
    }
  }
  
  // Periksa perubahan pada nilai output digital
  if(streamPath.indexOf("/digital/") >= 0){
    // Dapatkan panjang jalur string
    int stringLen = streamPath.length();
    // Dapatkan indeks slash terakhir
    int lastSlash = streamPath.lastIndexOf("/");
    // Dapatkan nomor GPIO (setelah slash terakhir "/")
    // UsersData/<uid>/outputs/digital/<gpio_number>
    String gpio = streamPath.substring(lastSlash+1, stringLen);
    Serial.print("GPIO DIGITAL: ");
    Serial.println(gpio);
    // Dapatkan data yang dipublikasikan pada jalur stream (status GPIO)
    if(data.dataType() == "int") {
      int gpioState = data.intData();
      Serial.print("NILAI: ");
      Serial.println(gpioState);
      //Perbarui status GPIO
      digitalWrite(gpio.toInt(), gpioState);
    }
    Serial.println();
  }
  
  // Periksa perubahan pada nilai slider
  else if(streamPath.indexOf("/pwm/") >= 0){
    // Dapatkan panjang jalur string
    int stringLen = streamPath.length();
    // Dapatkan indeks slash terakhir
    int lastSlash = streamPath.lastIndexOf("/");
    // Dapatkan nomor GPIO (setelah slash terakhir "/")
    // UsersData/<uid>/outputs/PWM/<gpio_number>
    int gpio = (streamPath.substring(lastSlash+1, stringLen)).toInt();
    Serial.print("GPIO PWM: ");
    Serial.println(gpio);
    // Dapatkan Nilai PWM
    if(data.dataType() == "int"){
      int PWMValue = data.intData();
      Serial.print("NILAI: ");
      Serial.println(PWMValue);
      // Atur duty cycle (PWM) pada channel yang sesuai
      if (gpio == slider1){
        ledcWrite(slider1Channel, map(PWMValue, 0, 100, 0, 255));
      }
      if (gpio == slider2){
        ledcWrite(slider2Channel, map(PWMValue, 0, 100, 0, 255));
      }
    }
  }
  
  // Periksa perubahan pada pesan
  else if (streamPath.indexOf("/message") >= 0){
    if (data.dataType() == "string") {
      message = data.stringData();
      Serial.print("PESAN: ");
      Serial.println(message);
      // Cetak pada OLED
      displayMessage(message);
    }
  }
  
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
  
  // Inisialisasi sensor BME, OLED, dan WiFi
  initBME();
  initOLED();
  initWiFi();
  
  // Inisialisasi Output
  pinMode(output1, OUTPUT);
  pinMode(output2, OUTPUT);
  
  // konfigurasi fungsi PWM
  ledcSetup(slider1Channel, freq, resolution);
  ledcSetup(slider2Channel, freq, resolution);
  
  // menghubungkan channel ke GPIO yang akan dikontrol
  ledcAttachPin(slider1, slider1Channel);
  ledcAttachPin(slider2, slider2Channel);
  
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

## 💻 Kode ESP8266

Gunakan kode berikut jika Anda menggunakan board ESP8266. Perbedaan utama dengan ESP32 adalah pada cara mengelola sinyal PWM. Anda perlu memasukkan kredensial jaringan (SSID dan password), API key proyek Firebase, URL database, serta email dan password pengguna yang diotorisasi agar kode berfungsi.

```cpp
#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <Firebase_ESP_Client.h>
#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BME280.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

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

//OLED Display
#define SCREEN_WIDTH 128 // Lebar layar OLED, dalam pixel
#define SCREEN_HEIGHT 64 // Tinggi layar OLED, dalam pixel
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// Variabel timer (mengirim data baru setiap dua menit)
unsigned long sendDataPrevMillis = 0;
unsigned long timerDelay = 120000;

// Deklarasi output
const int output1 = 2;
const int output2 = 12;
const int slider1 = 13;
const int slider2 = 14;

// Variabel untuk menyimpan pesan input
String message;

// Inisialisasi BME280
void initBME(){
  if (!bme.begin(0x76)) {
    Serial.println("Tidak dapat menemukan sensor BME280 yang valid, periksa pengkabelan!");
    while (1);
  }
}

// Inisialisasi OLED
void initOLED(){
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println(F("Alokasi SSD1306 gagal"));
    for(;;);
  }
  display.clearDisplay();
}

// Menampilkan pesan pada layar OLED
void displayMessage(String message){
  display.clearDisplay();
  display.setTextSize(2);
  display.setCursor(0,0);
  display.setTextColor(WHITE);
  display.print(message);
  display.display();
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
  
  // Dapatkan jalur yang memicu fungsi
  String streamPath = String(data.dataPath());
  
  /* Ketika pertama kali berjalan, fungsi dipicu pada jalur root (/) dan mengembalikan JSON dengan semua kunci
  dan nilai dari jalur tersebut. Jadi, kita dapat memperoleh semua nilai dari database dan memperbarui status GPIO,
  PWM, dan pesan pada OLED*/
  if (data.dataTypeEnum() == fb_esp_rtdb_data_type_json){
    FirebaseJson *json = data.to<FirebaseJson *>();
    FirebaseJsonData result;
    if (json->get(result, "/digital/" + String(output1), false)){
      bool state = result.to<bool>();
      digitalWrite(output1, state);
    }
    if (json->get(result, "/digital/" + String(output2), false)){
      bool state = result.to<bool>();
      digitalWrite(output2, state);
    }
    if (json->get(result, "/message", false)){
      String message = result.to<String>();
      displayMessage(message);
    }
    if (json->get(result, "/pwm/" + String(slider1), false)){
      int pwmValue = result.to<int>();
      analogWrite(slider1, map(pwmValue, 0, 100, 0, 255));
    }
    if (json->get(result, "/pwm/" + String(slider2), false)){
      int pwmValue = result.to<int>();
      analogWrite(slider2, map(pwmValue, 0, 100, 0, 255));
    }
  }
  
  // Periksa perubahan pada nilai output digital
  if(streamPath.indexOf("/digital/") >= 0){
    // Dapatkan panjang jalur string
    int stringLen = streamPath.length();
    // Dapatkan indeks slash terakhir
    int lastSlash = streamPath.lastIndexOf("/");
    // Dapatkan nomor GPIO (setelah slash terakhir "/")
    // UsersData/<uid>/outputs/digital/<gpio_number>
    String gpio = streamPath.substring(lastSlash+1, stringLen);
    Serial.print("GPIO DIGITAL: ");
    Serial.println(gpio);
    // Dapatkan data yang dipublikasikan pada jalur stream (status GPIO)
    if(data.dataType() == "int") {
      int gpioState = data.intData();
      Serial.print("NILAI: ");
      Serial.println(gpioState);
      //Perbarui status GPIO
      digitalWrite(gpio.toInt(), gpioState);
    }
    Serial.println();
  }
  
  // Periksa perubahan pada nilai slider
  else if(streamPath.indexOf("/pwm/") >= 0){
    // Dapatkan panjang jalur string
    int stringLen = streamPath.length();
    // Dapatkan indeks slash terakhir
    int lastSlash = streamPath.lastIndexOf("/");
    // Dapatkan nomor GPIO (setelah slash terakhir "/")
    // UsersData/<uid>/outputs/PWM/<gpio_number>
    int gpio = (streamPath.substring(lastSlash+1, stringLen)).toInt();
    Serial.print("GPIO PWM: ");
    Serial.println(gpio);
    // Dapatkan Nilai PWM
    if(data.dataType() == "int"){
      int PWMValue = data.intData();
      Serial.print("NILAI: ");
      Serial.println(PWMValue);
      analogWrite(gpio, map(PWMValue, 0, 100, 0, 255));
    }
  }
  
  // Periksa perubahan pada pesan
  else if (streamPath.indexOf("/message") >= 0){
    if (data.dataType() == "string") {
      message = data.stringData();
      Serial.print("PESAN: ");
      Serial.println(message);
      // Cetak pada OLED
      displayMessage(message);
    }
  }
  
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
  
  // Inisialisasi sensor BME, OLED, dan WiFi
  initBME();
  initOLED();
  initWiFi();
  
  // Inisialisasi Output
  pinMode(output1, OUTPUT);
  pinMode(output2, OUTPUT);
  pinMode(slider1, OUTPUT);
  pinMode(slider2, OUTPUT);
  
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
  
  //Rekomendasi untuk stream ESP8266, sesuaikan ukuran buffer dengan ukuran data stream Anda
  stream.setBSSLBufferSize(2048 /* Rx dalam bytes, 512 - 16384 */, 512 /* Tx dalam bytes, 512 - 16384 */);
  
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

Mari kita bahas bagian-bagian penting dari kode:

### 1️⃣ Deklarasi OLED Display

Kita menggunakan library Adafruit SSD1306 untuk mengendalikan layar OLED:

```cpp
//OLED Display
#define SCREEN_WIDTH 128 // Lebar layar OLED, dalam pixel
#define SCREEN_HEIGHT 64 // Tinggi layar OLED, dalam pixel
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);
```

### 2️⃣ Deklarasi Variabel Output

```cpp
// Deklarasi output
const int output1 = 2;    // LED digital pertama
const int output2 = 12;   // LED digital kedua
const int slider1 = 13;   // LED PWM pertama
const int slider2 = 14;   // LED PWM kedua

// Variabel untuk menyimpan pesan input
String message;
```

### 3️⃣ Konfigurasi PWM untuk ESP32

ESP32 menggunakan sistem channel PWM yang lebih fleksibel:

```cpp
// Mengatur properti PWM
const int freq = 5000;
const int slider1Channel = 0;
const int slider2Channel = 1;
const int resolution = 8;
```

Dalam `setup()`, kita mengatur channel PWM dan menghubungkannya ke GPIO:

```cpp
// konfigurasi fungsi PWM
ledcSetup(slider1Channel, freq, resolution);
ledcSetup(slider2Channel, freq, resolution);

// menghubungkan channel ke GPIO yang akan dikontrol
ledcAttachPin(slider1, slider1Channel);
ledcAttachPin(slider2, slider2Channel);
```

### 4️⃣ Fungsi Inisialisasi OLED

```cpp
// Inisialisasi OLED
void initOLED(){
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println(F("Alokasi SSD1306 gagal"));
    for(;;);  // Loop tak terbatas jika inisialisasi gagal
  }
  display.clearDisplay();
}

// Menampilkan pesan pada layar OLED
void displayMessage(String message){
  display.clearDisplay();
  display.setTextSize(2);
  display.setCursor(0,0);
  display.setTextColor(WHITE);
  display.print(message);
  display.display();
}
```

### 5️⃣ Fungsi Streaming Callback

Fungsi ini sangat penting karena dipanggil setiap kali terjadi perubahan di database:

```cpp
void streamCallback(FirebaseStream data){
  // Dapatkan jalur yang memicu fungsi
  String streamPath = String(data.dataPath());
```

Ketika ESP pertama berjalan dan terhubung, seluruh JSON structure dikirim sebagai event pertama:

```cpp
if (data.dataTypeEnum() == fb_esp_rtdb_data_type_json){
  FirebaseJson *json = data.to<FirebaseJson *>();
  FirebaseJsonData result;
  // Memeriksa dan mengatur semua perangkat berdasarkan nilai awal
}
```

Selanjutnya, kita memeriksa jenis perubahan berdasarkan jalur untuk menentukan tindakan yang sesuai:

```cpp
// Periksa perubahan pada nilai output digital
if(streamPath.indexOf("/digital/") >= 0){
  // Ekstrak nomor GPIO dan update statusnya
}
// Periksa perubahan pada nilai slider
else if(streamPath.indexOf("/pwm/") >= 0){
  // Ekstrak nomor GPIO dan sesuaikan kecerahan LED
}
// Periksa perubahan pada pesan
else if (streamPath.indexOf("/message") >= 0){
  // Update pesan pada layar OLED
}
```

### 6️⃣ Perbedaan PWM antara ESP32 dan ESP8266

Untuk ESP32, kita menggunakan:
```cpp
ledcWrite(slider1Channel, map(pwmValue, 0, 100, 0, 255));
```

Sedangkan untuk ESP8266, kita menggunakan:
```cpp
analogWrite(slider1, map(pwmValue, 0, 100, 0, 255));
```

Fungsi `map()` digunakan untuk mengkonversi dari skala 0-100 (di database) ke skala 0-255 (untuk PWM).

### 7️⃣ Pengaturan Buffer untuk ESP8266

ESP8266 memiliki keterbatasan memori, jadi kita menyesuaikan ukuran buffer:

```cpp
//Rekomendasi untuk stream ESP8266, sesuaikan ukuran buffer dengan ukuran data stream Anda
stream.setBSSLBufferSize(2048 /* Rx dalam bytes, 512 - 16384 */, 512 /* Tx dalam bytes, 512 - 16384 */);
```

## 🚀 Demonstrasi

Setelah memasukkan kredensial Anda, unggah kode ke board Anda. Buka Serial Monitor pada baud rate 115200 dan reset board.

1. Pertama, kode akan memuat data yang tersimpan di database, dan akan memperbarui status LED serta pesan pada layar OLED.

2. Kemudian, buka Firebase Realtime Database Anda dan ubah secara manual nilai digital untuk GPIO 2 dan 12, pesan, dan nilai PWM untuk GPIO 13 dan 14.

3. ESP akan mendeteksi perubahan hampir seketika. Serial Monitor akan menampilkan semua informasi terkait event, dan akan memperbarui status semua perangkat Anda.

<div align="center">
  <img src="../images/screenshots/part3/demo-full-circuit.png" alt="Working Demo" width="700">
</div>

## 🎉 Langkah Selanjutnya

Selamat! Kode ESP32/ESP8266 Anda sudah lengkap. Board ESP Anda sekarang berkomunikasi dengan Firebase Realtime Database: mendengarkan perubahan dan mengirim pembacaan sensor.

Alih-alih mengubah nilai-nilai tersebut secara manual di database untuk mengontrol GPIO ESP, kita akan membangun aplikasi web dengan antarmuka yang bagus dengan tombol dan slider untuk mengontrol ESP melalui halaman web yang dapat Anda akses dari mana saja.

Biarkan ESP menjalankan kode ini dan lanjutkan ke bagian berikutnya untuk membuat Aplikasi Web Firebase Anda.

[➡️ Lanjut ke Part 4 (1): Membuat Firebase Web App (1)](https://github.com/micro-boy/firebase-esp32-course/blob/main/firebase-esp32-course-4-1.md)
---

<div align="Left">
  <p>Dibuat dengan ❤️ oleh <a href="https://github.com/micro-boy">Micro Boy</a></p>
  <p>Referensi: Firebase Web App with ESP32 and ESP8266 oleh Rui Santos dan Sara Santos</p>
</div>
