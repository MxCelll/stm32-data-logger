# STM32 Data Logger

Proyek ini adalah persiapan magang embedded system.  
Dibuat oleh: Muhammad Excel Trisnaro (Mahasiswa Teknologi Rekayasa Sistem Elektronika)

## Rencana Proyek
- **Tujuan**: Membangun perangkat data logger berbasis STM32 yang mampu membaca sensor suhu (MAX6675 + Thermocouple) dan gerakan (MPU6050), menyimpan data ke SD card dengan timestamp, dan mengirim ringkasan via serial (UART).
- **Mikrokontroler target**: STM32F103C8T6 (Blue Pill)
- **Fitur utama**:
  - Inisialisasi clock dan GPIO
  - Komunikasi SPI (MAX6675) dan I2C (MPU6050, DS3231)
  - Logging data ke UART (via PuTTY/Serial Monitor)
  - Penyimpanan data ke SD card dalam format CSV dengan timestamp dari RTC
  - (Opsional) Integrasi sensor tambahan

## Struktur Folder
- `/Core` : Kode aplikasi utama (main.c, interrupt handler, inisialisasi)
- `/Drivers` : Driver khusus untuk sensor dan periferal (MAX6675, MPU6050, DS3231, SD Card)
- `/Docs` : Datasheet, referensi, diagram, catatan desain

## Status Pengerjaan
- [x] Setup repository GitHub
- [x] Struktur folder siap
- [x] Project CubeIDE + debug simulator
- [x] Blinky LED pertama
- [ ] Komunikasi serial (UART)
- [ ] Integrasi sensor MAX6675 (SPI)
