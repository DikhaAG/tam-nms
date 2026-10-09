# Source of Truth: Technology Stack & Engineering Standards
## Next-Generation Network Management System (NMS 2026)

Dokumen ini adalah **panduan mutlak (*Single Source of Truth*)** untuk seluruh pemilihan bahasa pemrograman, framework, pustaka (*libraries*), basis data, protokol, dan standar dependensi yang digunakan dalam pengembangan proyek **Next-Gen NMS**.

Seluruh implementasi modul dan fitur selanjutnya **wajib** merujuk pada standar yang tertera di dokumen ini untuk mencegah fragmentasi teknologi.

---

## 1. Arsitektur Makro Sistem

Sistem mengadopsi pola **Event-Driven Modular Monorepo / Microservices** yang terbagi menjadi 5 lapisan utama:

```
[ Layer 1: Edge & Telemetry Collectors ] (Go, gNMI, SNMP, eBPF)
                      │ (Streaming sub-detik)
                      ▼
[ Layer 2: Message Streaming & Buffer ] (Apache Kafka + Vector)
                      │
        ┌─────────────┴─────────────┐
        ▼                           ▼
[ Layer 3A: Storage Engines ]   [ Layer 3B: Processing Engines ]
 - Time-Series (VictoriaMetrics) - Topology & RCA Engine (Go)
 - Graph DB (Memgraph/Neo4j)     - AIOps Anomaly Detection (Python)
 - Relational/Meta (PostgreSQL)  - GitOps Reconciler (Go + Temporal)
        └─────────────┬─────────────┘
                      ▼
[ Layer 4: Application Core & API Gateway ] (Go / Node.js BFF, gRPC, WebSocket)
                      ▼
[ Layer 5: Web UI & Visualization ] (Next.js, TypeScript, WebGL/PixiJS)
```

---

## 2. Bahasa Pemrograman & Runtime

| Domain | Bahasa & Versi | Runtime / Toolchain | Alasan Pemilihan |
| :--- | :--- | :--- | :--- |
| **Collector & High-Throughput Services** | **Go (Golang)** `1.23+` | Official Go Toolchain | Konkurensi tinggi (*goroutines*), *low-memory footprint*, performa jaringan mendekati bare-metal, ekosistem gRPC/gNMI yang matang. |
| **Kernel-Level Observability** | **C / Go** (eBPF) | `cilium/ebpf` / Clang LLVM | Akses langsung ke kernel Linux untuk inspeksi socket, degradasi paket, dan RTT tanpa *overhead* userspace. |
| **AIOps & Machine Learning** | **Python** `3.12+` | UV / Poetry / CPython | Ekosistem data science, deteksi anomali deret waktu (*time-series anomaly detection*), dan model dependensi graf. |
| **Frontend & Web UI** | **TypeScript** `5.5+` | Node.js `20 LTS` / Bun | Type safety menyeluruh, integrasi kanvas graf WebGL, modern component lifecycle. |

---

## 3. Komponen Per Lapisan & Pustaka Resmi

### 3.1. Layer 1: Edge Ingestion & Telemetry Collectors (Go)
* **Framework / Core**: Go standard library (`net`, `sync`) dengan arsitektur worker-pool.
* **Protokol Telemetri Modern (gNMI/gRPC)**:
  * `github.com/openconfig/gnmic`: Library dan client resmi untuk gNMI subscription.
  * `google.golang.org/grpc`: Transport gRPC berperforma tinggi.
  * `github.com/openconfig/ygot`: Generator Go struct dan validator data model OpenConfig YANG.
* **Protokol Warisan (SNMP & Flow)**:
  * `github.com/gosnmp/gosnmp`: Pustaka SNMP v2c/v3 client dan trap listener.
  * `github.com/netsampler/goflow2`: Kolektor NetFlow v5/v9, IPFIX, dan sFlow.
* **Sensor Kernel**:
  * `github.com/cilium/ebpf`: Pustaka Go murni untuk memuat dan berinteraksi dengan program eBPF di Linux kernel.
* **Edge Aggregator & Shipper**:
  * **Vector** (Timber.io/Datadog): Ditulis dalam Rust, bertindak sebagai *sidecar collector/shipper* untuk parsing syslog dan pengiriman batch ke Kafka.

### 3.2. Layer 2: Message Streaming & Event Buffer
* **Message Broker Utama**: **Apache Kafka** `v3.7+` (Mode KRaft - tanpa ZooKeeper).
  * *Client Go*: `github.com/twmb/franz-go` (Performa tinggi, pure Go, alokasi memori minimal dibanding Sarama/Confluent).
