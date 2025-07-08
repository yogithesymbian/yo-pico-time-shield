# against-device-time-offline-mode-any-device

lets go 💡...

---

## ✅ Tujuan:

Membuat **program untuk Raspberry Pi Pico** (dengan RTC **DS3231**) agar:

* Pico bisa **baca jam dari RTC**.
* Pico bisa **respon via serial** (USB) saat Flutter (atau perangkat apa pun) **minta waktu**.
* Cocok untuk aplikasi offline mode anti manipulasi waktu.

---

## 📦 Alat yang Dibutuhkan:

| Alat              | Keterangan                         |
| ----------------- | ---------------------------------- |
| Raspberry Pi Pico | Bisa pakai Pico biasa (tanpa WiFi) |
| Modul RTC DS3231  | Real Time Clock dengan baterai     |
| Kabel Micro USB   | Untuk koneksi ke HP/PC             |
| Jumper kabel      | Untuk menghubungkan Pico ↔ RTC     |

---

## 🧠 Wiring RTC DS3231 ke Pico

```
DS3231     ↔️     Pico
------------------------
VCC        →     3.3V (pin 36)
GND        →     GND (pin 38)
SDA        →     GP0 (pin 1)
SCL        →     GP1 (pin 2)
```

> Bisa juga pakai pin SDA/SCL lain, tapi ini yang umum.

---

## 🔧 MicroPython Program (untuk Pico)

1. Siapkan firmware MicroPython ke Pico dulu (pakai [Thonny](https://thonny.org)).
2. Lalu upload script berikut sebagai `main.py`.

```python
from machine import Pin, I2C
import time
import sys

# DS3231 class
class DS3231:
    def __init__(self, i2c):
        self.i2c = i2c
        self.address = 0x68

    def _bcd2dec(self, bcd):
        return (bcd // 16) * 10 + (bcd % 16)

    def get_time(self):
        data = self.i2c.readfrom_mem(self.address, 0x00, 7)
        second = self._bcd2dec(data[0])
        minute = self._bcd2dec(data[1])
        hour = self._bcd2dec(data[2])
        day = self._bcd2dec(data[4])
        month = self._bcd2dec(data[5] & 0x1F)
        year = self._bcd2dec(data[6]) + 2000
        return f"{year:04d}-{month:02d}-{day:02d}T{hour:02d}:{minute:02d}:{second:02d}Z"

# Setup I2C
i2c = I2C(0, scl=Pin(1), sda=Pin(0), freq=400_000)
rtc = DS3231(i2c)

# Main loop: listen via USB serial
while True:
    try:
        if sys.stdin in select.select([sys.stdin], [], [], 0)[0]:
            command = sys.stdin.readline().strip()
            if command == "yo_minta_jam":
                print(rtc.get_time())
    except:
        pass
    time.sleep(0.1)
```

---

## 🔌 Uji Coba dari Komputer

Jika kamu sambung ke komputer, bisa pakai **Serial Monitor** (seperti Thonny, PuTTY, atau Arduino IDE):

1. Kirim teks: `yo_minta_jam`
2. Pico akan balas: `2025-07-08T13:14:30Z`

---

## 📲 Integrasi dengan Flutter (next step)

Flutter akan:

* Connect via `usb_serial` (Android)
* Kirim `yo_minta_jam`
* Terima timestamp `YYYY-MM-DDTHH:MM:SSZ`

---

```
// maual set time for first time
def set_time(year, month, day, hour, minute, second):
    def dec2bcd(val):
        return (val // 10) << 4 | (val % 10)
    
    rtc_data = bytearray([
        dec2bcd(second),
        dec2bcd(minute),
        dec2bcd(hour),
        0,  # Day of week (0 = ignore)
        dec2bcd(day),
        dec2bcd(month),
        dec2bcd(year - 2000)
    ])
    i2c.writeto_mem(rtc.address, 0x00, rtc_data)

// # Misal sekarang 8 Juli 2025, jam 13:45:30
// set_time(2025, 7, 8, 13, 45, 30)
// check with rtc.get_time()

// kalau otomatis
// import time
// now = time.localtime()
// set_time(now[0], now[1], now[2], now[3], now[4], now[5])

```
