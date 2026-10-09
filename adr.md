# Architecture Decision Records (ADR)
## Next-Generation Network Management System (NMS 2026)

Dokumen ini mendokumentasikan keputusan arsitektural penting (Architecture Decision Records) untuk pembangunan platform **Next-Generation Network Management System (NMS)** berdasarkan spesifikasi pada [prd.md](file:///home/rizxenzs/Desktop/nms/prd.md).

---

## Daftar Isi ADR

- [ADR-001: Strategi Penyimpanan Ganda (Dual-Storage Architecture: Time-Series & Graph Database)](#adr-001-strategi-penyimpanan-ganda-dual-storage-architecture)
- [ADR-002: Protokol Pengumpulan Data Bertingkat (gNMI Streaming Telemetry, SNMP Gateway, dan eBPF)](#adr-002-protokol-pengumpulan-data-bertingkat)
- [ADR-003: Pipeline Ingesti dan Message Streaming Backbone (Apache Kafka & Vector)](#adr-003-pipeline-ingesti-dan-message-streaming-backbone)
- [ADR-004: Otomasi Konfigurasi Deklaratif dan Pemulihan Mandiri Berbasis GitOps (Temporal Engine)](#adr-004-otomasi-konfigurasi-deklaratif-dan-gitops)
- [ADR-005: Korelasi Alarm dan Root Cause Analysis (RCA) Berbasis Graf Dependensi AIOps](#adr-005-korelasi-alarm-dan-root-cause-analysis-aiops)
- [ADR-006: Arsitektur Antarmuka Visualisasi Topologi Skala Besar Berbasis WebGL](#adr-006-arsitektur-antarmuka-visualisasi-topologi-webgl)
- [ADR-007: Autentikasi Mandiri Berbasis Basis Data (Internal / Custom Database-Backed Auth)](#adr-007-autentikasi-mandiri-berbasis-basis-data-internal-custom-auth)

---

## ADR-001: Strategi Penyimpanan Ganda (Dual-Storage Architecture)

* **Status**: Accepted
* **Tanggal**: 2026-10-09
* **Pengambil Keputusan**: Arsitek Solusi & Tim Rekayasa Inti

### 1. Konteks & Masalah
Sistem NMS mengelola dua karakteristik data yang bertolak belakang:
1. **Data Metrik Performa (Deret Waktu)**: Volume masif ($\ge 1.000.000$ titik metrik/detik) dengan operasi tulis terus-menerus dan kueri analitik berbasis rentang waktu (misal: utilisasi bandwidth 30 hari terakhir).
2. **Data Topologi & Dependensi (Graf)**: Model relasi fisik dan logis antar perangkat (induk-anak, link agregasi, rute BGP) yang membutuhkan traversal rekursif berkecepatan tinggi untuk melacak jalur kegagalan (*failure propagation*).

Database relasional tradisional (RDBMS) seperti PostgreSQL/MySQL tidak mampu menangani throughput tulis 1 juta metrik/detik secara efisien dan mengalami degradasi eksponensial saat melakukan join rekursif mendalam untuk topologi jaringan ribuan simpul.

### 2. Pilihan yang Dipertimbangkan
* **Opsi A**: Single Database Relasional / Time-Series Hybrid (misal: PostgreSQL + TimescaleDB + ekstensi Apache AGE).
* **Opsi B**: Poliglut Multi-Model Berbasis Dokumen (misal: MongoDB / Elasticsearch).
* **Opsi C**: Dual-Storage Engine Terspesialisasi: Time-Series DB (VictoriaMetrics / ClickHouse) + Graph DB (Neo4j / Memgraph).

### 3. Keputusan
Memilih **Opsi C**: Mengadopsi arsitektur **Dual-Storage Engine**:
* **Time-Series Engine**: Menggunakan **VictoriaMetrics / ClickHouse** untuk seluruh metrik numerik berstempel waktu (CPU, RAM, daya optik, counter paket).
* **Graph Engine**: Menggunakan **Neo4j / Memgraph** untuk menyimpan topologi fisik/logis (nodes, interfaces, cables, BGP peerings) dan hierarki dependensi.
* **Metrik Penyerta**: UUID perangkat dan antarmuka bertindak sebagai foreign-key penghubung antar kedua database.

### 4. Konsekuensi
* **Positif**:
  * Efisiensi kompresi metrik tinggi (ClickHouse/VictoriaMetrics mampu mencapai rasio kompresi 5-10x dan penulisan jutaan data/detik).
  * Traversal graf (mencari simpul terdampak kegagalan) selesai dalam waktu sub-10ms tanpa locking pada tabel metrik.
  * Beban kueri dasbor analitik tidak mengganggu proses kalkulasi jalur topologi.
* **Negatif & Mitigasi**:
  * *Eventual Consistency*: Inkonsistensi sementara antara metrik dan graf dimitigasi dengan pengiriman event update melalui Kafka.
  * Kompleksitas operasional: Membutuhkan pengelolaan dua cluster database terpisah yang dimonitor dengan observabilitas infrastruktur tersendiri.

---

## ADR-002: Protokol Pengumpulan Data Bertingkat

* **Status**: Accepted
* **Tanggal**: 2026-10-09
* **Pengambil Keputusan**: Network Architect & Telemetry Lead

### 1. Konteks & Masalah
Jaringan enterprise bersifat *hybrid multi-vendor* yang mencakup:
* Perangkat modern pusat data (Cisco Nexus, Arista, Juniper) yang mendukung *push telemetry*.
* Perangkat *legacy* (switch akses lama, PDU, AP Wi-Fi) yang hanya mendukung SNMP v2c/v3 berbasis polling.
* Node server bare-metal dan kluster kontainer Kubernetes yang memerlukan visibilitas beban internal.

Jika hanya mengandalkan SNMP, MTTD akan tertahan di rentang 5–15 menit. Jika hanya mendukung gNMI, perangkat *legacy* akan menjadi *blind spots* (titik buta).

### 2. Pilihan yang Dipertimbangkan
* **Opsi A**: Standardisasi tunggal menggunakan SNMP Polling frekuensi tinggi (polling per 10 detik).
* **Opsi B**: Pendekatan bertingkat (*Tiered Ingestion*):
  * **gNMI (OpenConfig/gRPC)** untuk router/switch modern (push sub-detik).
  * **SNMP Gateway & Trap Listener** untuk perangkat legacy.
  * **eBPF Agent** untuk host Linux, server bare-metal, dan kontainer.

### 3. Keputusan
Memilih **Opsi B**: Mengimplementasikan **Tiered Telemetry Ingestion Pipeline**:
1. **gNMI over gRPC (OpenConfig)**: Sebagai protokol prioritas utama untuk streaming data antarmuka, status BGP, dan counter trafik secara kontinu.
2. **SNMP v3 / Flow Ingestion Gateway**: Berfungsi sebagai adapter kompatibilitas untuk perangkat warisan, menggunakan polling adaptif (heartbeat ringan per 5 detik, metrik berat per 5 menit) dan SNMP Traps / Syslog.
3. **eBPF DaemonSet / Agent**: Menginjeksi probe observabilitas ke Linux kernel untuk melacak throughput socket, RTT, dan penurunan paket intra-host.

### 4. Konsekuensi
* **Positif**:
  * Memenuhi target MTTD < 3 detik untuk infrastruktur tulang punggung modern.
  * Nol *blind spots* pada armada perangkat warisan tanpa membebani CPU perangkat tua.
  * Visibilitas *east-west traffic* internal server tercapai tanpa memasang proxy berat.
* **Negatif & Mitigasi**:
  * Membutuhkan normalisasi skema data (data parsing layer) untuk menyatukan data SNMP MIB dan OpenConfig YANG models ke dalam format canonical payload.

---

## ADR-003: Pipeline Ingesti dan Message Streaming Backbone

* **Status**: Accepted
* **Tanggal**: 2026-10-09
* **Pengambil Keputusan**: Data Platform & Systems Architect

### 1. Konteks & Masalah
Pipeline ingesti harus mampu menyerap minimal **1.000.000 metrik/detik** dengan batas latensi transmisi ke database kurang dari **500 milidetik**, serta tahan terhadap lonjakan trafik (*traffic spikes*) saat terjadi badai alarm.

### 2. Pilihan yang Dipertimbangkan
* **Opsi A**: Direct Write dari Telemetry Collector langsung ke VictoriaMetrics dan Neo4j.
* **Opsi B**: RabbitMQ / Redis Streams sebagai message buffer.
* **Opsi C**: Pipeline berbasis **Vector (Data Aggregator) + Apache Kafka Cluster**.

### 3. Keputusan
Memilih **Opsi C**:
* **Vector**: Ditempatkan di edge/collector tier untuk parsing, filtering, deduplikasi, dan transformasi data telemetri dengan performa tinggi (berbasis Rust, memory-safe).
* **Apache Kafka**: Berfungsi sebagai buffer terdistribusi berkecepatan tinggi dengan *partitioning key* berbasis `device_id` / `site_id` untuk memastikan penjaminan urutan pesan (*ordered event stream*).

### 4. Konsekuensi
* **Positif**:
  * *Decoupling*: Kegagalan pada lapisan database tidak menyebabkan packet drop pada telemetri perangkat jaringan.
  * *Backpressure handling*: Kafka dapat menahan antrean jutaan pesan selama periode puncak gangguan jaringan.
  * Fleksibilitas *multi-consumer*: Data yang sama dapat dikonsumsi paralel oleh Time-Series DB, Graph Engine, dan AI Anomaly Detection Engine.
* **Negatif & Mitigasi**:
  * Kebutuhan resource infrastruktur Kafka cluster. Dimodernisasi menggunakan mode Kafka KRaft (tanpa Zookeeper) untuk menyederhanakan pemeliharaan.

---

## ADR-004: Otomasi Konfigurasi Deklaratif dan GitOps Self-Healing

* **Status**: Accepted
* **Tanggal**: 2026-10-09
* **Pengambil Keputusan**: Automation & DevOps Lead

### 1. Konteks & Masalah
Konfigurasi jaringan secara manual berbasis CLI rentan terhadap *human error* dan konfigurasi *drift*. Pemulihan mandiri (*self-healing*) membutuhkan jaminan transaksi (*state consistency*), fitur rollback otomatis bila verifikasi gagal, serta logika pencegahan *link flapping*.

### 2. Pilihan yang Dipertimbangkan
* **Opsi A**: Skrip Ansible / Python cron job yang dijalankan secara ad-hoc melalui SSH.
* **Opsi B**: GitOps Declarative Controller yang diorkestrasi menggunakan **Temporal Workflow Engine**.

### 3. Keputusan
Memilih **Opsi B**:
1. **Single Source of Truth (Git)**: Seluruh status konfigurasi jaringan didefinisikan secara deklaratif (misal: intent YAML/JSON berbasis OpenConfig) di repositori Git terlacak.
2. **Orkestrasi Alur Kerja (Temporal Engine)**: Eksekusi perubahan, validasi pra/pasca-penerapan (*canary verification*), rollback instan, dan alur pemulihan mandiri dijalankan sebagai *Temporal Workflows* yang memiliki sifat *resilient, retryable*, dan *stateful*.
3. **Flapping Suppression Mechanism**: Menerapkan *hold-down timer* wajib (misal: 30 detik stabilitas link tanpa interupsi) sebelum failback lalu lintas ke jalur utama.

### 4. Konsekuensi
* **Positif**:
  * Riwayat perubahan konfigurasi 100% terlacak (*immutable audit trail*).
  * Pemulihan otomatis (*reconciliation loop*) mampu mengembalikan konfigurasi menyimpang dalam hitungan detik.
  * Meniadakan risiko terhentinya alur automasi di tengah jalan saat terjadi restart server orkestrator (keunggulan persistensi Temporal).
* **Negatif & Mitigasi**:
  * Kurva pembelajaran tim operasional terhadap paradigma GitOps dan Temporal SDK. Disediakan GUI portal konfigurasi untuk Network Engineer Tier 1-2.

---

## ADR-005: Korelasi Alarm dan Root Cause Analysis (AIOps)

* **Status**: Accepted
* **Tanggal**: 2026-10-09
* **Pengambil Keputusan**: AI/ML Engineer & NOC Operations Lead

### 1. Konteks & Masalah
Saat kabel transmisi utama atau switch core mengalami down, ratusan perangkat downstream memancarkan ribuan notifikasi kegagalan secara bersamaan (*alarm flood*), menyebabkan *alert fatigue* pada tim NOC Tier 1/2 dan memperlambat MTTR.

### 2. Pilihan yang Dipertimbangkan
* **Opsi A**: Rule-based Alert Filtering statis (berdasarkan regex atau tabel pemetaan manual).
* **Opsi B**: Graph Dependency Propagation Traversal dikombinasikan dengan Time-Series ML Anomaly Detection.

### 3. Keputusan
Memilih **Opsi B**:
1. **Graph Traversal RCA**: Saat alarm diterima, engine mengidentifikasi posisi simpul pada Graph Database. Jika simpul induk (*parent link*) berstatus *DOWN*, seluruh alarm dari simpul turunan (*children*) otomatis diagregasikan ke dalam satu tiket insiden utama (*suppressed/impacted*).
2. **Predictive Time-Series Anomaly Detection**: Menerapkan model deteksi anomali pada metrik tren (suhu optik laser, kenaikan jitter, buffer drop) sebelum terjadi *hard down*, memicu peringatan proaktif (*early warning*).

### 4. Konsekuensi
* **Positif**:
  * Mencapai target reduksi kebisingan alarm $\ge 85\%$.
  * Tiket insiden langsung menunjuk simpul sumber akar masalah (*root cause*), bukan korban turunan.
* **Negatif & Mitigasi**:
  * Ketergantungan pada akurasi topologi graf. Diminimalisasi melalui protokol *Auto-Discovery* berkelanjutan (LLDP/CDP/BGP) yang menyinkronkan graf secara real-time.

---

## ADR-006: Visualisasi Topologi Skala Besar Berbasis WebGL

* **Status**: Accepted
* **Tanggal**: 2026-10-09
* **Pengambil Keputusan**: Frontend Architect & UI/UX Lead

### 1. Konteks & Masalah
Jaringan enterprise memiliki puluhan ribu simpul (router, switch, server, VM) dan ratusan ribu tautan penghubung. Rendering berbasis DOM standar (HTML Divs) atau pure SVG mengalami penurunan FPS drastis (macet) ketika jumlah elemen melebihi 2.000 simpul.

### 2. Pilihan yang Dipertimbangkan
* **Opsi A**: Rendering graf berbasis SVG/HTML5 Canvas standar (D3.js / standard React Flow).
* **Opsi B**: Rendering graf terakselerasi perangkat keras (Hardware-Accelerated WebGL/WebGPU) menggunakan engine seperti PixiJS, Cytoscape dengan WebGL canvas, atau Sigma.js / Three.js di atas framework Next.js.

### 3. Keputusan
Memilih **Opsi B**:
* Membangun antarmuka visualisasi topologi menggunakan **Next.js** dipadu dengan kanvas **WebGL (PixiJS / Sigma.js / Cytoscape.js)**.
* Mengadopsi teknik *Level of Detail (LoD)* dan *viewport culling* (hanya merender simpul yang terlihat di layar, mengelompokkan simpul jauh ke dalam kluster/site bundel).

### 4. Konsekuensi
* **Positif**:
  * Panning, zooming, dan animasi real-time lancar pada 60 FPS bahkan dengan 50.000+ simpul dan tautan.
  * Waktu render dasbor memenuhi SLA non-fungsional (< 1 detik).
* **Negatif & Mitigasi**:
  * Kompleksitas interaksi UI (aksesibilitas dan tooltip kustom pada canvas WebGL). Diatasi dengan membuat overlay DOM tipis untuk elemen interaksi detail/kartu inspeksi simpul.

---

## ADR-007: Autentikasi Mandiri Berbasis Basis Data (Internal / Custom Auth)

* **Status**: Accepted
* **Tanggal**: 2026-10-09
* **Pengambil Keputusan**: Security Engineer, Backend Lead & System Architect

### 1. Konteks & Masalah
Sistem NMS mengelola infrastruktur jaringan vital yang sering beroperasi dalam lingkungan terisolasi (*air-gapped*) atau *on-premise enterprise data center* tanpa akses internet publik. Ketergantungan pada Identity Provider (IdP) cloud pihak ketiga seperti Okta atau Azure AD:
* Menimbulkan risiko *single point of failure* (SPOF) saat link internet luar padam.
* Menciptakan biaya lisensi per-pengguna yang tinggi.
* Melanggar kebijakan kepatuhan kedaulatan data pada jaringan tertutup (*air-gapped compliance*).

Di sisi lain, mengoperasikan cluster SSO terpisah yang berat (seperti Keycloak) memperkenalkan kompleksitas devops ekstra dan overhead memori JVM yang tidak sebanding untuk sistem kontrol jaringan internal.

### 2. Pilihan yang Dipertimbangkan
* **Opsi A**: Enterprise Cloud IdP (Okta / Azure AD / Auth0).
* **Opsi B**: Dedicated Heavyweight SSO Server (Keycloak cluster).
* **Opsi C**: Autentikasi Mandiri Berbasis Basis Data Internal (**PostgreSQL 16+** + **Argon2id** + **JWT Stateless** + **Redis Session Blacklist** + **TOTP RFC 6238**).

### 3. Keputusan
Memilih **Opsi C**: Mengimplementasikan **Autentikasi Mandiri Berbasis Basis Data (Internal / Custom Auth)**:
1. **Penyimpanan Kredensial Mandiri**: Menyimpan kredensial pengguna langsung di database internal PostgreSQL dengan hashing password **Argon2id** (tahan serangan GPU/ASIC).
2. **Manajemen Sesi & Token**:
   * Token akses stateless menggunakan **JWT** dengan masa aktif singkat (15 menit).
   * *Rotating Refresh Token* (7 hari) yang divalidasi silang terhadap *session revocation blacklist* di **Redis** untuk memungkinkan terminasi sesi seketika (*instant force logout*).
3. **MFA Mandiri (TOTP)**: Mendukung autentikasi 2 faktor berbasis **TOTP (RFC 6238)** dengan secret terenkripsi AES-256-GCM tanpa perlu gateway SMS/email pihak ketiga.
4. **Proteksi Akses**: Menerapkan *rate limiting* dan pencegahan brute-force (*progressive account lock*) di level API Gateway via Redis.

### 4. Konsekuensi
* **Positif**:
  * *100% Self-Contained*: Berjalan mulus pada jaringan terisolasi (*air-gapped*) tanpa memerlukan koneksi ke internet publik.
  * Zero External License Cost: Bebas biaya lisensi IdP komersial.
  * Latensi Login Rendah: Verifikasi autentikasi selesai dalam waktu $< 50$ milidetik tanpa *network round-trip* ke server SSO eksternal.
  * Arsitektur Ringan: Menggunakan PostgreSQL dan Redis yang sudah menjadi dependensi utama sistem.
* **Negatif & Mitigasi**:
  * Tanggung jawab pengelolaan siklus hidup keamanan berada di tangan tim internal: Dimitigasi dengan menggunakan pustaka kriptografi resmi yang teruji (`golang.org/x/crypto/argon2`, `pquerna/otp`, `golang-jwt/jwt/v5`), enforcing password policy ketat, dan audit berkala.
