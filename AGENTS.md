# AGENTS.md — AI Agent Operating Guidelines & Rules

## Project: Next-Generation Network Management System (NMS 2026)

Dokumen ini adalah **pedoman operasional dan aturan mutlak (*Rules & Constraints*)** bagi seluruh AI Agent maupun pengembang otomatis yang bekerja di repositori ini.

---

## 1. Hirarki Dokumen & Doktrin Repositori

Sebelum melakukan analisis, perancangan, atau penulisan kode, setiap agen **wajib** memahami hierarki kebenaran (*hierarchy of truth*) berikut:

1. prd.md — **Product Requirements Document**: Mendefinisikan visi bisnis, kebutuhan fungsional (Melihat, Mengatur, Mengingatkan), personas, dan metrik keberhasilan (MTTD < 3 detik, MTTR < 15 menit, reduksi alarm $\ge 85\%$)[cite: 8, 9].
2. adr.md — **Architecture Decision Records**: Mendefinisikan keputusan arsitektural yang mengikat (Dual-Storage, Tiered Ingestion, Kafka backbone, Temporal GitOps, WebGL Canvas, standarisasi Rust)[cite: 7, 8].
3. TECH_STACK.md — **Single Source of Truth**: Mendefinisikan bahasa resmi, versi runtime, pustaka yang diizinkan, dan struktur Cargo Workspace monorepo[cite: 8, 10].
4. **AGENTS.md (Dokumen Ini)** — **Rules of Engagement**: Tata tertib, batasan teknis, dan standar kualitas eksekusi.



---

## 2. Aturan Mutlak (*Non-Negotiable Rules*)

1. **Dilarang Menambah Library / Crate Tanpa Izin**:
* Gunakan hanya pustaka yang telah dideklarasikan di TECH_STACK.md.


* Dilarang mengganti framework (misal: dilarang mengganti Next.js ke SPA polos, atau mengganti VictoriaMetrics/Memgraph ke database lain) tanpa membuat ADR resmi.




2. **Kualitas Visual & UI Kelas Enterprise (*No Cheap/Basic UI*)**:
* Antarmuka web harus terlihat memukau, modern, dan sangat profesional (*dark-mode first, enterprise dashboard*).


* Dilarang menggunakan placeholder murah, teks tiruan tanpa konteks ("Lorem ipsum"), atau tata letak rusak.


* Visualisasi graf wajib responsif dan mempertahankan performa **60 FPS** saat memanipulasi simpul.




3. **Type-Safety Tanpa Kompromi & Zero Unsafe**:
* TypeScript: Wajib `strict: true`. **Dilarang keras menggunakan tipe `any**`; gunakan `unknown` dengan type guards atau tipe domain spesifik.


* Rust: Dilarang keras menggunakan blok `unsafe` kecuali untuk isolasi probe kernel eBPF tingkat rendah. Dilarang memanggil `.unwrap()` atau `.expect()` pada kode produksi; gunakan propagasi galat `Result<T, E>` dengan operator `?` dan tipe galat domain eksplisit.


4. **Resilience & Zero Single-Point-of-Failure (SPOF)**:
* Setiap service Rust/Python wajib mengimplementasikan penghentian anggun (*graceful shutdown*) menggunakan `tokio::signal` dan penanganan sinyal sistem operasi.


* Seluruh pembacaan data streaming wajib memiliki mekanisme penyambungan ulang bertahap (*reconnection exponential backoff*) dan penanganan tekanan balik (*backpressure*).




5. **Keamanan & Kepatuhan**:
* Seluruh API internal wajib mengasumsikan komunikasi terenkripsi TLS 1.3 (mTLS).


* Autentikasi dikelola secara mandiri (*Custom Database-backed Auth*) menggunakan PostgreSQL + Argon2id (`argon2`) + JWT stateless (`jsonwebtoken`) + TOTP MFA mandiri (`totp-rs`) tanpa bergantung pada IdP eksternal[cite: 7, 8].
* Seluruh aksi mutasi konfigurasi jaringan wajib melewati mekanisme otorisasi RBAC granular dan dicatat dalam log audit tak terubah (*immutable audit log*).





---

## 3. Peran & Domain Spesialisasi Agent

