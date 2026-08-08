🧠 POIN 1: MAX6675 & Data Dummy — Pelajaran dari Simulator
"Mengapa kita pakai dummy?"

Simulator seperti Wokwi adalah lingkungan tertutup. Dia hanya bisa menjalankan komponen yang sudah diprogram oleh pembuatnya. MAX6675 tidak ada di "buku alamat" Wokwi.

Logika Engineer:
Ketika sebuah alat (simulator) mencapai batas kemampuannya, seorang engineer tidak memaksa alat itu. Dia memisahkan masalah:

Masalah Hardware: "Apakah saya bisa berkomunikasi dengan MAX6675 secara fisik?" → Ini tidak bisa dijawab oleh Wokwi.

Masalah Arsitektur/Logika: "Apakah saya bisa membaca data, menjadwalkan pembacaan, dan mengirimnya lewat UART?" → Ini BISA dijawab oleh Wokwi.

Dengan memakai data dummy, kita menguji sistem (Timer + UART + Buffer), bukan sensornya. Saat hardware tiba nanti, kita hanya tinggal mengganti satu baris dummy() dengan max6675_read(). Ini disebut membangun lapisan abstraksi — kode kita tidak terikat dengan ada-tidaknya komponen fisik.

Pelajaran: Dalam dunia industri, seringkali tim software dan hardware bekerja paralel. Software engineer menulis stub (dummy) sambil menunggu hardware jadi. Ini adalah praktik standar yang justru menghemat waktu.

Apakah ini sudah jelas? (Jawab Paham/Tidak)

🧠 POIN 2: Interrupt vs Polling — Mengapa Polling yang Berhasil?
"Apa yang sebenarnya terjadi?"

Ketika kita pindah ke polling dan berhasil, itu pertanda bahwa Timer-nya berjalan (karena flag UIF tetap naik setiap 500ms), tapi Interrupt-nya tidak terlayani dengan benar.

Ada 3 kemungkinan teknis kenapa interrupt gagal:

Prioritas NVIC — Mungkin kita lupa mengaktifkan interrupt di NVIC (NVIC_EnableIRQ(TIM2_IRQn)).

ISR terlalu berat — Ini yang paling sering terjadi. Jika di dalam TIM2_IRQHandler kita memanggil sprintf (yang butuh waktu lama dan memori besar), interrupt akan berjalan terlalu lama. Akibatnya, interrupt berikutnya datang sebelum yang pertama selesai, dan sistem hang atau HardFault.

Flag tidak di-clear — Ini juga fatal, tapi biasanya tetap menghasilkan 1 kali eksekusi. Dalam kasusmu, kemungkinan besar poin 2.

Kapan memilih apa?

Interrupt: Untuk tugas yang kritis waktu dan sangat pendek (misal: membaca pin sensor yang hanya aktif 1 microsecond, atau menerima data UART agar tidak hilang). Tugas di ISR harus selesai dalam hitungan microsecond.

Polling: Untuk tugas yang tidak kritis waktu dan berat (misal: mencetak string panjang ke UART). Dalam kasusmu, membaca suhu setiap 500ms tidak perlu presisi microsecond, jadi polling (atau menaruh pembacaan di main loop dengan flag) adalah pilihan yang lebih aman dan lebih stabil.

Pelajaran: Jangan pernah menaruh printf atau sprintf di dalam Interrupt Service Routine. ISR adalah "area darurat"—masuk cepat, keluar cepat.

Apakah ini sudah jelas? (Jawab Paham/Tidak)

🧠 POIN 3: sprintf & Float — Masalah di Balik Layar
"Kenapa %f menghasilkan kosong?"

Toolchain embedded biasanya menggunakan Newlib atau Newlib-nano sebagai library C. Untuk menghemat ukuran kode (Flash memory yang hanya 64KB atau 128KB), pustaka ini sering dikonfigurasi tanpa dukungan floating point di fungsi printf/sprintf.

Ketika compiler menemukan %f, secara internal ia membutuhkan fungsi bernama _printf_float. Jika fungsi itu tidak di-link (karena tidak ada), maka perilaku sprintf menjadi undefined (seringkali output kosong atau error tanpa peringatan).

