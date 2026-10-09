# AGENTS.md — AI Agent Operating Guidelines & Rules
## Project: Next-Generation Network Management System (NMS 2026)

Dokumen ini adalah **pedoman operasional dan aturan mutlak (*Rules & Constraints*)** bagi seluruh AI Agent maupun pengembang otomatis yang bekerja di repositori ini.

---

## 1. Hirarki Dokumen & Doktrin Repositori

Sebelum melakukan analisis, perancangan, atau penulisan kode, setiap agent **wajib** memahami hirarki kebenaran (*hierarchy of truth*) berikut:

1. [prd.md](file:///home/rizxenzs/Desktop/nms/prd.md) — **Product Requirements Document**: Mendefinisikan visi bisnis, kebutuhan fungsional (Melihat, Mengatur, Mengingatkan), personas, dan metrik keberhasilan (MTTD < 3 detik, MTTR < 15 menit, reduksi alarm $\ge 85\%$).
2. [adr.md](file:///home/rizxenzs/Desktop/nms/adr.md) — **Architecture Decision Records**: Mendefinisikan keputusan arsitektural yang mengikat (Dual-Storage, Tiered Ingestion, Kafka backbone, Temporal GitOps, WebGL Canvas).
3. [TECH_STACK.md](file:///home/rizxenzs/Desktop/nms/TECH_STACK.md) — **Single Source of Truth**: Mendefinisikan bahasa resmi, versi runtime, pustaka yang diizinkan, dan struktur monorepo.
4. **AGENTS.md (Dokumen Ini)** — **Rules of Engagement**: Tata tertib, batasan teknis, dan standar kualitas eksekusi.

---

## 2. Aturan Mutlak (*Non-Negotiable Rules*)

1. **Dilarang Menambah Library / Framework Tanpa Izin**:
   * Gunakan hanya pustaka yang telah dideklarasikan di [TECH_STACK.md](file:///home/rizxenzs/Desktop/nms/TECH_STACK.md).
   * Dilarang mengganti framework (misal: dilarang mengganti Next.js ke SPA polos, atau mengganti VictoriaMetrics/Memgraph ke database lain) tanpa membuat ADR resmi.
2. **Kualitas Visual & UI Kelas Enterprise (*No Cheap/Basic UI*)**:
   * Antarmuka web harus terlihat memukau, modern, dan sangat profesional (*dark-mode first, enterprise dashboard*).
   * Dilarang menggunakan placeholder murah, teks tiruan tanpa konteks ("Lorem ipsum"), atau layout rusak.
   * Visualisasi graf wajib responsif dan mempertahankan performa **60 FPS** saat memanipulasi simpul.
3. **Type-Safety Tanpa Kompromi**:
   * TypeScript: Wajib `strict: true`. **Dilarang keras menggunakan tipe `any`**. Gunakan `unknown` dengan type guards atau tipe domain spesifik.
   * Go: Hindari `interface{}` kosong kecuali pada serialization serializer payload generic; gunakan struct bertipe kuat dengan validasi model OpenConfig.
4. **Resilience & Zero Single-Point-of-Failure (SPOF)**:
   * Setiap service Go/Python harus mengimplementasikan *graceful shutdown* (`os.Interrupt`, `syscall.SIGTERM`).
   * Seluruh pembacaan data streaming wajib memiliki mekanisme *reconnection exponential backoff* dan penanganan *backpressure*.
5. **Keamanan & Kepatuhan**:
   * Seluruh API internal wajib mengasumsikan komunikasi terenkripsi TLS 1.3 (mTLS).
   * Autentikasi dikelola secara mandiri (*Custom Database-backed Auth*) menggunakan PostgreSQL + Argon2id + JWT stateless + TOTP MFA mandiri tanpa bergantung pada IdP eksternal (Okta/Keycloak/Azure AD).
   * Seluruh aksi mutasi konfigurasi jaringan wajib melewati mekanisme otorisasi RBAC granular dan dicatat dalam *immutable audit log*.

---

## 3. Peran & Domain Spesialisasi Agent

Saat menangani tugas spesifik, Agent harus bertindak sesuai domain berikut:

### 3.1. Telemetry & Ingestion Agent (`services/collector-*`)
* **Bahasa**: Go 1.23+
* **Fokus**:
  * Mengonsumsi streaming gNMI menggunakan `openconfig/gnmic`.
  * Mengatur polling adaptif SNMP v3 (`gosnmp`) dan listener Trap/Flow (`goflow2`).
  * Mengirim data terkompresi ke topik Apache Kafka (`telemetry.metrics.raw`).
* **Kriteria Keberhasilan**: Throughput tinggi, alokasi memori minimal, *zero goroutine leaks*, penanganan timeout koneksi perangkat jaringan yang elegan.

### 3.2. Topology & Graph Agent (`services/topology-engine`)
* **Bahasa**: Go 1.23+ & Cypher (Memgraph / Neo4j)
* **Fokus**:
  * Sinkronisasi data topologi fisik dan logis dari LLDP/CDP/BGP peering ke dalam bentuk simpul (*nodes*) dan relasi (*edges*).
  * Menjaga relasi hierarki induk-anak (*parent-child dependency*) agar selalu mutakhir.
* **Kriteria Keberhasilan**: Kueri traversal penelusuran simpul terdampak (*blast radius*) selesai dalam waktu $< 10$ milidetik.

### 3.3. AIOps & Correlation Agent (`services/aiops-correlator` & `anomaly-detector`)
* **Bahasa**: Go (Graph RCA Engine) & Python 3.12 (Time-Series Anomaly Detector)
* **Fokus**:
  * Mengisolasi simpul biang kerok (*root cause*) dan meredam $\ge 85\%$ alarm turunan (*secondary/impacted alerts*).
  * Menghitung skor anomali deret waktu (suhu optik, fluktuasi jitter, penurunan daya laser) sebelum terjadi pemutusan link fisik.
* **Kriteria Keberhasilan**: Mengeliminasi kepanikan notifikasi bagi tim NOC Tier 1/2.

### 3.4. GitOps & Self-Healing Agent (`services/gitops-reconciler`)
* **Bahasa**: Go 1.23+ & Temporal Workflow SDK
* **Fokus**:
  * Membandingkan *running config* pada perangkat dengan *target state* deklaratif di Git.
  * Menerapkan *reconciliation loop* otomatis untuk rollback konfigurasi yang menyimpang (*drift*).
  * Menjalankan logika **stability hold-down timer** (peredam flapping): menahan failback ke jalur utama selama interval kestabilan terbukti aman.
* **Kriteria Keberhasilan**: Transaksional, idempotent, tahan terhadap gangguan jaringan saat eksekusi perubahan.

### 3.5. Frontend & Visualization Agent (`apps/web`)
* **Bahasa**: Next.js 15+ (App Router), TypeScript, WebGL (PixiJS v8 / Sigma.js v3)
* **Fokus**:
  * Membangun kanvas topologi jaringan interaktif dengan akselerasi perangkat keras GPU.
  * Menerapkan teknik *Level of Detail (LoD)* dan *viewport culling* untuk merender puluhan ribu simpul tanpa *lag*.
  * Dasbor metrik real-time menggunakan uPlot, WebSocket push updates, dan kartu inspeksi insiden interaktif.
* **Kriteria Keberhasilan**: Tampilan ultra-premium, 60 FPS saat pan/zoom, waktu pemuatan dasbor $< 1$ detik.

---

## 4. Standar Kode & Praktik Terbaik

### 4.1. Bahasa Go (Backend Services)
* Gunakan package `context.Context` pada seluruh fungsi I/O dan kueri database untuk mendukung pembatalan (*cancellation propagation*).
* Bungkus error dengan konteks menggunakan `fmt.Errorf("action description: %w", err)`.
* Hindari *magic strings* untuk nama topik Kafka, jenis alarm, dan status perangkat; gunakan tipe enumerasi bertipe kuat (`type DeviceStatus string`).

### 4.2. Bahasa TypeScript / React (Frontend)
* Pisahkan komponen Presentasional (*pure UI*) dari komponen Logika Data (*container / hooks*).
* Gunakan `@tanstack/react-query` untuk seluruh interaksi data server (caching, invalidasi, polling berkala jika diperlukan).
* Gunakan `zustand` untuk *client state* yang membutuhkan akses global (misal: simpul yang sedang dipilih di kanvas topologi, filter aktif).

### 4.3. Skema Data & Protokol
* Seluruh model payload telemetri dan kontrol jaringan harus bersandar pada model data **OpenConfig YANG** atau schema Protobuf yang telah divalidasi.

---

## 5. Protokol Pengujian & Validasi

Setiap kode yang dihasilkan harus memenuhi kriteria validasi:
1. **Mock Telemetry & Topology Generator**: Saat menguji tanpa perangkat fisik, sediakan generator data telemetri realistis multi-vendor (Cisco IOS-XE, ArubaOS-CX, FortiOS) yang menghasilkan metrik metrik hidup, alarm berkala, dan simulasi pemutusan kabel (*link failure*).
2. **Uji Beban & Flapping**: Validasi bahwa logika *stability timer* benar-benar mereset hitungan saat link terputus di tengah periode observasi kestabilan.
3. **Uji Reduksi Alarm**: Pastikan bahwa saat node induk dimatikan, sistem menerbitkan **1 tiket insiden akar masalah** dan menandai simpul turunan sebagai **terdampak**, bukan melontarkan ratusan alert terpisah.

---

## 6. Alur Kerja Rekayasa Berpasangan (*Pair-Programming Workflow*)

1. **Jelaskan Niat & Langkah Singkat**: Sebelum menulis perubahan kode besar, jelaskan rencana implementasi secara ringkas kepada pengguna.
2. **Tautkan File Secara Eksplisit**: Selalu sertakan tautan berkas bergaya markdown GitHub (contoh: [TECH_STACK.md](file:///home/rizxenzs/Desktop/nms/TECH_STACK.md)) saat mereferensikan file.
3. **Fokus pada Solusi Solid**: Jangan buat prototipe setengah jadi atau rapuh. Setiap komponen yang dibuat harus memiliki arsitektur yang siap diintegrasikan ke level produksi.
