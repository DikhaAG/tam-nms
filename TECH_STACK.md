# Source of Truth: Technology Stack & Engineering Standards

## Next-Generation Network Management System (NMS 2026)

Dokumen ini adalah **panduan mutlak (*Single Source of Truth*)** untuk seluruh pemilihan bahasa pemrograman, framework, pustaka (*crates*), basis data, protokol, dan standar dependensi yang digunakan dalam pengembangan proyek **Next-Gen NMS**.

Seluruh implementasi modul dan fitur selanjutnya **wajib** merujuk pada standar yang tertera di dokumen ini untuk mencegah fragmentasi teknologi.

---

## 1. Arsitektur Makro Sistem

Sistem mengadopsi pola **Event-Driven Modular Monorepo / Microservices** berbasis ekosistem **Rust** yang terbagi menjadi 5 lapisan utama:

```
[ Layer 1: Edge & Telemetry Collectors ] (Rust, gNMI, SNMP, eBPF)
                      │ (Streaming sub-detik)
                      ▼
[ Layer 2: Message Streaming & Buffer ] (Apache Kafka + Vector)
                      │
        ┌─────────────┴─────────────┐
        ▼                           ▼
[ Layer 3A: Storage Engines ]   [ Layer 3B: Processing Engines ]
 - Time-Series (VictoriaMetrics) - Topology & RCA Engine (Rust)
 - Graph DB (Memgraph/Neo4j)     - AIOps Anomaly Detection (Python)
 - Relational/Meta (PostgreSQL)  - GitOps Reconciler (Rust + Temporal)
        └─────────────┬─────────────┘
                      ▼
[ Layer 4: Application Core & API Gateway ] (Rust Axum BFF, gRPC, WebSocket)
                      ▼
[ Layer 5: Web UI & Visualization ] (Next.js 16, TypeScript, WebGL/PixiJS)

```

---

## 2. Bahasa Pemrograman & Runtime

| Domain | Bahasa & Versi | Runtime / Toolchain | Alasan Pemilihan |
| --- | --- | --- | --- |
| **Collector & High-Throughput Services** | **Rust** `2024 Edition / 1.82+`<br> | Official Rust Toolchain (`cargo`, `rustc`, `clippy`) | Bebas *Garbage Collection pause*, determinisme latensi sub-milidetik, keamanan memori tingkat sistem, dan *zero-cost abstractions*.

 |
| **Kernel-Level Observability** | **Rust** (eBPF)

 | Framework `aya` / LLVM | Penulisan dan pemuatan probe kernel Linux berbasis Rust murni tanpa dependensi compiler C eksternal saat runtime.

 |
| **AIOps & Machine Learning** | **Python** `3.12+`<br> | UV / Poetry / CPython

 | Ekosistem data science, deteksi anomali deret waktu (*time-series anomaly detection*), dan model dependensi graf.

 |
| **Frontend & Web UI** | **TypeScript** `5.5+`<br> | Node.js `20 LTS` / Bun

 | Type-safety menyeluruh, integrasi kanvas graf WebGL, modern component lifecycle.

 |

---

## 3. Komponen Per Lapisan & Pustaka Resmi

### 3.1. Layer 1: Edge Ingestion & Telemetry Collectors (Rust)

* **Async Runtime & Core**: `tokio` (multi-thread engine), `tracing` (structured logging/telemetry), `bytes` (zero-copy memory management).


* **Protokol Telemetri Modern (gNMI/gRPC)**:
* `tonic`: Framework gRPC asinkron berkecepatan tinggi berbasis HTTP/2.


* `prost` & `prost-build`: Kompilator dan runtime skema Protobuf untuk validasi model OpenConfig YANG ke dalam struct Rust bertipe kuat.




* **Protokol Warisan (SNMP & Flow)**:
* `rasn-snmp`: Implementasi pustaka ASN.1 / BER parsing untuk klien SNMP v2c/v3.


* `netflow-parser`: Parser paket telemetri NetFlow v5/v9 dan IPFIX.