* **Topic Partitioning Strategy**:
  * `telemetry.metrics.raw`: Key = `device_id` (menjamin urutan metrik per perangkat).
  * `telemetry.events.alarms`: Key = `site_id` atau `device_id`.
  * `telemetry.topology.changes`: Key = `topology_domain`.

### 3.3. Layer 3: Database & Penyimpanan (*Dual-Storage Strategy*)

#### A. Database Deret Waktu (*Time-Series*)
* **Mesin**: **VictoriaMetrics** (atau **ClickHouse** `v24+`).
  * *Fungsi*: Ingesti $\ge 1.000.000$ poin metrik/detik, kompresi ZSTD 5–10x, retensi data historis hingga 30+ hari.
  * *Go Driver*: HTTP client berbasis VictoriaMetrics Influx/Prometheus line protocol, atau `github.com/ClickHouse/clickhouse-go/v2`.

#### B. Database Graf (*Topology & Dependency*)
* **Mesin**: **Memgraph** atau **Neo4j** (Enterprise / Community).
  * *Fungsi*: Menyimpan simpul perangkat (*nodes*), antarmuka (*ports*), tautan fisik/logis (*edges*), relasi dependensi, dan rute BGP.
  * *Query Language*: Cypher (OpenCypher).
  * *Go Driver*: `github.com/neo4j/neo4j-go-driver/v5`.

#### C. Database Relasional, Kredensial & Metadata
* **Mesin**: **PostgreSQL** `16+`.
  * *Fungsi*: Manajemen akun pengguna & autentikasi mandiri (kredensial hash Argon2id, secret key TOTP terenkripsi, status aktif akun), data katalog inventaris perangkat, definisi peran granular RBAC, metadata konfigurasi GitOps, dan audit trail log.
  * *ORM / Query Builder*: `github.com/jackc/pgx/v5` + `sqlc` (Type-safe SQL compiler untuk Go).

#### D. In-Memory Cache & Ephemeral State
* **Mesin**: **Redis** `7.2+` (Cluster).
  * *Fungsi*: Penyimpanan live heartbeat (*status liveness* perangkat sub-detik), *distributed lock* untuk peredam flapping, rate-limiting API (termasuk proteksi brute-force login), serta cache pembatalan sesi (*token revocation / session blacklist*).
  * *Go Client*: `github.com/redis/go-redis/v9`.

---

### 3.4. Layer 4: Orchestration, GitOps & AIOps

#### A. Workflow Engine & Pemulihan Mandiri (Self-Healing)
* **Engine**: **Temporal.io** (`v1.24+`).
  * *Fungsi*: Menjalankan *workflow* pemulihan mandiri yang tahan uji (*fault-tolerant*), otomasi failover jalur, dan countdown stabilitas link (*hold-down timer*).
  * *SDK*: `go.temporal.io/sdk`.

#### B. Rekonsiliasi Konfigurasi Berbasis GitOps
* **Git Engine**: `github.com/go-git/go-git/v5` (Manipulasi repositori Git deklaratif langsung dari Go).
* **Format Konfigurasi Niat (*Intent*)**: Schema berbasis YAML / JSON yang divalidasi terhadap model OpenConfig YANG.

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
* **Framework**: **Next.js** `15+` (App Router, React 19).
* **Bahasa**: **TypeScript** (Strict Mode diaktifkan).
* **Bundler & Tooling**: Turbopack, ESLint, Prettier.

#### B. State Management & Data Fetching
* **Server State**: `@tanstack/react-query` (v5) untuk caching kueri metrik REST/GraphQL.
* **Client / UI State**: `zustand` (State ringan untuk kontrol tampilan dasbor, seleksi simpul graf, filter).
* **Komunikasi Realtime**:
  * WebSocket murni atau Server-Sent Events (SSE) untuk push event alarm dan live status.
  * Reconnection handling otomatis dengan *exponential backoff*.

