# IoT Temperature Sensor — ESP32

Panduan utama & dokumentasi proyek firmware ESP32 untuk membaca sensor suhu,
menampilkan hasilnya di OLED SSD1306, lalu mengirim ke REST API secara berkala.
WiFi dikonfigurasi lewat captive portal (WiFiManager), jadi tidak perlu upload
ulang saat ganti jaringan. Bisa diuji dengan Wokwi (simulasi) atau hardware asli.

Proyek punya beberapa branch untuk pilihan sensor yang berbeda.

## Branch

| Branch    | Sensor              | Data                   | Keterangan                          |
| --------- | ------------------- | ---------------------- | ----------------------------------- |
| `main`    | DS18B20             | Suhu                   | Baseline (sama seperti `ds18b20`)   |
| `ds18b20` | DS18B20             | Suhu                   | Sensor suhu one-wire, butuh pull-up |
| `dht22`   | DHT22               | Suhu + kelembapan      | Menambah humidity di OLED & payload |

Setiap branch punya README sendiri yang lebih detail. README ini (branch `main`)
adalah ikhtisar proyek.

## Fitur Umum (semua branch)

- Baca sensor secara berkala (interval per branch)
- OLED SSD1306 128x64: SSID WiFi, suhu, uptime, status HTTP
- Boot animation (bitmap) di OLED
- WiFiManager captive portal — ganti WiFi tanpa reflash
- Deteksi otomatis Wokwi (`Wokwi-GUEST`) vs hardware asli
- Kirim JSON ke REST API berkala (`sendInterval`, default 5 menit)
- Tombol fisik reset WiFi (tahan 3 detik)
- Perintah via Serial: ganti device ID, reset WiFi, reboot, status
- `device_id` unik otomatis dari MAC ESP32; tersimpan di NVS & bisa di-override via Serial

## Hardware Minimal

| Komponen                | Jumlah |
| ----------------------- | ------ |
| ESP32 DevKit V1         | 1      |
| Sensor (DS18B20 / DHT22)| 1      |
| OLED SSD1306 I2C 128x64 | 1      |
| Push button             | 1      |
| Resistor 4.7kΩ          | 1 (khusus DS18B20) |

## Wiring Umum

| Perangkat       | Pin perangkat | Pin ESP32 |
| --------------- | ------------- | --------- |
| OLED VCC        | VCC           | 3V3       |
| OLED GND        | GND           | GND       |
| OLED SDA        | SDA           | GPIO21    |
| OLED SCL        | SCL           | GPIO22    |
| Sensor DATA/DQ  | DQ / SDA      | GPIO4     |
| Sensor VCC      | VCC           | 3V3       |
| Sensor GND      | GND           | GND       |
| Button          | 1             | GND       |
| Button          | 2             | GPIO18    |

- **DS18B20**: butuh resistor pull-up 4.7kΩ antara DQ dan 3V3.
- **DHT22**: butuh resistor pull-up ~10kΩ antara DATA dan 3V3 (modul breakout biasanya sudah ada).

Lihat wiring lengkap masing-masing di README branch terkait.

## Struktur File

```
.
├── diagram.json        # Skema rangkaian Wokwi
├── platformio.ini      # Konfigurasi PlatformIO + daftar library
├── wokwi.toml          # Path firmware untuk Wokwi
└── src/
    └── main.cpp        # Firmware utama
```

## Instalasi & Build

Butuh [PlatformIO](https://platformio.org/).

```bash
# Build
pio run

# Build + upload ke board
pio run -t upload

# Serial monitor
pio device monitor
```

Upload port diatur di `platformio.ini`:

```ini
upload_port = /dev/cu.usbserial-0001
monitor_port = /dev/cu.usbserial-0001
```

Cek port dengan `ls /dev/cu.*` (macOS/Linux) dan sesuaikan.

## Cara Pakai

1. Flash firmware.
2. Saat boot, OLED menampilkan animasi lalu status WiFi.
3. Jika belum pernah tersambung: perangkat membuat AP `ESP32-IoT-Setup`
   (password `12345678`). Sambungkan, buka captive portal, isi SSID + password WiFi.
4. Setelah tersambung, OLED menampilkan hasil pembacaan sensor + status API.

### Wokwi (simulasi)

Buka proyek di Wokwi; firmware terdeteksi otomatis dan tersambung ke `Wokwi-GUEST`.

## Konfigurasi Umum

Edit di `src/main.cpp` (nilai bisa beda per branch):

```cpp
const char* apiEndpoint = "http://example.com/api/temperature"; // GANTI
const char* AP_NAME     = "ESP32-IoT-Setup";
const char* AP_PASSWORD = "12345678";        // minimal 8 karakter
const unsigned long sendInterval = 300000;   // 5 menit
```

## Perintah Serial (115200 baud)

| Perintah          | Fungsi                          |
| ----------------- | ------------------------------- |
| `RESETWIFI`       | Hapus kredensial WiFi lalu reboot |
| `REBOOT`          | Reboot perangkat                |
| `STATUS`          | Tampilkan device_id, WiFi, HTTP |
| `DEVICE_ID=xxx`   | Simpan device ID baru ke NVS    |

## Format Payload API

HTTP `POST` ke `apiEndpoint`, header `Content-Type: application/json`.

Branch `main` / `ds18b20` (DS18B20):

```json
{
  "temperature": 28.50,
  "device_id": "esp32-01",
  "unit": "C",
  "uptime": 1234
}
```

Branch `dht22` (tambah humidity):

```json
{
  "temperature": 28.50,
  "humidity": 65.30,
  "device_id": "esp32-01",
  "unit": "C",
  "uptime": 1234
}
```

## Library

`platformio.ini`:

- `adafruit/Adafruit SSD1306`
- `adafruit/Adafruit GFX Library`
- `paulstoffregen/OneWire` + `milesburton/DallasTemperature` (DS18B20)
- `adafruit/DHT sensor library` + `adafruit/Adafruit Unified Sensor` (DHT22)
- `knolleary/PubSubClient`
- `tzapu/WiFiManager`