* `nom`: Zero-copy combinator parser untuk parsing biner sFlow dan syslog mentah dengan alokasi memori minimal.




* **Sensor Kernel**:
* `aya`: Pustaka Rust murni untuk memuat dan mengelola program eBPF di Linux kernel.




* **Edge Aggregator & Shipper**:
* **Vector**: Ditulis dalam Rust, bertindak sebagai *sidecar collector/shipper* untuk parsing syslog dan pengiriman batch terkompresi ke Kafka.





### 3.2. Layer 2: Message Streaming & Event Buffer

* **Message Broker Utama**: **Apache Kafka** `v3.7+` (Mode KRaft - tanpa ZooKeeper).


* *Client Rust*: `rdkafka` (klien asinkron berbasis librdkafka teroptimasi untuk performa throughput tinggi).




* **Topic Partitioning Strategy**:
* `telemetry.metrics.raw`: Key = `device_id` (menjamin urutan metrik per perangkat).


* `telemetry.events.alarms`: Key = `site_id` atau `device_id`.


* `telemetry.topology.changes`: Key = `topology_domain`.





### 3.3. Layer 3: Database & Penyimpanan (*Dual-Storage Strategy*)

#### A. Database Deret Waktu (*Time-Series*)

* **Mesin**: **VictoriaMetrics** atau **ClickHouse** `v24+`.


* *Fungsi*: Ingesti $\ge 1.000.000$ poin metrik/detik, kompresi ZSTD 5–10x, retensi data historis hingga 30+ hari.


* *Rust Driver*: `clickhouse` crate (asinkron berbasis HTTP protokol dengan kompresi LZ4/ZSTD) atau `reqwest` streaming line protocol.





#### B. Database Graf (*Topology & Dependency*)

* **Mesin**: **Memgraph** atau **Neo4j** (Enterprise / Community).


* *Fungsi*: Menyimpan simpul perangkat (*nodes*), antarmuka (*ports*), tautan fisik/logis (*edges*), relasi dependensi, dan rute BGP.


* *Query Language*: Cypher (OpenCypher).


* *Rust Driver*: `neo4rs` (klien Bolt Protocol asinkron murni Rust).





#### C. Database Relasional, Kredensial & Metadata

* **Mesin**: **PostgreSQL** `16+`.


* *Fungsi*: Manajemen akun pengguna & autentikasi mandiri (kredensial hash Argon2id, secret key TOTP terenkripsi, status aktif akun), data katalog inventaris perangkat, definisi peran granular RBAC, metadata konfigurasi GitOps, dan audit trail log.


* *Database Toolchain*: `sqlx` (Postgres async driver dengan validasi sintaks SQL dan skema statis saat kompilasi).





#### D. In-Memory Cache & Ephemeral State

* **Mesin**: **Redis** `7.2+` (Cluster) / **Valkey**.


* *Fungsi*: Penyimpanan live heartbeat (*status liveness* perangkat sub-detik), distributed lock untuk peredam flapping, rate-limiting API (termasuk proteksi brute-force login), serta cache pembatalan sesi (*token revocation / session blacklist*).


* *Rust Driver*: `fred` (klien Redis asinkron berkinerja tinggi dengan koneksi pipelining dan multiplexing).





---

### 3.4. Layer 4: Orchestration, GitOps & AIOps

#### A. Workflow Engine & Pemulihan Mandiri (Self-Healing)

* **Engine**: **Temporal.io** (`v1.24+`).


* *Fungsi*: Menjalankan workflow pemulihan mandiri yang tahan uji (*fault-tolerant*), otomasi failover jalur, dan countdown stabilitas link (*hold-down timer*).


* *SDK*: `temporal-sdk-core` dan `temporal-sdk` (Rust SDK resmi).





#### B. Rekonsiliasi Konfigurasi Berbasis GitOps

* **Git Engine**: `gix` (Pure Rust Git implementation) atau `git2` (libgit2 binding thread-safe).