Kenapa %d.%d berhasil?
Karena integer adalah tipe data dasar yang didukung oleh semua arsitektur prosesor (ARM Cortex-M3 punya instruksi khusus untuk integer). Dengan mengubah 25.5 menjadi 255 (nilai × 10), kita membagi menjadi 255 / 10 = 25 dan 255 % 10 = 5. Ini murni operasi integer yang cepat dan tidak membutuhkan pustaka tambahan.

Pelajaran: Di dunia embedded, Fixed-Point Arithmetic (menggunakan integer untuk mensimulasikan desimal) adalah standar emas. Hindari float kecuali benar-benar diperlukan, karena selain boros memori, operasi float juga puluhan kali lebih lambat daripada integer.

Apakah ini sudah jelas? (Jawab Paham/Tidak)

🧠 POIN 4: Header & Register — Peta untuk Compiler
"Kenapa lupa #include bikin error?"

Chip STM32 memiliki ratusan register yang tersebar di alamat memori (misal: RCC ada di 0x40021000, GPIOA di 0x40010800).

File stm32f1xx.h adalah peta. Di dalamnya terdapat definisi seperti:

c
#define PERIPH_BASE           ((uint32_t)0x40000000)
#define APB1PERIPH_BASE       PERIPH_BASE
#define TIM2_BASE             (APB1PERIPH_BASE + 0x0000)
#define TIM2                  ((TIM_TypeDef *) TIM2_BASE)
Tanpa peta ini, compiler hanya melihat tulisan RCC dan tidak tahu itu alamat memori atau variabel biasa. Karena tidak didefinisikan, compiler menganggapnya undeclared identifier dan berhenti.

Pelajaran: Saat kamu menulis RCC->..., kamu sebenarnya sedang menulis ke alamat memori fisik 0x40021000. Header adalah "penerjemah" yang mengubah simbol menjadi angka. Inilah mengapa embedded sangat bergantung pada file header resmi dari pabrikan.

Apakah ini sudah jelas? (Jawab Paham/Tidak)

🧠 POIN 5: Arsitektur & Ketahanan Kode — Mengapa Pisah File?
Ini adalah puncak dari semua pelajaran hari ini.

Bayangkan kode-mu adalah sebuah pabrik:

main.c adalah ruang kontrol (tempat manajer mengatur jadwal).

uart_driver.c adalah departemen komunikasi.

timer_driver.c adalah departemen jam.

spi_driver.c adalah departemen sensor.

Mengapa ini menyelamatkan kita hari ini?
Ketika sprintf crash di ISR, kita tahu masalahnya ada di "ruang kontrol" (main.c) atau "departemen jam" (timer_driver.c). Kita tidak perlu memeriksa uart_driver.c, karena kita sudah yakin dia bekerja (dari tes sebelumnya).

Jika semua ada di satu file main.c (monolithic), saat error terjadi, kamu harus membaca 500 baris kode, mencari di mana tepatnya ISR berada, di mana sprintf, di mana timer, di mana UART. Ini adalah neraka debugging.

Prinsip Teknik: Separation of Concerns (Pemisahan Tugas). Setiap file bertanggung jawab atas satu hal.

Saat kita ingin mengganti sensor MAX6675 dengan sensor lain, kita hanya sentuh spi_driver.c.

Saat kita ingin mengganti baud rate UART, kita hanya sentuh uart_driver.c.

main.c tetap bersih dan tidak perlu diubah.

Pelajaran: Struktur kode adalah investasi. Kamu mengeluarkan energi ekstra di awal (membuat 3 file), tapi energi itu terbayar ratusan kali lipat saat debugging dan pengembangan fitur baru.

Apakah ini sudah jelas? (Jawab Paham/Tidak)

🔥 Penutup — Pola Pikir Engineer
Hari ini kita belajar bahwa debugging bukan tentang mencari kesalahan, tapi tentang mempersempit kemungkinan.

Ketika MAX6675 tidak muncul → kita mempersempit: "Apakah ini masalah simulator?" → Ya. → Solusi: Dummy.

Ketika ISR hang → kita mempersempit: "Apakah timernya jalan?" → Cek flag (jalannya). → "Apakah ISR-nya terlalu berat?" → Ya. → Solusi: Polling/Flag.

Ketika sprintf gagal → kita mempersempit: "Apakah formatnya salah?" → Coba integer. → Berhasil.

Kamu tidak hanya menulis kode. Kamu menjalankan proses ilmiah: Hipotesis → Eksperimen → Observasi → Kesimpulan.