# IoT Temperature Sensor — ESP32 (DS18B20)

Proyek firmware ESP32 untuk membaca **suhu** DS18B20, menampilkan di OLED,
lalu mengirim ke REST API secara berkala. WiFi dikonfigurasi lewat captive portal
(WiFiManager), jadi tidak perlu upload ulang saat ganti jaringan. Diuji dengan Wokwi
atau hardware asli.

> Branch ini (`ds18b20`) memakai sensor **DS18B20** (suhu saja, one-wire).
> Versi DHT22 (suhu + kelembapan) ada di branch `dht22`.

## Fitur

- Baca suhu DS18B20, interval 1 detik
- OLED SSD1306 128x64: SSID, suhu, uptime, status HTTP
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
| DS18B20         | 1      |
| Resistor 4.7kΩ  | 1      |
| OLED SSD1306 I2C 128x64 | 1 |
| Push button     | 1      |

## Wiring

| Perangkat   | Pin perangkat | Pin ESP32 |
| ----------- | ------------- | --------- |
| OLED VCC    | VCC           | 3V3       |
| OLED GND    | GND           | GND       |
| OLED SDA    | SDA           | GPIO21    |
| OLED SCL    | SCL           | GPIO22    |
| DS18B20 VCC | VCC           | 3V3       |
| DS18B20 GND | GND           | GND       |
| DS18B20 DQ  | DQ            | GPIO4     |
| Resistor    | 1             | 3V3       |
| Resistor    | 2             | GPIO4 (DQ) |
| Button      | 1             | GND       |
| Button      | 2             | GPIO18    |

Resistor 4.7kΩ adalah pull-up wajib di jalur DQ (DATA) → 3V3.

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
#define ONE_WIRE_BUS  4
```

## Cara Pakai

1. Flash firmware.
2. Saat boot, OLED menampilkan animasi lalu status WiFi.
3. Jika belum pernah tersambung: perangkat membuat AP `ESP32-IoT-Setup`
   (password `12345678`). Sambungkan, buka captive portal, isi SSID + password WiFi.
4. Setelah tersambung, OLED menampilkan suhu, uptime, dan status API.

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
  "device_id": "esp32-01",
  "unit": "C",
  "uptime": 1234
}
```

Turunan `ds18b20` dari proyek `iot-temperature-sensor`.