* **Format Konfigurasi Niat (*Intent*)**: Schema berbasis YAML / JSON yang divalidasi terhadap model OpenConfig YANG via `serde_yaml` dan `serde_json`.



#### C. AIOps & Deteksi Anomali Deret Waktu (Python Service)

* **Framework Service**: **FastAPI** + **Uvicorn** (asynchronous gRPC/REST service).


* **Machine Learning & Time-Series**:
* `numpy`, `pandas`, `scipy`

* `scikit-learn`: Isolasi anomali clustering.


* `torch` / `darts`: Model prediksi tren kenaikan temperatur optik & saturasi buffer sebelum kegagalan terjadi.


* `networkx`: Analisis topologi graf lokal untuk penentuan jalur kegagalan tercepat.





---

### 3.5. Layer 5: Frontend & Visualisasi Web

#### A. Core Web Framework

* **Framework**: **Next.js** `16+` (App Router, React 19).
* **Bahasa**: **TypeScript** (Strict Mode diaktifkan).


* **Bundler & Tooling**: Turbopack, ESLint, Prettier.



#### B. State Management & Data Fetching

* **Server State**: `@tanstack/react-query` (v5) untuk caching kueri metrik REST/GraphQL.


* **Client / UI State**: `zustand` (State ringan untuk kontrol tampilan dasbor, seleksi simpul graf, filter).


* **Komunikasi Realtime**:
* WebSocket murni ke API Gateway Rust (`tokio-tungstenite`) untuk push event alarm dan live status.


* Reconnection handling otomatis dengan *exponential backoff*.





#### C. Styling & UI Components

* **Styling**: **Vanilla CSS / CSS Modules** dengan *Design Token System* (Dark-mode primary: Slate/Zinc, Neon status indicators: Emerald untuk Up, Rose untuk Down, Amber untuk Warning).


* **Headless UI Primitives**: `@radix-ui/react-*` (Dialog, Dropdown, Tooltip, Tabs).


* **Ikonografi**: `lucide-react`.



#### D. Visualisasi Topologi Graf Skala Besar (Hardware-Accelerated WebGL)

* **Rendering Engine Graf**:
* **PixiJS** `v8` (WebGL/WebGPU 2D rendering engine) ATAU **Sigma.js** `v3` / **Cytoscape.js** dengan WebGL canvas backend.


* *Target*: Mampu merender $50.000+$ simpul & kabel tautan pada **60 FPS** tanpa freeze.


* *Fitur Wajib*:
* *Level of Detail (LoD)*: Detail port hanya muncul saat zoom in; level cluster/site saat zoom out.


* *Viewport Culling*: Hanya merender simpul yang berada di dalam viewport layar.






* **Visualisasi Grafik Metrik Time-Series**:
* **uPlot** atau **ECharts** (uPlot direkomendasikan untuk jutaan titik metrik grafik garis karena latensi render sub-10ms).





---

## 4. Keamanan, Autentikasi & Tata Kelola

* **Autentikasi Mandiri Berbasis Basis Data (Internal / Custom Auth)**:
* **Penyimpanan Kredensial Internal**: Seluruh identitas akun, kredensial, dan hak akses dikelola langsung di database **PostgreSQL 16+**, menghilangkan ketergantungan pada Identity Provider pihak ketiga (seperti Okta, Keycloak, atau Azure AD) sehingga cocok untuk lingkungan on-premise maupun air-gapped.


* **Algoritma Hashing Password**: **Argon2id** via crate `argon2` dengan salt acak kriptografis 16-byte dan parameter memori tahan serangan brute-force GPU/ASIC (Memory: 64MB, Iterations: 3, Parallelism: 2).


* **Arsitektur Token & Manajemen Sesi**:
* Akses API stateless menggunakan **JWT** via crate `jsonwebtoken` bertanda tangan HMAC-SHA256 / Ed25519 dengan masa berlaku singkat (*short-lived access token*, 15 menit).