Saat menangani tugas spesifik, Agen harus bertindak sesuai domain berikut:

### 3.1. Telemetry & Ingestion Agent (`services/collector-*`)

* **Bahasa**: Rust 2024 / 1.82+ (Cargo Workspace crate)[cite: 7].
* **Fokus**:
* Mengonsumsi streaming gNMI menggunakan klien gRPC asinkron berbasis `tonic` dan skema `prost`[cite: 7].
* Mengatur polling adaptif SNMP v3 (`rasn-snmp`) dan listener Trap/Flow (`netflow-parser`, zero-copy parser `nom`)[cite: 7].
* Menjalankan probe kernel eBPF menggunakan framework Rust murni `aya`[cite: 7].
* Mengirim data terkompresi ke topik Apache Kafka (`telemetry.metrics.raw`) via `rdkafka`[cite: 7, 8].


* **Kriteria Keberhasilan**: Throughput $\ge 1.000.000$ metrik/detik, alokasi memori minimal (*zero-copy parsing*), *zero task leaks* pada runtime Tokio, dan penanganan batas waktu koneksi perangkat jaringan yang elegan[cite: 7, 8].

### 3.2. Topology & Graph Agent (`services/topology-engine`)

* **Bahasa**: Rust 2024 / 1.82+ & Cypher (Memgraph / Neo4j via `neo4rs`)[cite: 7, 8].
* **Fokus**:
* Sinkronisasi data topologi fisik dan logis dari LLDP/CDP/BGP peering ke dalam bentuk simpul (*nodes*) dan relasi (*edges*).


* Menjaga relasi hierarki induk-anak (*parent-child dependency*) agar selalu mutakhir.




* **Kriteria Keberhasilan**: Kueri traversal penelusuran simpul terdampak (*blast radius*) selesai dalam waktu $< 10$ milidetik tanpa interupsi siklus pembersihan memori[cite: 7, 8].

### 3.3. AIOps & Correlation Agent (`services/aiops-correlator` & `anomaly-detector`)

* **Bahasa**: Rust (Graph RCA Engine via `neo4rs`) & Python 3.12 (Time-Series Anomaly Detector)[cite: 7, 8].
* **Fokus**:
* Mengisolasi simpul biang kerok (*root cause*) dan meredam $\ge 85\%$ alarm turunan (*secondary/impacted alerts*).


* Menghitung skor anomali deret waktu (suhu optik, fluktuasi jitter, penurunan daya laser) sebelum terjadi pemutusan link fisik.




* **Kriteria Keberhasilan**: Mengeliminasi kepanikan notifikasi bagi tim NOC Tier 1/2.



### 3.4. GitOps & Self-Healing Agent (`services/gitops-reconciler`)

* **Bahasa**: Rust 2024 / 1.82+ & Rust Temporal SDK Core / Worker[cite: 7].
* **Fokus**:
* Membandingkan *running config* pada perangkat dengan *target state* deklaratif di Git menggunakan pustaka Git murni `gix`[cite: 7, 8].
* Menerapkan *reconciliation loop* otomatis untuk membatalkan konfigurasi yang menyimpang (*drift*).


* Menjalankan logika **stability hold-down timer** (peredam flapping): menahan failback ke jalur utama selama interval kestabilan terbukti aman menggunakan timer asinkron Tokio[cite: 7, 8].


* **Kriteria Keberhasilan**: Transaksional, idempoten, dan tahan terhadap gangguan jaringan saat eksekusi perubahan.



### 3.5. Frontend & Visualization Agent (`apps/web`)

* **Bahasa**: Next.js 15+ (App Router), TypeScript, WebGL (PixiJS v8 / Sigma.js v3).


* **Fokus**:
* Membangun kanvas topologi jaringan interaktif dengan akselerasi perangkat keras GPU.


* Menerapkan teknik *Level of Detail (LoD)* dan penapisan area pandang (*viewport culling*) untuk merender puluhan ribu simpul tanpa *lag*.


* Dasbor metrik real-time menggunakan uPlot, WebSocket push updates, dan kartu inspeksi insiden interaktif.




* **Kriteria Keberhasilan**: Tampilan enterprise modern, 60 FPS saat pan/zoom, waktu pemuatan dasbor $< 1$ detik.



