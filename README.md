# IoT Temperature & Humidity Sensor — ESP32 (DHT22)

Proyek firmware ESP32 untuk membaca **suhu + kelembapan** DHT22, menampilkan di OLED,
lalu mengirim ke REST API secara berkala. WiFi dikonfigurasi lewat captive portal
(WiFiManager), jadi tidak perlu upload ulang saat ganti jaringan. Diuji dengan Wokwi
atau hardware asli.

> Branch ini (`dht22`) memakai sensor **DHT22** (suhu + kelembapan).
> Versi DS18B20 ada di branch `main` / `ds18b20`.

## Fitur

- Baca suhu & kelembapan DHT22, interval 2 detik (batas minimum DHT22)
- OLED SSD1306 128x64: SSID, suhu, kelembapan, uptime, status HTTP
- Boot animation (bitmap) di OLED
- WiFiManager captive portal — ganti WiFi tanpa reflash
- Deteksi otomatis Wokwi (`Wokwi-GUEST`) vs hardware asli
- Kirim JSON ke REST API tiap 5 menit (`sendInterval`)
- Tombol fisik reset WiFi (tahan 3 detik)
- Perintah via Serial (ganti device ID, reset, reboot, status)
- `device_id` unik otomatis dari MAC ESP32; tersimpan di NVS & bisa di-override via Serial

## Hardware

| Komponen        | Jumlah |
| --------------- | ------ |
| ESP32 DevKit V1 | 1      |
| DHT22           | 1      |
| OLED SSD1306 I2C 128x64 | 1 |
| Push button     | 1      |

## Wiring

| Perangkat  | Pin perangkat | Pin ESP32 |
| ---------- | ------------- | --------- |
| OLED VCC   | VCC           | 3V3       |
| OLED GND   | GND           | GND       |
| OLED SDA   | SDA           | GPIO21    |
| OLED SCL   | SCL           | GPIO22    |
| DHT22 VCC  | VCC           | 3V3       |
| DHT22 GND  | GND           | GND       |
| DHT22 DATA | SDA/DATA      | GPIO4     |
| Button     | 1             | GND       |
| Button     | 2             | GPIO18    |

DHT22 butuh pull-up ~10kΩ di jalur DATA → 3V3 (modul breakout biasanya sudah ada).

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

## Konfigurasi

Edit di `src/main.cpp`:

```cpp
const char* apiEndpoint = "http://example.com/api/temperature"; // GANTI
const char* AP_NAME     = "ESP32-IoT-Setup";
const char* AP_PASSWORD = "12345678";        // minimal 8 karakter
const unsigned long sendInterval = 300000;   // 5 menit
#define DHTPIN   4
#define DHTTYPE  DHT22
```

## Cara Pakai

1. Flash firmware.
2. Saat boot, OLED menampilkan animasi lalu status WiFi.
3. Jika belum pernah tersambung: perangkat membuat AP `ESP32-IoT-Setup`
   (password `12345678`). Sambungkan, buka captive portal, isi SSID + password WiFi.
4. Setelah tersambung, OLED menampilkan suhu, kelembapan, uptime, dan status API.

### Wokwi (simulasi)

Buka proyek di Wokwi; firmware terdeteksi otomatis dan tersambung ke `Wokwi-GUEST`.

## Perintah Serial (115200 baud)

| Perintah          | Fungsi                          |
| ----------------- | ------------------------------- |
| `RESETWIFI`       | Hapus kredensial WiFi lalu reboot |
| `REBOOT`          | Reboot perangkat                |
| `STATUS`          | Tampilkan device_id, WiFi, HTTP |
| `DEVICE_ID=xxx`   | Simpan device ID baru ke NVS    |

## Format Payload API

HTTP `POST` ke `apiEndpoint`, header `Content-Type: application/json`:

```json
{
  "temperature": 28.50,
  "humidity": 65.30,
  "device_id": "esp32-01",
  "unit": "C",
  "uptime": 1234
}
```

Turunan `dht22` dari proyek `iot-temperature-sensor`.