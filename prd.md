# Product Requirement Document (PRD)

## Next-Generation Network Management System (NMS 2026)

Dokumen ini mendefinisikan persyaratan produk (*Product Requirement Document*) untuk platform **Next-Generation Network Management System (NMS)** berbasis arsitektur performa tinggi menggunakan ekosistem **Rust** dan visualisasi grafis terakselerasi perangkat keras.

---

### 1. Ringkasan Eksekutif dan Visi Produk

Sistem ini dirancang untuk menggantikan pemantauan reaktif berbasis *polling* tradisional dengan platform observabilitas prediktif, terotomatisasi, dan cerdas. Platform melayani infrastruktur jaringan gabungan fisik dan komputasi awan (*hybrid*) lintas merek perangkat keras (*multi-vendor*) pada skala menengah hingga enterprise.

Sistem bertumpu pada tiga pilar operasional:

* **Melihat**: Observabilitas menyeluruh dari transmisi fisik kabel hingga lapisan inti sistem operasi server secara *real-time*.


* **Mengatur**: Pemulihan kendali mandiri dan automasi konfigurasi berbasis rekonsiliasi status target tanpa jeda komputasi.


* **Mengingatkan**: Penapisan alarm cerdas berbasis graf relasi untuk menunjuk akar masalah secara presisi.



---

### 2. Pengguna Sasaran dan Analisis Masalah

| Peran Pengguna | Masalah Utama | Kebutuhan Solusi Sistem |
| --- | --- | --- |
| **NOC Tier 1 dan Tier 2**<br> | Kelelahan notifikasi akibat banjir ribuan alarm saat kabel utama putus; proses triase insiden lambat.

 | Peringatan terkonsolidasi, isolasi alarm turunan, serta kartu insiden tunggal yang menunjukkan titik kegagalan fisik.

 |
| **Network Engineer & Architect**<br> | Kesalahan ketik sintaks konfigurasi manual; minimnya visibilitas topologi *hybrid* lintas merek.

 | Validasi konfigurasi deklaratif, pemetaan topologi otomatis, dan audit kepatuhan setelan perangkat berkala.

 |
| **Security Operations Engineer**<br> | Ketiadaan pemantauan lalu lintas data lateral antarsimpul; inventarisasi kerentanan perangkat manual.

 | Visibilitas lalu lintas internal antarsimpul dan deteksi dini celah keamanan pada komponen sistem (*firmware*).

 |

---

### 3. Kebutuhan Fungsional

**Modul 1: Penyerapan Data dan Observabilitas**

* **Pengumpulan Data Warisan**: Menyediakan protokol penarikan berkala berbasis SNMP v2c/v3 dan penampung aliran data (*flow*) untuk router, switch lama, dan perangkat nirkabel yang belum mendukung *streaming* modern menggunakan *engine* parser biner Rust berkecepatan tinggi.


* **Siaran Telemetri Sub-Detik**: Mengintegrasikan gNMI (*gRPC Network Management Interface*) berbasis pustaka Rust asinkron (`tonic`) untuk mengalirkan data kesehatan antarmuka dan lonjakan beban trafik secara kontinu dengan jeda transmisi di bawah satu detik.


* **Observabilitas Lapisan Inti Server**: Menjalankan sensor eBPF langsung di dalam kernel sistem operasi via pustaka Rust murni (`aya`) guna memantau interaksi aplikasi server, latensi soket, dan anomali koneksi internal tanpa *overhead* memori.



**Modul 2: Pemetaan Topologi Dinamis dan Graf Ketergantungan**

* **Penemuan Otomatis Multi-Protokol**: Menemukan simpul dan relasi fisik antar-perangkat secara mandiri menggunakan protokol pengenalan tetangga jaringan (LLDP, CDP, dan *peering* BGP).


* **Penyusunan Pohon Ketergantungan**: Mengonstruksi hierarki silsilah jaringan dari tingkat gerbang internet, tulang punggung pusat data, switch distribusi, hingga ke terminal kerja pengguna akhir secara *real-time*.



**Modul 3: Manajemen Insiden dan Analisis Akar Masalah (AIOps)**

* **Peredam Kepanikan Alarm**: Saat simpul induk terputus, *engine* korelasi graf Rust otomatis membungkam seluruh alarm keterputusan dari ratusan simpul turunan di bawahnya.


* **Penyajian Kartu Insiden Presisi**: Menampilkan satu tiket kerja yang membedakan antara perangkat yang mengalami kerusakan riil (*root cause*) dan perangkat yang sekadar kehilangan sinyal pasokan (*impacted*).


* **Deteksi Anomali Prediktif**: Menerapkan model deret waktu untuk mengenali deviasi performa seperti kenaikan *jitter*, penurunan level daya optik serat optik, dan saturasi buffer antarmuka sebelum terjadi pemutusan koneksi fisik.



**Modul 4: Automasi Jaringan dan Rekonsiliasi GitOps**

* **Sumber Kebenaran Tunggal (*Single Source of Truth*)**: Seluruh aturan rute dan alokasi port disimpan sebagai dokumen teks deklaratif (OpenConfig YAML/JSON) dalam repositori Git yang terlacak riwayat versinya.