* Mekanisme *Rotating Refresh Token* (7 hari) yang disimpan dalam database PostgreSQL dengan session hash unik, dipadukan dengan blacklist/revocation cache di Redis via `fred` untuk mendukung terminasi sesi seketika (*instant force logout*).




* **Multi-Factor Authentication (MFA / 2FA) Mandiri**:
* Mengimplementasikan **TOTP** (RFC 6238) menggunakan crate `totp-rs` yang kompatibel dengan aplikasi authenticator umum (Google Authenticator, Microsoft Authenticator, Aegis).


* Rahasia (*secret key*) TOTP disimpan dalam PostgreSQL dalam keadaan terenkripsi menggunakan crate `aes-gcm` (**AES-256-GCM**).




* **Proteksi Brute-Force & Rate Limiting**:
* Rate limiting ketat pada rute `/api/v1/auth/login` menggunakan Redis *sliding-window algorithm* (maksimal 5 percobaan gagal per IP/Username dalam 5 menit).


* Penguncian akun progresif (*progressive account lockout*) dan pencatatan otomatis ke immutable audit log.






* **Otorisasi & RBAC**:
* Granular RBAC hingga level antarmuka/perangkat menggunakan evaluasi kebijakan terdistribusi di middleware Rust `axum`.




* **Enkripsi**:
* *In-Transit*: **TLS 1.3** wajib di seluruh komunikasi internal (mTLS antar microservices) dan eksternal.


* *At-Rest*: **AES-256-GCM** pada level disk/database.




* **Audit Trail**:
* Log audit bersifat *immutable* (hanya append) yang disegel dengan rantai hash kriptografis (*cryptographic SHA-256 block chain*).





---

## 5. Standar Struktur Repositori (Cargo Workspace Layout)