#### C. Styling & UI Components
* **Styling**: **Vanilla CSS / CSS Modules** dengan *Design Token System* (Dark-mode primary: Slate/Zinc, Neon status indicators: Emerald untuk Up, Rose untuk Down, Amber untuk Warning).
* **Headless UI Primitives**: `@radix-ui/react-*` (Dialog, Dropdown, Tooltip, Tabs).
* **Ikonografi**: `lucide-react` (Ikon jaringan, server, switch, alarm).

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
  * **Penyimpanan Kredensial Internal**: Seluruh identitas akun, kredensial, dan hak akses dikelola langsung di database **PostgreSQL 16+**, menghilangkan ketergantungan pada Identity Provider pihak ketiga (seperti Okta, Keycloak, atau Azure AD) sehingga cocok untuk lingkungan *on-premise* maupun *air-gapped*.
  * **Algoritma Hashing Password**: **Argon2id** (`golang.org/x/crypto/argon2`) dengan *salt* acak kriptografis 16-byte dan parameter memori tahan serangan brute-force GPU/ASIC (Memory: 64MB, Iterations: 3, Parallelism: 2).
  * **Arsitektur Token & Manajemen Sesi**:
    * Akses API stateless menggunakan **JWT** (`github.com/golang-jwt/jwt/v5`) bertanda tangan HMAC-SHA256 / Ed25519 dengan masa berlaku singkat (*short-lived access token*, 15 menit).
    * Mekanisme *Rotating Refresh Token* (7 hari) yang disimpan dalam database PostgreSQL dengan *session hash* unik, dipadukan dengan *blacklist/revocation cache* di **Redis** untuk mendukung terminasi sesi seketika (*instant force logout*).
  * **Multi-Factor Authentication (MFA / 2FA) Mandiri**:
    * Mengimplementasikan **TOTP** (Time-based One-Time Password, RFC 6238) menggunakan `github.com/pquerna/otp` yang kompatibel dengan aplikasi authenticator umum (Google Authenticator, Microsoft Authenticator, Aegis).
    * Rahasia (*secret key*) TOTP disimpan dalam PostgreSQL dalam keadaan terenkripsi menggunakan **AES-256-GCM**.
  * **Proteksi Brute-Force & Rate Limiting**:
    * *Rate limiting* ketat pada rute `/api/v1/auth/login` menggunakan Redis *sliding-window algorithm* (maksimal 5 percobaan gagal per IP/Username dalam 5 menit).
    * Penguncian akun progresif (*progressive account lockout*) dan pencatatan otomatis ke *immutable audit log*.
* **Otorisasi & RBAC**:
  * Granular RBAC hingga level antarmuka/perangkat menggunakan **Open Policy Agent (OPA)** atau **Casbin** Go engine.
* **Enkripsi**:
  * *In-Transit*: **TLS 1.3** wajib di seluruh komunikasi internal (mTLS antar microservices) dan eksternal.
  * *At-Rest*: **AES-256-GCM** pada level disk/database.
* **Audit Trail**:
  * Log audit bersifat *immutable* (hanya append) yang disegel dengan rantai hash kriptografis (*cryptographic SHA-256 block chain*).

---

## 5. Standar Struktur Repositori (Monorepo Layout)

```
/home/rizxenzs/Desktop/nms/
├── prd.md                          # Product Requirement Document
├── adr.md                          # Architecture Decision Records
├── TECH_STACK.md                   # Single Source of Truth (Dokumen Ini)
│
├── apps/
│   └── web/                        # Next.js Frontend Dashboard & WebGL Canvas
│       ├── app/                    # Next.js App Router
│       ├── components/             # Reusable UI & Topology Canvas components
│       ├── lib/                    # API clients, hooks, stores
│       └── styles/                 # Design tokens & CSS
│
├── services/
│   ├── collector-gnmi/             # Go: gNMI streaming telemetry collector
│   ├── collector-snmp/             # Go: SNMP v3 poller & trap listener
│   ├── topology-engine/            # Go: Penemuan topologi & Neo4j/Memgraph syncer
│   ├── aiops-correlator/           # Go / Python: Root cause analysis & alert suppression
│   ├── anomaly-detector/           # Python: Time-series predictive models
│   ├── gitops-reconciler/          # Go: Git state comparator & Temporal worker
│   └── api-gateway/                # Go: GraphQL / REST / WebSocket Gateway (BFF)
│
├── packages/
│   ├── proto/                      # Protobuf & gRPC schema definitions
│   ├── models/                     # Shared domain models & YANG mappings
│   └── design-system/              # Reusable token & UI assets
│
└── deployments/
    ├── docker/                     # Dockerfiles & local docker-compose
    └── k8s/                        # Helm charts & Kubernetes manifests
```

---

## 6. Aturan Ketat untuk Pengembang & AI Agent

1. **Dilarang Menambahkan Framework Baru Tanpa ADR**: Jangan menambahkan framework frontend lain (misal: Tailwind jika tidak disetujui, Vue, Angular) atau database baru di luar daftar di atas tanpa membuat ADR baru.
2. **Type Safety Wajib 100%**: Tidak diizinkan menggunakan tipe `any` pada TypeScript atau `interface{}` tanpa justifikasi arsitektural pada Go.
3. **Desain Visual Wajib Kelas Enterprise**: Antarmuka web harus berpenampilan modern, gelap (*dark-mode first*), responsif, bebas *lag*, dan tanpa elemen placeholder dummy.
4. **Target Performa**: Setiap kueri metrik analitik wajib lolos uji beban $< 1$ detik untuk rentang data 30 hari.