* **Pemulihan Mandiri Otomatis**: Agen rekonsiliasi berbasis *Rust Temporal Worker* membandingkan konfigurasi aktif perangkat dengan status target di Git secara terus-menerus. Jika terjadi ketidaksesuaian atau kekeliruan operator, konfigurasi perangkat ditimpa ulang ke kondisi resmi dalam hitungan detik.


* **Logika Peredam Flapping**: Menerapkan *hold-down stability timer* wajib sebelum mengembalikan arus lalu lintas data ke jalur utama untuk mencegah data bolak-balik saat kabel fisik masih goyah.



---

### 4. Arsitektur Komponen dan Teknologi

| Lapisan Sistem | Tanggung Jawab | Rekomendasi Teknologi |
| --- | --- | --- |
| **Ingesti Data & Telemetri**<br> | Menampung, mem-parsing, dan memvalidasi arus telemetri berkecepatan tinggi.

 | **Rust** (`tonic`, `rasn-snmp`, `nom`, `aya`), **Vector**, **Apache Kafka**.

 |
| **Penyimpanan Deret Waktu**<br> | Menyimpan metrik performa numerik jangka panjang secara terkompresi.

 | **VictoriaMetrics** / **ClickHouse**.

 |
| **Penyimpanan Graf & Topologi**<br> | Memodelkan relasi topologi fisik, logis, dependensi, dan jalur *routing*.

 | **Neo4j** / **Memgraph**.

 |
| **Engine Orkestrasi & Automasi**<br> | Menjalankan alur kerja automasi, rekonsiliasi Git, dan alur pemulihan mandiri.

 | **Rust**, **Temporal Engine** (`temporal-sdk-core`).

 |
| **Analisis Prediktif AIOps** | Analisis tren anomali deret waktu dan kalkulasi batas saturasi buffer. | **Python 3.12+** (FastAPI, PyTorch, scikit-learn).

 |
| **Antarmuka Pengguna**<br> | Menyajikan visualisasi topologi dinamis 60 FPS dan dasbor observabilitas.

 | **Next.js 15+**, **TypeScript**, **WebGL** (PixiJS v8 / Sigma.js v3).

 |

---

### 5. Kebutuhan Non-Fungsional

**Performa dan Skalabilitas**

* **Daya Tampung Masif**: Mampu menerima dan mengolah minimal 1.000.000 titik metrik per detik dengan keterlambatan pemrosesan data maksimal 500 milidetik berkat eksekusi bebas jeda *Garbage Collector*.


* **Respons Kueri Dasbor**: Waktu muat visualisasi riwayat performa 30 hari terakhir di bawah 1 detik.


* **Ketersediaan Tinggi**: Arsitektur terdistribusi horizontal tanpa titik kegagalan tunggal (*no single point of failure*), dengan target ketersediaan layanan minimal 99,99%.



**Keamanan dan Kepatuhan**

* **Autentikasi Mandiri Berbasis Basis Data**: Menggunakan penyimpanan internal PostgreSQL 16+ dengan enkripsi kata sandi **Argon2id**, token sesi stateless **JWT**, dan verifikasi multi-faktor mandiri **TOTP (RFC 6238)** terenkripsi AES-256-GCM tanpa ketergantungan pada IdP eksternal.


* **Kontrol Akses Granular (RBAC)**: Pemisahan hak akses berbasis peran yang ketat hingga tingkat antarmuka perangkat.


* **Enkripsi Menyeluruh**: Seluruh transmisi data dienkripsi menggunakan protokol TLS 1.3, dan data tersimpan dilindungi enkripsi AES 256-bit.


* **Log Audit Tak Terubah (*Immutable Audit Trail*)**: Setiap instruksi konfigurasi, eksekusi pemulihan mandiri, dan aktivitas pengguna dicatat secara permanen dengan segel rantai hash SHA-256.



---

### 6. Metrik Keberhasilan Operasional

| Indikator Kinerja | Tolok Ukur Tradisional | Target Platform NMS 2026 |
| --- | --- | --- |
| **Mean Time to Detect (MTTD)**<br> | 5 hingga 15 menit (bergantung siklus *polling*).

 | **< 3 detik** (berbasis *streaming telemetry* gNMI).

 |
| **Mean Time to Resolve (MTTR)**<br> | 30 hingga 60 menit.

 | **< 15 menit** untuk perbaikan fisik, **sub-detik** untuk pemulihan GitOps otomatis.

 |
| **Tingkat Reduksi Alarm Bising**<br> | 0% (seluruh alarm meledak bersamaan).

 | **$\ge 85\%$** alarm turunan diredam oleh traversal graf RCA.

 |
| **Akurasi Pemetaan Topologi**<br> | Rentan usang (*stale data*) akibat survei manual. | **$\ge 99\%$** akurat secara kontinu melalui penemuan otomatis.

 |
| **Tingkat Deteksi Dini**<br> | Reaktif (menunggu laporan pengguna atau link *hard-down*).

 | **Prediktif** terhadap kehabisan kapasitas dan lonjakan suhu sebelum perangkat padam.

 |