```
/home/rizxenzs/Desktop/nms/
├── Cargo.toml                      # Workspace Root Manifest
├── Cargo.lock
├── prd.md                          # Product Requirement Document
├── adr.md                          # Architecture Decision Records
├── TECH_STACK.md                   # Single Source of Truth (Dokumen Ini)
├── AGENTS.md                       # AI Agent Directives & Rules
│
├── apps/
│   └── web/                        # Next.js 16 Enterprise Frontend (App Router)
│       ├── public/                 # Static Assets, WebGL Shaders (.vert, .frag), Fonts
│       │   ├── icons/              # SVG Device & Status Icons
│       │   └── shaders/            # Custom WebGL/WebGPU pipeline shaders
│       │
│       ├── src/
│       │   ├── app/                # Next.js 16 App Router Directory
│       │   │   ├── (auth)/         # Route Group: Isolated Authentication Layout
│       │   │   │   ├── login/
│       │   │   │   │   └── page.tsx
│       │   │   │   ├── mfa/
│       │   │   │   │   └── page.tsx
│       │   │   │   └── layout.tsx
│       │   │   │
│       │   │   ├── (dashboard)/    # Route Group: Main Operational Portal
│       │   │   │   ├── topology/   # Interactive Hardware-Accelerated Topology Canvas
│       │   │   │   │   ├── loading.tsx
│       │   │   │   │   └── page.tsx
│       │   │   │   ├── metrics/    # High-density Time-Series Graphs (uPlot)
│       │   │   │   │   ├── loading.tsx
│       │   │   │   │   └── page.tsx
│       │   │   │   ├── alerts/     # Incident Triage & Root Cause Analysis Console
│       │   │   │   │   ├── loading.tsx
│       │   │   │   │   └── page.tsx
│       │   │   │   ├── gitops/     # Declarative Reconciliation & Drift Audit
│       │   │   │   │   ├── loading.tsx
│       │   │   │   │   └── page.tsx
│       │   │   │   └── layout.tsx  # Persistent Sidebar, Status Bar & Global State
│       │   │   │
│       │   │   ├── api/            # Server Route Handlers / BFF Proxies
│       │   │   │   └── proxy/      # Streaming Proxy ke Rust Axum Gateway
│       │   │   │       └── [...path]/
│       │   │   │           └── route.ts
│       │   │   │
│       │   │   ├── error.tsx       # Segment Error Boundary
│       │   │   ├── global-error.tsx# Root Error Boundary
│       │   │   ├── layout.tsx      # Root Layout (Theme Tokens, QueryClientProvider)
│       │   │   ├── loading.tsx     # Streaming Suspense Fallback
│       │   │   ├── not-found.tsx   # Enterprise 404 Handler
│       │   │   └── page.tsx        # System Executive Overview / Root Redirect
│       │   │
│       │   ├── components/         # Reusable Component Architecture
│       │   │   ├── canvas/         # WebGL / PixiJS Canvas Engine & LoD Viewport
│       │   │   ├── charts/         # uPlot Wrappers & Streaming Metrics Panels
│       │   │   ├── feedback/       # Toast notifications, Alarm Dialogs, Badges
│       │   │   └── ui/             # Radix Headless Primitives & Token Wrappers
│       │   │
│       │   ├── hooks/              # Custom Hooks (useTopologyStream, useWebSocket)
│       │   ├── lib/                # API Client, Token Interceptors & Formatting Utils
│       │   ├── stores/             # Zustand Stores (Topology Selection, UI Filters)
│       │   ├── styles/             # Design Token System, Typography & CSS Modules
│       │   └── types/              # Domain Models, Protobuf Typings, Topology Interfaces
│       │
│       ├── middleware.ts           # Edge/Node Route Guard (JWT & Sliding Session Verification)
│       ├── next.config.ts          # Next.js 16 Config (Turbopack, Worker & Shaders Support)
│       ├── package.json
│       └── tsconfig.json
│
├── crates/                         # Shared Internal Rust Libraries
│   ├── proto/                      # Protobuf & OpenConfig schema bindings (prost/tonic)
│   ├── models/                     # Shared domain models, canonical payload, & types
│   ├── auth-core/                  # Modul Argon2id, AES-256-GCM, TOTP, dan JWT
│   └── config/                     # Configuration loader & environment validator
│
├── services/                       # Binary Services (Rust Crates)
│   ├── collector-gnmi/             # Rust: Tonic gNMI streaming telemetry client
│   ├── collector-snmp/             # Rust: SNMP v3 poller & trap listener
│   ├── topology-engine/            # Rust: Neo4rs topology syncer & traversal engine
│   ├── aiops-correlator/           # Rust: Root cause analysis & graph RCA engine
│   ├── anomaly-detector/           # Python: Time-series predictive models (FastAPI)
│   ├── gitops-reconciler/          # Rust: Gix state comparator & Temporal worker
│   └── api-gateway/                # Rust: Axum HTTP / WebSocket Gateway (BFF)
│
└── deployments/
    ├── docker/                     # Dockerfiles & local docker-compose
    └── k8s/                        # Helm charts & Kubernetes manifests

```

---

## 6. Aturan Ketat untuk Pengembang & AI Agent

1. **Dilarang Menambahkan Framework / Crate Baru Tanpa ADR**: Jangan menambahkan framework frontend lain atau crate pihak ketiga di luar daftar di atas tanpa membuat ADR baru.


2. **Type Safety & Zero Unsafe**: Tidak diizinkan menggunakan tipe `any` pada TypeScript. Dilarang keras menggunakan blok `unsafe` pada Rust kecuali pada modul isolasi probe kernel eBPF. Dilarang menggunakan `.unwrap()` pada jalur kode produksi.


3. **Desain Visual Wajib Kelas Enterprise**: Antarmuka web harus berpenampilan modern, gelap (*dark-mode first*), responsif, bebas *lag*, dan tanpa elemen placeholder dummy.


4. **Target Performa**: Setiap kueri metrik analitik wajib lolos uji beban $< 1$ detik untuk rentang data 30 hari. Seluruh pipeline pengumpulan data wajib lolos pengujian throughput $\ge 1.000.000$ titik metrik/detik tanpa jeda Garbage Collector.