---

## 4. Standar Kode & Praktik Terbaik

### 4.1. Bahasa Rust (Backend Services & Engines)

* Gunakan runtime multi-thread `tokio` untuk seluruh penanganan operasi asinkron I/O[cite: 7].
* Terapkan propagasi pembatalan tugas secara bersih menggunakan `tokio_util::sync::CancellationToken` alih-alih membiarkan tugas asinkron berjalan tanpa batas waktu[cite: 7].
* Kelola galat secara bertingkat: gunakan `thiserror` untuk mendefinisikan enum galat spesifik pada level modul domain/crate internal, dan gunakan `anyhow` hanya pada lapisan biner aplikasi teratas[cite: 7].
* Terapkan observabilitas sistem menggunakan `tracing` (`tracing::info!`, `tracing::error!`, `#[tracing::instrument]`) terstruktur dalam format JSON; hindari makro cetak standar `println!`.
* Minimalkan alokasi memori berlebih (*zero-copy data path*): manfaatkan irisan penyangga `&[u8]`, referensi pointer string `&str`, atau `bytes::Bytes` saat melakukan ingesti payload jaringan berkecepatan tinggi[cite: 7].
* Dilarang menggunakan *magic strings* untuk nama topik Kafka, jenis alarm, dan status perangkat; wajib mendefinisikan enum bertipe kuat dengan trait `Serialize`, `Deserialize`, `PartialEq`, dan `Eq`[cite: 7, 8].
* Lakukan pemeriksaan sintaks dan skema SQL pada waktu kompilasi menggunakan makro `sqlx::query!` atau `sqlx::query_as!`[cite: 7].

### 4.2. Bahasa TypeScript / React (Frontend)

* Pisahkan komponen presentasi (*pure UI*) dari komponen logika data (*container / hooks*).


* Gunakan `@tanstack/react-query` untuk seluruh interaksi data server (caching, invalidasi, polling berkala jika diperlukan).


* Gunakan `zustand` untuk state klien yang membutuhkan akses global (misal: simpul yang sedang dipilih di kanvas topologi, filter aktif).



### 4.3. Skema Data & Protokol

* Seluruh model payload telemetri dan kontrol jaringan wajib mengacu pada definisi Protobuf atau model OpenConfig YANG yang dikompilasi ke struktur Rust melalui `prost-build` dan `tonic-build`[cite: 7, 8].

---

## 5. Protokol Pengujian & Validasi

Setiap kode yang dihasilkan harus memenuhi kriteria validasi berikut:

1. **Mock Telemetry & Topology Generator**: Saat menguji tanpa perangkat fisik, sediakan generator data telemetri realistis multi-vendor (Cisco IOS-XE, ArubaOS-CX, FortiOS) yang menghasilkan metrik metrik hidup, alarm berkala, dan simulasi pemutusan kabel (*link failure*).


2. **Uji Beban & Flapping**: Validasi bahwa logika *stability timer* benar-benar mereset hitungan saat link terputus di tengah periode observasi kestabilan.


3. **Uji Reduksi Alarm**: Pastikan bahwa saat node induk dimatikan, sistem menerbitkan **1 tiket insiden akar masalah** dan menandai simpul turunan sebagai **terdampak**, bukan melontarkan ratusan alert terpisah.


4. **Verifikasi Kompilasi & Clippy**: Seluruh kode Rust wajib lolos kompilasi `cargo check` dan `cargo clippy --all-targets -- -D warnings` tanpa peringatan galat.

---

## 6. Alur Kerja Rekayasa Berpasangan (*Pair-Programming Workflow*)

1. **Jelaskan Niat & Langkah Singkat**: Sebelum menulis perubahan kode besar, jelaskan rencana implementasi secara ringkas kepada pengguna.


2. **Tautkan File Secara Eksplisit**: Selalu sertakan tautan berkas bergaya markdown GitHub (contoh: TECH_STACK.md) saat mereferensikan file.


3. **Fokus pada Solusi Solid**: Jangan buat prototipe setengah jadi atau rapuh; setiap komponen yang dibuat harus memiliki arsitektur yang siap diintegrasikan ke level produksi.