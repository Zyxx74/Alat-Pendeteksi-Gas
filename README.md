# Alat-Pendeteksi-Gas# Edge AI Air Quality Monitor: Real-Time Classification & Forecasting on ESP32

Purwarupa sistem tertanam (*embedded system*) berbasis **Edge AI / TinyML** untuk memantau, mengklasifikasikan, dan memprediksi tren kualitas udara secara *real-time*. Sistem ini ditenagai oleh mikrokontroler **ESP32 DevKit v1** dengan implementasi struktur data dan algoritma *Machine Learning* yang dibangun murni dari nol (tanpa pustaka/library ML pihak ketiga).

Proyek ini diajukan untuk memenuhi Tugas Proyek Akhir mata kuliah **Algoritma dan Pemrograman**.

---

## 🚀 Fitur Utama (Tiga Pilar Sistem)

### 1. Pilar 1 — Algoritma & Struktur Data (Murni C++)
* **ADC Noise Reduction:** Implementasi filter *Moving Average* (10 sampel) untuk mereduksi *noise* pembacaan analog kimia.
* **Circular Buffer:** Penyangga data historis berkapasitas 60 sampel ($O(1)$ *push*) untuk efisiensi memori RAM.
* **Feature Extraction:** Mengekstrak 3 fitur statistik vital (*Mean* PPM, *Variance* PPM, dan *Mean* Temperatur) dari jendela data konstan ($w=10$).
* **Time-Series Forecasting:** Menggunakan **Regresi Linier (Least Squares)** secara dinamis untuk meramal konsentrasi gas pada waktu $t+1, t+2,$ dan $t+3$.

### 2. Pilar 2 — Komponen AI / Machine Learning
* **k-Nearest Neighbors (k-NN) Classifier:** Klasifikasi cerdas lokal ke dalam 5 kelas indeks kualitas udara (AQI) dengan parameter $k=3$ dan optimasi *partial insertion sort*.
* **Self-Evaluation Model:** Validasi akurasi model otomatis menggunakan metode *Leave-One-Out Cross Validation (LOO)* saat fase inisialisasi perangkat.

### 3. Pilar 3 — Perangkat Keras & IoT Cloud
* **Aksi Aktuator Lokal:** Transisi mulus 3 level LED indikator dan alarm peringatan *Buzzer* yang berjalan asinkron (tanpa fungsi `delay()` yang memblokir program).
* **Dual-Cloud Integration:** Visualisasi data interaktif via *BDashboard* Blynk IoT sekaligus pencatatan data riwayat terautomasi (*data logging*) ke **Google Sheets** via HTTP POST/GET Request setiap 30 detik.

---

## 🔌 Skema Konfigurasi Pin (Hardware Wiring)

| Komponen | Pin Komponen | Pin ESP32 | Keterangan |
| :--- | :--- | :--- | :--- |
| **MQ-135** | VCC | 5V / VIN | Tegangan kerja elemen pemanas (*heater*) |
| | GND | GND | *Common Ground* |
| | AOUT | **GPIO 34** | Input analog murni (ADC1_CH6) |
| **DHT11** | VCC | 3.3V | Tegangan kerja sensor |
| | DATA | **GPIO 4** | Jalur data *1-wire* (+ Resistor *Pull-up* 10kΩ) |
| | GND | GND | *Common Ground* |
| **OLED SSD1306** | VCC | 3.3V | Tegangan kerja modul display |
| | SDA | **GPIO 21** | Jalur data I2C standar ESP32 |
| | SCL | **GPIO 22** | Jalur *clock* I2C standar ESP32 |
| | GND | GND | *Common Ground* |
| **Buzzer Aktif** | Positif (+) | **GPIO 5** | Sinyal digital *output* (Aktif HIGH saat polusi) |
| | Negatif (-) | GND | *Common Ground* |
| **LED Hijau** | Positif (Anoda)| **GPIO 12** | Indikator kondisi *FRESH AIR* / *GOOD* |
| **LED Biru** | Positif (Anoda)| **GPIO 13** | Indikator kondisi *MODERATE* |
| **LED Merah** | Positif (Anoda)| **GPIO 14** | Indikator kondisi *POOR* / *HAZARDOUS* |
| *(Semua LED)* | Negatif (Katoda)| GND | Wajib menggunakan Resistor 220Ω - 330Ω |

---

## 📈 Analisis Kompleksitas & Memori

Sistem beroperasi secara efisien dengan batas komputasi lokal yang sangat ketat:
* **Manipulasi Antrian Buffer:** $O(1)$
* **Ekstraksi Fitur:** $O(w)$ (linier terhadap ukuran jendela data, $w=10$).
* **Klasifikasi k-NN:** $O(n \cdot f + n \cdot k) \approx O(n)$ (linier terhadap jumlah data latih, $n=40$).
* **Regresi Linier:** $O(n)$
* **Efisiensi Memori (RAM):** Keseluruhan model ML, *circular buffer*, dan dataset latih hanya mengonsumsi memori sebesar **$\approx 1.16 \text{ KB}$** (kurang dari **0.5%** dari total 320 KB RAM bebas pada chip ESP32).

---

## 🛠️ Langkah Instalasi & Penggunaan

1.  **Clone Repositori Ini:**
    ```bash
    git clone [https://github.com/USERNAME_KAMU/NAMA_REPO.git](https://github.com/USERNAME_KAMU/NAMA_REPO.git)
    ```
2.  **Persiapan IDE:**
    * Buka Arduino IDE atau PlatformIO.
    * Instal *third-party libraries* yang dibutuhkan melalui Library Manager:
        * `Blynk` oleh Volodymyr Shymanskyy
        * `DHT sensor library` oleh Adafruit
        * `Adafruit SSD1306` & `Adafruit GFX Library`
3.  **Konfigurasi Kode:**
    * Buka file `.ino`, lengkapi baris kredensial Blynk Anda:
        ```cpp
        #define BLYNK_TEMPLATE_ID   "TMPLxxxxxxxxx"
        #define BLYNK_AUTH_TOKEN    "xxxxxxxxxxxxxxxxxxxx"
        ```
    * Sesuaikan nama SSID dan Password WiFi lokal Anda.
    * Masukkan URL Deployment Google Apps Script Anda pada variabel `GOOGLE_SCRIPT_URL`.
4.  **Upload & Running:**
    * Pilih *Board*: **DOIT ESP32 DEVKIT V1**.
    * Lakukan kompilasi (*Compile*) dan unggah (*Upload*) kode ke ESP32 Anda.
    * Buka Serial Monitor pada *baud rate* `115200` untuk melihat proses evaluasi akurasi LOO dan log data komputasi AI.

---

