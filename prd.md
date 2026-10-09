Firefox about:blank
Cabang • PRD Network Management
System 2026
https://gemini.google.com/app/4d4d75d90e2dfc8a
User prompt: bantu saya merancang prd untuk aplikasi network management system terbaik
2026
Response: Product Requirement
Document: Next-Generation Network
Management System
1. Ringkasan Eksekutif
Aplikasi Network Management System ini dirancang untuk infrastruktur hybrid multi-
vendor skala menengah hingga enterprise. Sistem berfokus pada peralihan dari
monitoring reaktif berbasis polling tradisional ke observabilitas prediktif menggunakan
streaming telemetry, korelasi alarm berbasis machine learning, dan automasi pemulihan
mandiri dengan pendekatan GitOps.
2. Pengguna Target dan Masalah Utama
| Peran Pengguna | Kebutuhan Utama | Masalah Saat Ini            |
| -------------- | --------------- | --------------------------- |
| Network        |                 | Kewalahan menghadapi banjir |
Triage insiden cepat dan
| Operations Center |     | notifikasi saat terjadi kegagalan |
| ----------------- | --- | --------------------------------- |
visibilitas alert terkonsolidasi
| Tier 1 dan 2 |     | jaringan skala besar |
| ------------ | --- | -------------------- |
Network Engineer Validasi konfigurasi deklaratif Konfigurasi manual rawan
dan Network dan audit kepatuhan kesalahan dan minim visibilitas
| Architect | perangkat | topologi hybrid cloud |
| --------- | --------- | --------------------- |
Deteksi anomali lalu lintas
| Security Operations |     | Visibilitas terbatas terhadap lalu |
| ------------------- | --- | ---------------------------------- |
lateral dan pemetaan
| Engineer |     | lintas data antar segmen internal |
| -------- | --- | --------------------------------- |
kerentanan firmware
3. Kebutuhan Fungsional
Observabilitas dan Pengumpulan Data
1 of 38 10/9/26, 09:37

Firefox about:blank
•
Streaming Telemetry Terpadu: Mengumpulkan metrik secara kontinu
menggunakan protokol gNMI, gRPC, dan OpenConfig dengan latensi transmisi sub-
detik.
•
Dukungan Perangkat Multi-Vendor: Menyediakan gateway penampung data untuk
protokol SNMP v3, NetFlow, IPFIX, dan sFlow untuk mengakomodasi perangkat
warisan seperti router edge, switch modular, dan titik akses nirkabel.
•
Visibilitas Tingkat Kernel: Menggunakan instrumen eBPF untuk memantau
throughput jaringan, latensi round-trip, dan degradasi paket pada server bare-metal
serta kluster kontainer.
AIOps dan Analisis Insiden
•
Peredam Banjir Alarm: Mengelompokkan ribuan peristiwa alarm yang muncul
bersamaan menjadi satu tiket insiden tunggal berdasarkan relasi dependensi
perangkat.
•
Deteksi Anomali Prediktif: Menerapkan model deret waktu untuk mengenali
deviasi performa seperti kenaikan jitter, penurunan level daya optik serat optik, dan
saturasi buffer antarmuka sebelum terjadi pemutusan koneksi.
•
Analisis Akar Masalah Otomatis: Memetakan dependensi perangkat dalam bentuk
graf terarah guna menandai node sumber kegagalan secara spesifik.
Automasi dan Manajemen Konfigurasi
•
Konfigurasi Berbasis Niat: Penerapan konfigurasi jaringan secara deklaratif yang
otomatis memvalidasi parameter sebelum dieksekusi ke perangkat target.
•
Rekonsiliasi Status Berbasis Git: Seluruh status konfigurasi disimpan dalam
repositori Git dengan fitur rollback instan apabila pengujian pasca-penerapan
mengalami kegagalan.
•
Pemulihan Mandiri: Menyediakan penanganan otomatis untuk insiden berulang,
seperti pengalihan rute lalu lintas ketika terjadi penurunan kualitas tautan transmisi.
Visualisasi dan Pemetaan Topologi
•
Penemuan Topologi Otomatis Multi-Layer: Pemetaan dinamis lapisan data link dan
lapisan jaringan menggunakan protokol LLDP, CDP, serta peering BGP.
•
Visualisasi Graf Real-Time: Kanvas interaktif berbasis akselerasi grafis perangkat
keras yang mampu merender puluhan ribu perangkat dan tautan tanpa penurunan
performa rendering.
•
Audit Perubahan Rute: Perekaman historis perubahan jalur transmisi paket untuk
2 of 38 10/9/26, 09:37

Firefox about:blank
meninjau kembali performa rute saat terjadi degradasi koneksi.
4. Kebutuhan Non-Fungsional
Performa dan Skalabilitas
•
Mampu memproses sedikitnya satu juta titik metrik per detik dengan keterlambatan
pemrosesan data maksimal lima ratus milidetik.
•
Waktu respons kueri pada dasbor pemantauan di bawah satu detik untuk rentang
data historis tiga puluh hari.
•
Arsitektur backend bersifat horizontal dan tanpa komponen tunggal yang rentan
memicu kegagalan sistem menyeluruh.
Keamanan dan Kepatuhan
•
Autentikasi mandiri berbasis basis data internal (Internal / Custom Database-backed Auth) menggunakan hashing Argon2id, token sesi aman (JWT), proteksi MFA mandiri (TOTP), dengan kontrol
akses berbasis peran yang granular hingga tingkat antarmuka perangkat.
•
Seluruh transmisi data dienkripsi menggunakan protokol TLS 1.3, dan data
tersimpan dilindungi enkripsi AES 256-bit.
•
Pencatatan log audit immutable untuk setiap instruksi konfigurasi dan aktivitas
pengguna di dalam sistem.
5. Arsitektur Komponen dan Teknologi
Rekomendasi
Lapisan Sistem Tanggung Jawab
Teknologi
Menampung dan memvalidasi arus
Ingesti Data Apache Kafka, Vector
telemetri berkecepatan tinggi
Database Deret VictoriaMetrics, Menyimpan metrik metrik performa jangka
Waktu ClickHouse panjang secara efisien
Memodelkan relasi topologi fisik, logis, dan
Database Graf Neo4j, Memgraph
jalur routing
Engine Menjalankan alur kerja automasi dan
Go, Temporal
Orkestrasi pemulihan konfigurasi
Antarmuka Menyajikan visualisasi topologi dinamis dan
Next.js, WebGL
Pengguna dasbor metrik
3 of 38 10/9/26, 09:37

Firefox about:blank
6. Metrik Keberhasilan
•
Mean Time to Detect: Penurunan durasi deteksi insiden menjadi di bawah tiga
puluh detik sejak penyimpangan performa pertama terdeteksi.
•
Mean Time to Resolve: Pengurangan rata-rata durasi penanganan insiden sebesar
lima puluh persen berkat identifikasi akar masalah otomatis.
•
Tingkat Reduksi Alarm: Pengurangan volume alarm mentah minimal delapan puluh
lima persen melalui agregasi insiden cerdas.
•
Akurasi Topologi: Tingkat ketepatan pemetaan perangkat dan tautan mencapai
minimal sembilan puluh sembilan persen secara kontinu.
Sources
1. Otomatisasi PRD multiagen dengan MetaGPT, Ollama, dan DeepSeek
User prompt: bentar, apa itu network management system
Response: Bayangkan sebuah kota besar yang punya ribuan persimpangan jalan dan
lampu lalu lintas. Kalau ada satu lampu merah mati atau terjadi macet parah di sudut
kota, petugas nggak mungkin jalan kaki keliling satu per satu setiap menit buat ngecek.
Mereka butuh satu ruang kendali pusat dengan layar monitor besar yang terhubung ke
sensor di tiap jalan. Dari ruangan itu, petugas bisa langsung tahu titik mana yang
bermasalah dan bisa ngatur ulang lampu lalu lintas dari jarak jauh.
Network Management System adalah ruang kendali pusat itu, tapi wujudnya berupa
aplikasi perangkat lunak untuk jaringan komputer.
Di kantor, kampus, atau penyedia internet, ada ratusan perangkat keras seperti pemancar
Wi-Fi, kabel penghubung, dan komputer server yang saling ngobrol. Kalau nggak ada
sistem pemantau terpusat, teknisi bakal kebingungan pas internet mendadak putus
karena nggak tahu alat mana yang sebenarnya rusak di antara ratusan perangkat tadi.
Sistem ini bekerja lewat tiga fungsi dasar:
•
Melihat: Terus-menerus mengecek apakah semua perangkat masih hidup dan alur
data berjalan lancar.
•
Mengingatkan: Membunyikan alarm otomatis ke tim teknisi begitu ada jalur yang
mulai lambat, kepanasan, atau mati, bahkan sebelum pengguna sempat komplain.
•
Mengatur: Mengubah setelan perangkat dari satu layar kontrol tanpa teknisi harus
repot datang langsung ke lokasi buat mencolok kabel.
4 of 38 10/9/26, 09:37

Firefox about:blank
Biar makin jelas konsepnya, coba bayangkan skenario ini:
Di sebuah rumah sakit, koneksi data di ruang operasi tiba-tiba terputus, tapi para dokter
di dalam ruangan belum sempat melapor karena lagi fokus menangani pasien. Dari tiga
fungsi dasar tadi, fungsi mana yang paling pertama bekerja buat menyelamatkan situasi
itu, dan gimana cara kerjanya?
User prompt: Mengingatkan: Membunyikan alarm otomatis ke tim teknisi begitu ada jalur yang
mulai lambat, kepanasan, atau mati, bahkan sebelum pengguna sempat komplain.
Response: Alarm memang yang bikin teknisi langsung sadar sebelum ada yang komplain.
Tapi kalau kita bedah urutan kejadiannya, alarm sebenarnya nggak bisa bunyi sendiri
tanpa pemicu.
Fungsi yang bekerja paling awal adalah Melihat.
Sistem ini bekerja mirip petugas yang terus mengecek denyut nadi. Setiap beberapa detik,
sistem mengirim sinyal sapaan kecil ke perangkat di ruang operasi untuk memastikan alat
itu masih merespons.
Begitu sinyal sapaan tadi tidak berbalas, fungsi Melihat langsung mendeteksi bahwa
koneksi terputus. Baru sepersekian detik setelahnya, fungsi Mengingatkan aktif untuk
membunyikan alarm dan mengirim peringatan ke tim teknisi. Setelah itu, fungsi
Mengatur bisa langsung bekerja mengalihkan koneksi ke jalur cadangan.
Sekarang coba bayangkan tantangan teknisnya:
Kalau fungsi Melihat ini disetel terlalu agresif—misalnya sistem disuruh mengecek dan
menyapa ribuan perangkat setiap satu milidetik tanpa henti—masalah baru apa yang
bakal muncul di jaringan itu?
User prompt: saya tidak tahu
Response: Bayangkan Anda sedang fokus bekerja di sebuah ruangan. Tiba-tiba, setiap
satu detik ada orang yang mengetuk pintu dan bertanya, "Kamu masih aman? Kamu
masih kerja?" tanpa henti.
Dua masalah utama yang akan terjadi:
1. Perangkat kehabisan tenaga: Komputer kecil di dalam perangkat jaringan terpaksa
menghabiskan seluruh tenaganya hanya untuk membalas sapaan sistem pemantau.
Akibatnya, perangkat justru kewalahan dan macet karena tidak sempat mengurus
pengiriman data pengguna.
5 of 38 10/9/26, 09:37

Firefox about:blank
2. Jalanan menjadi macet: Kabel data memiliki batas kapasitas muatan. Jika jalurnya
dipenuhi oleh jutaan pesan tanya-jawab status yang lewat bersamaan, data penting
milik pengguna akan tersendat di antrean.
Alih-alih menjaga jaringan tetap sehat, sistem pemantau yang terlalu agresif justru
menjadi penyebab utama jaringan itu tumbang.
Jika memeriksa terlalu sering membuat jaringan macet, sedangkan memeriksa terlalu
jarang membuat kita terlambat mengetahui kerusakan, kira-kira bagaimana cara mencari
jalan tengah agar sistem tetap sigap tanpa membebani perangkat?
User prompt: saya tidak tahu
Response: Solusinya adalah mengubah cara komunikasinya: dari sistem yang terus
bertanya, menjadi perangkat yang berinisiatif melapor saat ada masalah.
Bayangkan seorang satpam dan pemilik rumah.
Daripada satpam mengetuk pintu setiap sepuluh detik untuk bertanya "Apakah rumah
aman?", satpam cukup memberi instruksi: "Tetaplah beraktivitas normal. Tapi begitu Anda
melihat ada asap atau maling masuk, langsung tekan tombol alarm darurat ke pos satpam."
Dalam jaringan komputer, para insinyur memadukan dua pendekatan ini:
•
Laporan Otomatis saat Darurat: Perangkat dibiarkan bekerja tenang tanpa terus
diganggu pertanyaan. Namun, begitu ada masalah genting—misalnya suhu mesin
mendadak panas atau ada kabel tercabut—perangkat itu sendiri yang langsung
mengirim pesan peringatan ke ruang kendali sepersekian detik saat kejadian
berlangsung.
•
Pemeriksaan Santai: Ruang kendali tetap mengirim sapaan, tetapi dengan ritme
santai (misalnya setiap beberapa menit sekali, bukan setiap detik), hanya untuk
memastikan perangkat tersebut tidak mati total.
Dengan kombinasi ini, kabel data tidak macet dan otak perangkat tidak terkuras
tenaganya.
Namun, ada satu celah logis pada metode ini:
Jika kabel penghubung perangkat tersebut mendadak putus total atau listriknya padam
seketika, menurut Anda, apakah perangkat itu masih sempat menekan "tombol alarm
darurat" untuk melapor ke ruang kendali?
User prompt: tidak
6 of 38 10/9/26, 09:37

Firefox about:blank
Response: Tepat sekali. Saat aliran listrik padam atau kabel terputus, perangkat
kehilangan daya dan jalur bicara, sehingga tidak mungkin sempat mengirimkan sinyal
darurat apa pun.
Di sinilah peran penting dari pemeriksaan santai yang kita bahas sebelumnya.
Mekanisme ini bekerja seperti aturan kapal yang wajib melapor ke pos pantai secara
berkala. Jika pos pantai biasanya menerima kabar setiap lima menit, lalu tiba-tiba kapal
tersebut hening total dan tidak membalas panggilan, pos pantai langsung menyimpulkan
terjadi kecelakaan. Dalam sistem pemantau, ketiadaan kabar atau keheningan itu sendiri
diperlakukan sebagai tanda bahaya.
Sistem tidak perlu menunggu jeritan minta tolong; cukup dengan melihat jam hitung
mundur yang habis tanpa ada balasan.
Sekarang, mari uji logika waktu pemeriksaannya:
Jika sistem pemantau dijadwalkan menyapa perangkat setiap sepuluh menit sekali, lalu
kabel perangkat tersebut putus tiba-tiba pada menit pertama:
Berapa lama ruang kendali baru menyadari bahwa perangkat itu mati, dan apa risiko
keterlambatan ini bagi pengguna layanan?
User prompt: sistem akan mengetahuinya pada 9 menit berikutnya, itu waktu yang sangat
banyak
Response: Sembilan menit adalah waktu yang sangat lama. Bagi rumah sakit atau
perbankan, keterlambatan selama itu bisa mengacaukan ribuan transaksi atau
membahayakan pasien sebelum teknisi sempat menyadari masalahnya.
Untuk menyelesaikan dilema antara kecepatan deteksi dan beban jaringan, para pembuat
sistem membagi komunikasi menjadi dua jenis pesan yang berbeda:
•
Pemeriksaan Menyeluruh: Dilakukan dengan ritme santai, misalnya setiap sepuluh
menit. Pesan ini meminta data yang berat dan terperinci, seperti sisa kapasitas
memori, beban kerja prosesor, dan suhu perangkat.
•
Detak Jantung: Dikirim secara terus-menerus setiap satu atau dua detik sekali.
Pesan ini bukan laporan panjang, melainkan hanya satu ketukan sinyal yang luar
biasa kecil, sekadar untuk memastikan perangkat tersebut masih bernapas.
Karena ukurannya yang sangat ringan, ribuan sinyal detak jantung yang dikirim setiap
detik tidak akan membebani kapasitas kabel maupun menguras tenaga perangkat. Jika
sistem pemantau tidak menerima tiga detak jantung berturut-turut, ruang kendali sudah
bisa memastikan koneksi putus hanya dalam hitungan tiga hingga lima detik.
7 of 38 10/9/26, 09:37

Firefox about:blank
Mari kita uji cara kerja gabungan ini:
Sebuah kantor memasang dua jalur internet sekaligus, yaitu kabel utama yang
berkecepatan tinggi dan kabel cadangan. Jika sistem pemantau mendeteksi detak jantung
di kabel utama mendadak hilang selama tiga detik, tindakan otomatis apa yang sebaiknya
langsung dijalankan oleh fungsi Mengatur agar karyawan di kantor tersebut sama sekali
tidak merasakan internet mati?
User prompt: saya tidak tahu
Response: Tindakannya adalah langsung membelokkan seluruh aliran data ke kabel
cadangan.
Ibarat lintasan kereta api yang memiliki rel utama dan rel cadangan berdampingan. Jika
rel utama mendadak amblas, sistem otomatis langsung menggeser tuas pemindah rel
sehingga kereta yang meluncur di belakangnya beralih ke rel cadangan tanpa perlu
mengerem mendadak.
Di dalam jaringan komputer, fungsi Mengatur bekerja persis seperti tuas pemindah rel
tersebut:
•
Memutuskan pengiriman data ke kabel utama yang mati.
•
Mengalihkan seluruh lalu lintas data detik itu juga ke kabel cadangan.
Karena proses peralihan ini tuntas dalam hitungan pecahan detik, karyawan yang sedang
berkirim data atau melakukan panggilan video tidak akan merasakan gangguan sama
sekali.
Setelah koneksi berhasil diselamatkan, barulah fungsi Mengingatkan mengirimkan
notifikasi ke ponsel teknisi untuk memberi tahu bahwa jalur internet sedang berjalan
menggunakan kabel cadangan dan kabel utama perlu diperbaiki secara fisik.
Rangkaian ketiga fungsi tersebut bekerja saling melengkapi:
1. Melihat mendeteksi hilangnya detak jantung pada kabel utama.
2. Mengatur menyelamatkan koneksi dengan beralih ke kabel cadangan secara instan.
3. Mengingatkan memanggil teknisi untuk membenahi kerusakan fisik di lokasi.
Sekarang bayangkan kabel utama yang putus tadi selesai disambung kembali oleh teknisi.
Menurut Anda, apakah sistem sebaiknya langsung memindahkan aliran data kembali ke
kabel utama pada detik pertama kabel menyala, atau menunggu beberapa saat terlebih
dahulu?
8 of 38 10/9/26, 09:37

Firefox about:blank
User prompt: langung memindahkan aliran data kembali ke kabel utama pada detik pertama
kabel menyala
Response: Jika sistem langsung memindahkan aliran data pada detik pertama kabel
menyala, jaringan justru berisiko mengalami kekacauan yang jauh lebih parah.
Bayangkan seseorang sedang mencolokkan kabel lampu yang kendur. Sebelum menancap
erat, ujung kabel biasanya berguncang dan lampunya berkedip-kedip menyala dan mati
beberapa kali dalam beberapa detik.
Hal yang sama sering terjadi saat kabel jaringan baru selesai diperbaiki. Teknisi di
lapangan mungkin masih merapikan lekukan kabel, mengencangkan baut konektor, atau
sinyal listriknya masih belum stabil. Jika sistem langsung beralih seketika pada kedipan
pertama:
•
Data terpental bolak-balik: Jalur data akan dipaksa melompat ke kabel utama,
terputus lagi sedetik kemudian, melompat kembali ke cadangan, lalu beralih lagi ke
utama.
•
Koneksi pengguna terputus-putus: Panggilan suara atau rapat video pengguna
akan mengalami gangguan jauh lebih buruk dibanding jika tetap bertahan di kabel
cadangan.
•
Beban prosesor melonjak: Alat pengatur rute data akan kewalahan karena dipaksa
memproses perpindahan jalur berulang kali dalam waktu sangat singkat.
Fenomena pantulan berulang ini dicegah dengan menerapkan waktu tunggu stabilitas.
Sistem pemantau akan bersikap tenang dan menahan diri terlebih dahulu. Sistem
mewajibkan kabel utama membuktikan kestabilannya—misalnya dengan mengirim detak
jantung tanpa putus sama sekali selama tiga puluh detik atau satu menit penuh. Begitu
kabel terbukti benar-benar kokoh, barulah fungsi Mengatur memindahkan arus data
kembali ke kabel utama secara mulus.
Sekarang coba uji prinsip ini lewat skenario berikut:
Anda sedang mengamati layar ruang kendali. Lampu kabel utama menyala selama dua
detik, padam satu detik, lalu menyala lagi selama dua detik.
Berdasarkan prinsip waktu tunggu stabilitas, apa tindakan yang seharusnya diambil oleh
fungsi Mengatur pada detik kelima tersebut?
User prompt: saya tidak tahu
Response: Tindakannya adalah tetap mempertahankan aliran data di kabel cadangan
9 of 38 10/9/26, 09:37

Firefox about:blank
dan mengatur ulang hitungan waktu tunggu dari nol.
Ibarat menguji jembatan kayu yang baru diperbaiki. Aturannya: jembatan hanya boleh
dilewati jika sudah tidak bergoyang sama sekali selama tiga puluh detik penuh. Jika
jembatan diam dua detik, bergoyang lagi satu detik, lalu diam lagi dua detik, kita tidak
boleh langsung menyuruh orang menyeberang. Begitu jembatan bergoyang, hitungan
tiga puluh detik harus diulang dari nol sejak goyangan terakhir berhenti.
Di dalam jaringan komputer, peristiwa kabel menyala lalu padam berulang kali ini disebut
kondisi goyah.
Ketika kabel padam pada detik ketiga, catatan stabilitas kabel utama langsung hangus.
Pada detik kelima, kabel tersebut baru berhasil membuktikan dirinya menyala selama dua
detik berturut-turut. Angka dua detik tentu belum membuktikan bahwa kabel sudah
benar-benar aman dan kokoh.
Jika sistem memindahkan data terlalu terburu-buru, pengguna akan langsung terkena
imbas pemutusan berikutnya. Oleh karena itu, sistem memilih tetap bertahan di kabel
cadangan yang terbukti aman sembari menyalakan kembali arloji hitung mundurnya.
Mari kita pastikan perhitungannya:
Jika sistem menetapkan syarat kabel utama harus menyala tanpa putus selama tiga puluh
detik penuh sebelum data dipindahkan kembali:
Terhitung sejak detik kelima tersebut, pada detik ke berapa kabel utama baru boleh
mengambil alih aliran data kembali jika setelah itu tidak pernah padam lagi?
User prompt: pada detik ke 35
Response: Mari kita telusuri garis waktunya detik demi detik:
•
Detik 0 sampai 2: Kabel menyala selama dua detik.
•
Detik 2 sampai 3: Kabel padam selama satu detik. Gangguan ini membatalkan
seluruh catatan sebelumnya dan mewajibkan arloji stabilitas diulang dari angka nol.
•
Detik 3: Kabel menyala kembali. Arloji hitung mundur tiga puluh detik mulai berjalan
tepat pada titik ini.
•
Detik 5: Kabel telah membuktikan dirinya menyala stabil tanpa putus selama dua
detik (dari detik ketiga hingga kelima).
•
Karena targetnya adalah tiga puluh detik berturut-turut terhitung sejak menyala
kembali di detik ketiga, kabel tersebut hanya membutuhkan dua puluh delapan detik
tambahan (30 dikurangi 2).
10 of 38 10/9/26, 09:37

Firefox about:blank
Jika dihitung dari detik ketiga ditambah tiga puluh detik, atau dari detik kelima ditambah
sisa waktu dua puluh delapan detik, pemindahan data dilakukan pada detik ke-33.
Angka 35 akan didapat jika arloji tiga puluh detik baru dinyalakan pada detik kelima.
Namun, karena kabel sudah mengantongi kestabilan selama dua detik sejak detik ketiga,
jawaban yang tepat adalah pada detik ke-33.
Sekarang ketiga pilar cara kerja Network Management System sudah utuh:
1. Melihat: Memeriksa detak jantung ringan tiap detik agar sigap mendeteksi
pemutusan tanpa membebani kabel.
2. Mengatur: Memindahkan jalur data ke kabel cadangan secara instan, serta
menahan peralihan kembali sampai kabel utama benar-benar stabil.
3. Mengingatkan: Memanggil teknisi ke lokasi hanya saat intervensi fisik diperlukan.
User prompt: Mari kita hubungkan pemahaman fungsi Melihat, Mengatur, dan Mengingatkan
ini kembali ke dalam rancangan PRD Network Management System sebelumnya.
Response: Ketiga fungsi dasar tersebut merupakan fondasi logis dari seluruh bab teknis di
dalam dokumen PRD:
Pemetaan pada
| Fungsi Dasar |     | Penerapan Teknis dalam Sistem |
| ------------ | --- | ----------------------------- |
Fitur PRD
Memisahkan aliran data menjadi dua jalur:
Observabilitas transmisi telemetri ringan untuk detak jantung
Melihat dan Pengumpulan tiap detik guna deteksi cepat, serta penarikan
|     | Data | metrik berat secara santai berkala agar |
| --- | ---- | --------------------------------------- |
perangkat tidak kehabisan daya komputasi.
Menjalankan pengalihan rute lalu lintas ke jalur
|          | Automasi dan | cadangan seketika saat detak jantung putus,     |
| -------- | ------------ | ----------------------------------------------- |
| Mengatur | Pemulihan    | sekaligus mengunci logika waktu tunggu          |
|          | Mandiri      | stabilitas agar data tidak memantul bolak-balik |
saat kabel utama baru menyala.
Menerima sinyal darurat langsung dari
|     | AIOps dan | perangkat saat anomali muncul, serta |
| --- | --------- | ------------------------------------ |
Mengingatkan Manajemen memadatkan ribuan rentetan notifikasi turunan
|     | Insiden | menjadi satu tiket insiden tunggal bagi teknisi |
| --- | ------- | ----------------------------------------------- |
lapangan.
11 of 38 10/9/26, 09:37

Firefox about:blank
1. Fungsi Melihat menjadi Fondasi Observabilitas
•
Detak Jantung Sub-Detik: Sistem mewajibkan pengiriman sinyal status ultra-ringan
secara terus-menerus. Jika tiga detak jantung hilang berturut-turut, sistem
menyimpulkan koneksi putus dalam tempo kurang dari tiga detik.
•
Audit Menyeluruh Terjadwal: Pembacaan data berat seperti beban prosesor,
temperatur perangkat, dan kapasitas memori dijadwalkan dengan interval santai
beberapa menit sekali agar kapasitas transmisi data pengguna tetap lapang.
2. Fungsi Mengatur menjadi Logika Automasi Jaringan
•
Pengalihan Jalur Instan: Begitu fungsi pemantau memastikan ketiadaan detak
jantung, pengendali otomatis langsung membelokkan rute paket ke jalur cadangan
dalam hitungan milidetik sehingga pengguna tidak merasakan pemutusan layanan.
•
Peredam Kondisi Goyah: Saat kabel utama tersambung kembali, sistem tidak
langsung memindahkan arus data. Sistem mewajibkan kabel utama menyala stabil
tanpa gangguan selama batas waktu tertentu secara penuh. Jika sinyal berkedip di
tengah jalan, hitungan waktu tunggu langsung diulang dari awal.
3. Fungsi Mengingatkan menjadi Sistem Pengendali Alarm Cerdas
•
Pelaporan Mandiri saat Darurat: Perangkat dikonfigurasi untuk langsung
melepaskan sinyal darurat ke ruang kendali pada detik pertama komponen internal
mengalami anomali.
•
Penyaringan Alarm Terpusat: Saat kabel transmisi utama putus, sistem menahan
ribuan alarm sekunder dari perangkat cabang yang ikut terdampak. Ruang kendali
hanya membunyikan satu peringatan utama yang mengarahkan teknisi ke titik
putusnya kabel fisik.
User prompt: baik, sekarang jelaskan maksud dari hybrid multi-vendor dari kalimat "Aplikasi
Network Management System ini dirancang untuk infrastruktur hybrid multi-vendor skala
menengah hingga enterprise.
Response: Istilah hybrid multi-vendor menggambarkan kondisi jaringan yang
memadukan dua lingkungan penempatan yang berbeda dan beragam merek
perangkat keras.
1. Multi-Vendor (Bermacam-Merek) Perusahaan jarang menggunakan satu merek
perangkat untuk seluruh jaringannya. Di dalam satu gedung, pengatur rute utama
mungkin bermerek Cisco, pemancar Wi-Fi bermerek Aruba, dan sistem pengaman data
bermerek Fortinet.
Masalahnya, setiap pabrikan biasanya membuat aplikasi pemantau khusus untuk alat
12 of 38 10/9/26, 09:37

Firefox about:blank
mereka sendiri. Jika sistem pemantau tidak mendukung multi-vendor, teknisi terpaksa
membuka tiga sampai empat aplikasi berbeda di layar yang berbeda hanya untuk
mengawasi satu ruangan. Sistem pemantau multi-vendor bertindak sebagai penerjemah
universal yang mampu berbicara dengan semua merek tersebut dan menyatukan
statusnya ke dalam satu layar kendali tunggal.
2. Hybrid (Gabungan Fisik dan Awan) Hybrid mengacu pada perpaduan dua model
penempatan komputer server:
•
Fisik di Lokasi: Perangkat keras nyata berupa komputer server dan kabel yang dibeli
sendiri lalu diletakkan di lemari ruang server kantor.
•
Awan (Cloud): Server virtual yang disewa dari pihak penyedia internet dan pusat
data di luar kantor.
Jaringan hybrid menghubungkan komputer karyawan di kantor langsung ke server
sewaan di awan tersebut. Sistem pemantau hybrid wajib mampu melihat kesehatan kabel
tembaga nyata di kantor sekaligus jalur transmisi virtual yang melintasi internet publik.
Mari kita uji konsep ini lewat skenario nyata:
Sebuah universitas menaruh data nilai mahasiswa di server fisik ruang kampus bermerek
A, sementara sistem pendaftaran mahasiswa baru disewa di server awan (cloud). Semua
pemancar Wi-Fi di ruang kelas menggunakan merek B.
Jika universitas tersebut keliru membeli aplikasi pemantau yang hanya bisa membaca
satu vendor tertentu dan hanya mengenali perangkat fisik lokal, kendala operasional
apa yang akan langsung dialami oleh teknisi kampus saat sistem pendaftaran di awan
tiba-tiba lambat atau pemancar Wi-Fi bermasalah?
User prompt: saya tidak tahu
Response: Dua kendala utama yang akan langsung dialami oleh teknisi:
Pertama, titik buta total pada layar pemantau. Layar di ruang kendali akan tetap
menampilkan warna hijau atau tanda normal karena server fisik memang menyala
dengan baik. Sistem pemantau tersebut sama sekali tidak mampu melihat kondisi
pemancar Wi-Fi maupun server awan. Akibatnya, teknisi tidak tahu ada kerusakan sampai
ratusan mahasiswa datang melapor dan menyampaikan keluhan.
Kedua, pencarian masalah menjadi sangat lambat. Teknisi terpaksa memeriksa setiap
bagian secara manual. Mereka harus berjalan kaki menyusuri lorong kampus untuk
melihat lampu indikator fisik pada pemancar Wi-Fi, lalu membuka aplikasi lain secara
terpisah untuk memeriksa server awan. Waktu habis hanya untuk mencari letak titik
kerusakan, bukan untuk memperbaikinya.
13 of 38 10/9/26, 09:37

Firefox about:blank
Ibarat seorang dokter yang hanya dibekali alat periksa mata, dokter tersebut tidak akan
bisa mengetahui apa yang terjadi ketika pasien mengeluh sakit perut atau sesak napas.
Itulah alasan aplikasi pemantau modern wajib mendukung pengelolaan lintas merek dan
gabungan fisik-awan: agar seluruh komponen terhubung ke satu layar kendali tanpa ada
bagian yang tersembunyi.
Sekarang, mari balik situasinya:
Jika universitas tersebut sudah memakai sistem pemantau yang mendukung lintas merek
dan gabungan fisik-awan, apa hal pertama yang bisa langsung dilihat oleh teknisi di layar
kendali saat sistem pendaftaran mahasiswa mendadak lambat?
User prompt: saya tidak tahu
Response: Hal pertama yang langsung terlihat oleh teknisi adalah peta visual alur
koneksi dari ujung ke ujung dengan penanda warna merah tepat pada titik yang
bermasalah.
Sistem pemantau terpadu bekerja persis seperti peta navigasi GPS di ponsel. Saat terjadi
hambatan lalu lintas, aplikasi langsung mewarnai ruas jalan tertentu dengan warna merah
pekat tanpa meminta pengemudi memeriksa setiap persimpangan secara manual.
Di layar kendali, alur perjalanan data dari mahasiswa hingga ke sistem pendaftaran
ditampilkan dalam satu rantai keterhubungan yang utuh:
• Laptop Mahasiswa → Pemancar Wi-Fi (Merek B) → Kabel dan Switch Kampus
(Merek A) → Jalur Internet → Server Pendaftaran di Awan.
Jika kendala utama berasal dari server pendaftaran yang kewalahan melayani ribuan
pendaftar sekaligus:
•
Ikon pemancar Wi-Fi dan perangkat switch kampus tetap menyala hijau karena
perangkat keras lokal beroperasi normal.
•
Ikon server pendaftaran di awan berubah menjadi merah, lengkap dengan informasi
bahwa kapasitas memori atau prosesor server tersebut telah menyentuh batas
seratus persen.
Dalam hitungan beberapa detik, teknisi langsung mengetahui letak sumber kendala tanpa
perlu menyusuri gedung kampus atau mencurigai perangkat yang sebenarnya sehat.
Mari kita uji pemahaman alur ini:
Jika di layar kendali semua ikon perangkat kampus menyala hijau normal, tetapi garis
penghubung yang mengarah ke server awan mendadak berubah warna menjadi merah
14 of 38 10/9/26, 09:37

Firefox about:blank
berkedip, langkah mana yang harus diambil oleh teknisi: menghubungi pihak penyedia
jaringan internet/server awan, atau memanjat tangga untuk membongkar pemancar Wi-Fi
di plafon ruang kelas?
User prompt: menghubungi pihak penyedia jaringan internet/server awan
Response: Tindakan tersebut tepat sasaran. Memanjat tangga untuk membongkar
pemancar Wi-Fi hanya akan membuang tenaga dan berisiko merusak perangkat yang
sebenarnya beroperasi normal.
Karena seluruh lampu perangkat di dalam kampus menyala hijau dan garis merah
mengarah ke luar, teknisi tahu bahwa sumber kendala berada di luar kendali fisik kampus
—yaitu pada pipa saluran internet kota atau pusat data penyedia layanan awan.
Inilah nilai strategis dari sistem pemantau terpadu (hybrid multi-vendor):
•
Meniadakan Salah Tuduh: Menghindarkan teknisi dari spekulasi liar atau
membongkar perangkat keras yang sebenarnya sehat.
•
Akurasi Penanganan: Memangkas durasi pemulihan karena laporan langsung
ditujukan kepada pihak yang tepat sejak menit pertama.
•
Satu Bahasa Operasional: Tim operasional cukup menguasai satu antarmuka tanpa
perlu mempelajari puluhan aplikasi bawaan dari masing-masing merek pabrikan.
Dengan ini, seluruh fondasi kalimat pembuka PRD Anda telah terpetakan secara konkret:
1. Network Management System: Ruang kendali pusat untuk Melihat, Mengatur, dan
Mengingatkan.
2. Hybrid: Pengawasan terpadu untuk kabel fisik di gedung kantor sekaligus server
virtual di awan.
3. Multi-Vendor: Penerjemah universal yang menyatukan beragam merek perangkat
ke dalam satu layar kendali.
User prompt: baik, sekarang jelaskan apa itu monitoring reaktif berbasis polling tradisional,
observabilitas prediktif menggunakan streaming telemetry, korelasi alarm berbasis machine
learning, dan automasi pemulihan mandiri dengan pendekatan GitOps. pada kalimat "Sistem
berfokus pada peralihan dari monitoring reaktif berbasis polling tradisional ke observabilitas
prediktif menggunakan streaming telemetry, korelasi alarm berbasis machine learning, dan
automasi pemulihan mandiri dengan pendekatan GitOps.". apa itu polling tradisional,
streaming telemetry, korelasi alarm berbasis machine learning, dan gitops
15 of 38 10/9/26, 09:37

Firefox about:blank
Response: Keempat istilah tersebut menggambarkan perubahan besar dari cara lama
yang lambat dan melelahkan, menuju cara baru yang serba otomatis dan cerdas.
1. Polling Tradisional (Monitoring Reaktif)
•
Analoginya: Satpam yang setiap sepuluh menit sekali berjalan kaki keliling gedung
untuk mengetuk pintu tiap ruangan satu per satu dan bertanya: "Semua aman?".
•
Cara kerjanya: Komputer pusat yang harus aktif bertanya ke setiap perangkat
secara berkala.
•
Kelemahannya: Bersifat reaktif (menunggu terjadi dulu). Jika kebakaran terjadi
sepuluh detik setelah satpam lewat, satpam baru akan tahu hampir sepuluh menit
kemudian saat giliran keliling berikutnya tiba.
2. Streaming Telemetry (Observabilitas Prediktif)
•
Analoginya: Jam tangan pintar (smartwatch) di pergelangan tangan atlet yang terus-
menerus menyiarkan data detak jantung, kadar oksigen, dan suhu tubuh secara
langsung tanpa henti ke layar monitor pelatih.
•
Cara kerjanya: Komputer pusat tidak perlu capek bertanya. Perangkat jaringan itu
sendiri yang terus mengalirkan data kondisinya secara langsung setiap milidetik
seperti pipa air yang mengucur tanpa henti.
•
Keunggulannya: Bersifat prediktif (mencegah sebelum terjadi). Karena data
mengalir rapat, grafik akan memperlihatkan tanda bahaya lebih awal—misalnya
suhu mesin yang merangkak naik sedikit demi sedikit dari 40°C ke 70°C—sebelum
perangkat benar-benar hangus atau mati mendadak.
3. Korelasi Alarm Berbasis Machine Learning
•
Analoginya: Saklar listrik utama di rumah padam mendadak. Akibatnya, lampu,
televisi, kulkas, pendingin ruangan, dan mesin cuci seketika mati. Seseorang yang
panik akan mengira lima alat tersebut rusak bersamaan. Namun, orang yang paham
kelistrikan langsung tahu: sumber masalahnya hanya satu saklar utama di depan
rumah.
•
Cara kerjanya: Ketika satu kabel serat optik utama terputus, ratusan komputer dan
pemancar di belakangnya akan serentak menjerit dan membanjiri ruang kendali
dengan ribuan notifikasi eror.
•
Peran Machine Learning: Algoritma cerdas bertugas menganalisis tumpukan alarm
tersebut dalam hitungan milidetik, mengenali pola sebab-akibat, lalu membungkam
alarm-alarm turunan. Sistem hanya menampilkan satu pesan jelas ke teknisi: "Kabel
utama di lorong barat putus, fokuslah ke titik itu saja."
4. GitOps (Automasi Pemulihan Mandiri)
16 of 38 10/9/26, 09:37

Firefox about:blank
•
Analoginya: Buku resep resmi masakan restoran bintang lima yang tersimpan di
dalam lemari berkaca dengan riwayat stempel persetujuan dari kepala koki. Jika ada
koki magang yang salah menaruh bumbu di dapur, sistem secara otomatis mereset
racikan bumbu tersebut agar kembali persis sesuai dengan takaran di buku resep
resmi.
•
Cara kerjanya: Seluruh aturan dan setelan perangkat jaringan dicatat dalam bentuk
dokumen teks di satu lemari arsip digital bernama Git.
•
Keunggulannya: Jika ada teknisi yang salah menyetel perangkat, atau ada
gangguan yang membuat konfigurasi mesin kacau, robot sistem tidak perlu
menunggu perintah manusia. Sistem akan membaca kembali isi arsip resmi di Git,
lalu secara otomatis menimpa balik setelan perangkat tersebut hingga normal
kembali dalam hitungan detik.
Mari kita uji gabungan keempat konsep ini lewat satu skenario:
Di sebuah pusat data, pendingin ruangan tiba-tiba mati sehingga suhu salah satu
perangkat pengatur rute data perlahan naik dari 45°C menuju 85°C.
Antara Polling Tradisional dan Streaming Telemetry, cara mana yang memungkinkan
sistem menyadari bahaya ini sebelum mesin tersebut mati karena kepanasan, dan
mengapa?
User prompt: streaming telemetry
Response: Pilihan tersebut tepat: streaming telemetry.
Karena data suhu dialirkan terus-menerus tanpa jeda, sistem bisa langsung mendeteksi
kemiringan grafik kenaikan suhu—misalnya naik 2°C setiap sepuluh detik—dan
membunyikan peringatan dini saat suhu baru menyentuh angka 60°C.
Sebaliknya, pada polling tradisional yang hanya mengecek setiap lima atau sepuluh menit
sekali, sistem baru akan datang menyapa saat perangkat sudah telanjur padam terbakar
pada menit ketiga. Itulah inti dari observabilitas prediktif: mencegah kerusakan total
sebelum terjadi.
Sekarang mari kita sambungkan ke bagian akhirnya: GitOps.
Bayangkan ketika suhu mulai naik, seorang teknisi pemula panik. Dia buru-buru masuk ke
sistem lalu tidak sengaja menghapus seluruh aturan rute jaringan di perangkat tersebut
sehingga koneksi kantor langsung lumpuh total.
Berdasarkan prinsip "buku resep resmi di lemari kaca" tadi, bagaimana robot GitOps
menyelamatkan situasi tersebut tanpa perlu menunggu teknisi senior datang
memperbaiki?
17 of 38 10/9/26, 09:37

Firefox about:blank
User prompt: saya tidak tahu
Response: Robot GitOps menyelamatkan situasi dengan menjalankan mekanisme
pemulihan mandiri dalam tiga langkah otomatis:
1. Membandingkan: Robot GitOps bertugas memantau dan membandingkan antara
"setelan yang sedang aktif di perangkat" dengan "buku resep resmi di lemari kaca
(Git)".
2. Menemukan Ketidaksesuaian: Begitu teknisi pemula menghapus aturan di
perangkat, robot langsung menyadari ada kejanggalan: "Di perangkat setelannya
mendadak kosong, padahal di buku resep resmi ada seratus aturan yang sah."
3. Menimpa Kembali Secara Otomatis: Tanpa menunggu instruksi manusia, robot
langsung menyalin kembali seluruh isi resep resmi dari Git dan menimpakannya ke
perangkat tersebut.
Dalam hitungan detik, seluruh aturan rute jaringan kembali pulih seperti sedia kala, dan
koneksi kantor normal kembali sebelum pimpinan kantor menyadari ada kesalahan
manusia.
Di dunia rekayasa perangkat lunak, proses ini disebut rekonsiliasi otomatis. Git bertindak
sebagai satu-satunya sumber kebenaran mutlak (single source of truth).
Sekarang keempat konsep canggih pada ringkasan PRD sudah terangkai utuh:
•
Streaming Telemetry: Sensor yang terus mengucurkan data performa tanpa henti
untuk mendeteksi tanda bahaya sejak dini.
•
Korelasi Alarm Machine Learning: Penyaring cerdas yang membungkam ribuan
kepanikan palsu dan langsung menunjuk satu sumber masalah utama.
•
GitOps: Penjaga buku resep resmi yang otomatis memulihkan setelan perangkat jika
ada kekeliruan atau kerusakan konfigurasi.
•
Network Management System: Ruang kendali pusat yang memadukan semuanya
agar jaringan bekerja stabil dan minim campur tangan manual.
User prompt: bahas tentang Pengguna Target dan Masalah Utama, dimulai dari Network
Operations Center Tier 1 dan 2Triage insiden cepat dan visibilitas alert terkonsolidasiKewalahan
menghadapi banjir notifikasi saat terjadi kegagalan jaringan skala besar, apa itu network
operating center?
Response: Network Operations Center adalah pos komando siaga 24 jam tempat tim
18 of 38 10/9/26, 09:37

Firefox about:blank
teknisi duduk di depan dinding layar monitor besar untuk mengawasi detak jantung
seluruh jaringan perusahaan tanpa henti.
Ibarat menara pengawas lalu lintas udara di bandara, petugas di dalam ruangan
tersebut tidak menerbangkan pesawat secara langsung, tetapi mereka mengawasi radar
untuk memastikan tidak ada pesawat yang bertabrakan, mendeteksi cuaca buruk, dan
langsung mengarahkan pesawat jika terjadi keadaan darurat.
Di dalam ruangan komando ini, petugas dibagi menjadi tingkatan tugas:
•
Tier 1 (Garda Terdepan): Petugas jaga yang pertama kali melihat layar monitor atau
mengangkat telepon laporan. Tugas mereka mirip perawat triase di instalasi gawat
darurat rumah sakit: menyaring keluhan, mengecek gejala awal yang sederhana, dan
menentukan seberapa gawat situasinya.
•
Tier 2 (Teknisi Spesialis): Tim ahli yang berada di lapis kedua. Jika masalah jaringan
terlalu rumit dan tidak bisa diselesaikan oleh Tier 1 dalam beberapa menit, masalah
tersebut langsung diserahkan kepada tim Tier 2 yang memiliki keahlian teknis lebih
mendalam.
Masalah Utama: Triage dan Banjir Notifikasi
Dua istilah pada tabel PRD menggambarkan tekanan berat yang dialami tim Tier 1 dan
Tier 2 setiap hari:
1. Banjir Notifikasi saat Gangguan Besar Ketika satu kabel serat optik utama di gedung
pusat terputus, ribuan komputer, printer, dan pemancar nirkabel di ratusan ruangan akan
serentak kehilangan sinyal.
Jika sistem pemantau bekerja bodoh, layar Tier 1 akan seketika dihujani ribuan pesan
alarm merah yang berbunyi dan berkedip bersamaan dalam hitungan detik. Fenomena ini
membuat tim jaga mengalami kelelahan peringatan: mereka panik, mata lelah, dan
kebingungan mencari mana pesan kerusakan yang sebenarnya menjadi sumber biang
kerok di antara ribuan notifikasi turunan tersebut.
2. Triage Cepat dan Peringatan Terkonsolidasi Kata triage berasal dari istilah medis
untuk memilah korban bencana berdasarkan tingkat keparahan luka: mana yang sekarat
dan harus segera dioperasi, mana yang hanya luka lecet dan bisa menunggu.
Di dunia jaringan, triage cepat berarti kemampuan sistem untuk memilah keparahan
insiden dalam hitungan detik. Tim Tier 1 membutuhkan peringatan terkonsolidasi—
yaitu layar bersih yang secara otomatis menyatukan ribuan alarm tadi menjadi satu kartu
insiden tunggal yang berkata: "Kabel utama di ruang bawah tanah putus; abaikan ribuan
alarm komputer di lantai atas karena itu hanya dampak turunan."
Mari kita uji pemahaman peran ini lewat skenario nyata di ruang operasi jaringan:
19 of 38 10/9/26, 09:37

Firefox about:blank
Sebuah kantor cabang bank kehilangan koneksi internet. Petugas Tier 1 menerima
laporan tersebut, lalu mencoba menyuruh kantor cabang menyalakan ulang alat dan
memeriksa colokan kabel, tetapi koneksi tetap mati total. Setelah dicek lebih dalam,
ternyata ada kesalahan konfigurasi rute enkripsi data yang sangat rumit di dalam mesin
pengatur jaringan.
Berdasarkan pembagian tugas tadi, langkah apa yang seharusnya diambil oleh petugas
Tier 1 tersebut agar antrean laporan lain tidak menumpuk?
User prompt: dengan mengabaikan kantor cabang dan fokus pada kesalahan konfiguras rute
enksripsi data
Response: Jika petugas Tier 1 memaksakan diri membedah masalah enkripsi yang rumit,
antrean laporan di meja komando justru akan menumpuk dan menimbulkan bencana
baru.
Ibarat seorang perawat triase di instalasi gawat darurat rumah sakit: saat perawat
mendapati pasien yang membutuhkan operasi bedah saraf, perawat tersebut tidak boleh
nekat masuk ke ruang operasi untuk membedah sendiri. Jika perawat melakukannya,
pasien lain yang baru datang dengan serangan jantung di lobi rumah sakit tidak akan ada
yang melayani.
Tindakan yang tepat bagi petugas Tier 1 adalah melakukan eskalasi ke Tier 2:
1. Mencatat Temuan: Menulis laporan singkat bahwa perangkat cabang mati,
pengecekan kabel fisik dan penyalaan ulang tidak berhasil, dan ada indikasi
kerusakan pada rute enkripsi.
2. Menyerahkan Tiket Insiden: Mengirimkan berkas masalah tersebut langsung ke
meja tim spesialis Tier 2 yang memang memiliki keahlian mendalam di bidang rute
data dan enkripsi.
3. Kembali Berjaga: Petugas Tier 1 langsung kembali mengawasi layar radar utama
untuk menyaring laporan dan alarm dari kantor cabang lainnya.
Inilah alasan mengapa aplikasi Network Management System sangat krusial bagi Tier 1.
Jika sistem mampu menyajikan visibilitas alert terkonsolidasi, petugas Tier 1 tidak perlu
membuang waktu menebak-nebak atau membongkar setelan rumit. Sistem sudah
merangkum kerusakan tersebut dalam satu kartu ringkas, sehingga proses penyerahan
tugas ke tim Tier 2 bisa selesai dalam hitungan detik.
Setelah tim operasional garda depan (NOC Tier 1 dan 2), peran pengguna berikutnya pada
dokumen PRD adalah Network Engineer dan Network Architect.
Di tabel PRD tertulis masalah mereka:
20 of 38 10/9/26, 09:37

Firefox about:blank
Konfigurasi manual rawan kesalahan dan minim visibilitas topologi hybrid cloud.
Kebutuhan mereka:
Validasi konfigurasi deklaratif dan audit kepatuhan perangkat.
User prompt: Jelaskan peran Network Engineer dan Architect, apa itu konfigurasi deklaratif, dan
audit kepatuhan perangkat dengan analogi sederhana.
Response: Perbedaan peran, cara konfigurasi, dan proses audit ini dapat dipahami dengan
membayangkan pembangunan sebuah gedung pencakar langit.
1. Network Architect dan Network Engineer
•
Network Architect (Arsitek Gedung): Perancang cetak biru. Tugasnya membuat
sketsa tata letak seluruh gedung, menentukan seberapa kuat fondasi yang
dibutuhkan, menghitung kapasitas listrik untuk sepuluh tahun ke depan, dan
memilih material terbaik. Mereka tidak memegang obeng di lapangan, melainkan
memikirkan strategi besar agar seluruh infrastruktur efisien dan tahan lama.
•
Network Engineer (Mandor dan Teknisi Konstruksi): Pelaksana lapangan yang
membaca cetak biru tersebut. Merekalah yang menarik kabel ke dinding, memasang
stopkontak, menyambungkan pipa air, dan memastikan listrik di tiap lantai menyala
sesuai rancangan sang arsitek.
2. Konfigurasi Deklaratif
Cara lama menyetel perangkat jaringan disebut konfigurasi imperatif, sedangkan cara
modern disebut konfigurasi deklaratif:
•
Cara Lama (Imperatif): Seperti mendiktekan resep langkah demi langkah kepada
juru masak: "Jalan lima langkah ke kompor, ambil wajan, tuang minyak dua sendok,
tunggu satu menit, masukkan telur, bolak-balik tiga kali." Jika ada satu langkah terlewat
atau kompor sedikit berbeda posisi, masakan akan gagal total. Teknisi jaringan dulu
harus mengetik puluhan baris perintah manual satu per satu ke setiap alat. Satu
salah ketik titik koma bisa membuat internet sekantor padam.
•
Cara Modern (Deklaratif): Seperti memesan hidangan di restoran: "Saya mau seporsi
telur mata sapi setengah matang di meja nomor lima." Anda cukup menyatakan hasil
akhir yang diinginkan, lalu biarkan sistem pintar yang secara mandiri merumuskan
dan mengeksekusi langkah-langkah teknisnya agar kondisi akhir tersebut terwujud.
Dalam jaringan modern, teknisi cukup menulis dokumen singkat: "Port nomor 1 sampai 12
harus tersambung ke jaringan bagian keuangan." Sistem otomasi yang akan
menerjemahkan niat tersebut menjadi kode teknis dan menerapkannya ke perangkat
secara presisi.
21 of 38 10/9/26, 09:37

Firefox about:blank
3. Audit Kepatuhan Perangkat
Ibarat inspeksi uji kelayakan kendaraan bermotor atau razia keselamatan gedung secara
berkala.
Petugas inspeksi memeriksa setiap kendaraan menggunakan daftar standar resmi:
apakah rem berfungsi normal, apakah kaca spion lengkap, dan apakah emisi gas buang
memenuhi ambang aman. Kendaraan yang tidak memenuhi standar dilarang beroperasi
di jalan raya.
Dalam jaringan perusahaan yang memiliki ribuan alat, audit kepatuhan adalah robot
pemeriksa otomatis yang terus menyisir setiap mesin untuk mengecek:
•
Apakah masih ada switch yang memakai kata sandi bawaan pabrik seperti admin123.
•
Apakah versi sistem operasi di dalam router sudah yang paling baru dan bebas dari
celah kebocoran data.
•
Apakah ada teknisi yang iseng mengubah setelan keamanan di luar prosedur resmi.
Jika ditemukan ada satu perangkat yang menyimpang dari standar baku, sistem audit
akan langsung membunyikan tanda peringatan atau bahkan memutus sementara
perangkat tersebut dari jaringan utama.
Mari kita uji logika konfigurasi deklaratif ini lewat skenario nyata:
Seorang teknisi ingin agar lampu koridor menyala setiap jam enam sore.
•
Perintah A: "Tekan saklar ke atas, pastikan arus listrik mengalir ke kabel lampu."
•
Perintah B: "Kondisi lampu koridor harus menyala pukul enam sore."
Di antara dua perintah tersebut, mana yang menerapkan pendekatan deklaratif, dan apa
keuntungannya jika ternyata tombol saklarnya diganti dengan model tuas putar di
kemudian hari?
User prompt: Perintah B: "Kondisi lampu koridor harus menyala pukul enam sore."
Response: Tepat sekali, Perintah B.
Keuntungannya terlihat jelas saat alat fisik di lapangan diganti:
Jika memakai Perintah A ("tekan saklar ke atas"), instruksi tersebut langsung macet dan
memicu kesalahan karena tuas model baru tidak bisa ditekan ke atas. Anda terpaksa
merombak ulang seluruh buku instruksi dari awal.
Sebaliknya, pada Perintah B, instruksi tidak perlu diubah sedikit pun. Niat akhirnya tetap
22 of 38 10/9/26, 09:37

Firefox about:blank
sama: "kondisi lampu harus menyala". Sistem pintar di baliknya yang akan menyesuaikan
cara kerja fisik, entah dengan memutar tuas, menggeser tombol, atau mengirim arus
digital.
Di dunia jaringan, prinsip inilah yang membebaskan Network Engineer dan Architect dari
kerepotan syntax: mereka cukup mendefinisikan aturan secara umum satu kali saja, tanpa
perlu pusing menghafal ribuan perintah baris kode manual yang berbeda-beda untuk tiap
merek perangkat.
Peran Terakhir: Security Operations Engineer
Di tabel PRD, pengguna target ketiga adalah Security Operations Engineer dengan dua
fokus utama:
1. Visibilitas Lalu Lintas Lateral Kebanyakan sistem keamanan lama hanya menjaga
gerbang depan gedung (pintu masuk internet). Begitu ada orang yang berhasil menyusup
ke lobi, orang tersebut bisa bebas berjalan masuk ke ruang arsip, ruang direksi, atau
kamar mandi tanpa ada yang mengawasi.
•
Lalu Lintas Lateral: Aliran data antar perangkat di dalam gedung yang sama
(misalnya dari komputer bagian administrasi melompat ke komputer bagian
keuangan).
•
Peran Sistem Pemantau: Memasang kamera pengawas di setiap koridor dalam
gedung. Jika komputer bagian administrasi mendadak mengirim ribuan berkas aneh
ke komputer bagian keuangan pada jam dua dini hari, sistem langsung menandai
pergerakan mencurigakan tersebut.
2. Pemetaan Kerentanan Firmware Firmware adalah sistem operasi mini yang tertanam
langsung di dalam komponen keras perangkat jaringan.
Ibarat model kunci gembok pada pintu kamar. Suatu hari, pabrik pembuat gembok
mengumumkan bahwa kunci tipe X ternyata memiliki cacat produksi dan bisa dibuka
memakai kawat sederhana.
Tim keamanan siber membutuhkan fitur pemetaan kerentanan otomatis: aplikasi
pemantau langsung menyisir seluruh gedung dan menampilkan daftar: "Ada lima puluh
gembok tipe X di lantai tiga yang harus segera diganti kuncinya hari ini juga."
Coba bayangkan skenario ini untuk menguji prinsip keamanan jaringan:
Seorang karyawan di bagian resepsionis tidak sengaja mengunduh berkas berbahaya ke
komputernya. Komputer tersebut mulai mencoba menyusup ke komputer server tempat
penyimpanan data rahasia perusahaan di ruangan sebelah.
Antara penjaga gerbang depan (firewall luar) dan pemantau lalu lintas lateral, siapa
yang paling pertama menangkap aksi penyusupan ini, dan apa alasannya?
23 of 38 10/9/26, 09:37

Firefox about:blank
User prompt: saya tidak tahu
Response: Yang paling pertama menangkap aksi penyusupan ini adalah pemantau lalu
lintas lateral.
Alasannya sederhana: berkas berbahaya tersebut sudah berada di dalam gedung.
Satpam di gerbang depan (firewall luar) hanya bertugas memeriksa orang yang keluar-
masuk melewati pintu pagar utama. Begitu berkas berhasil lolos masuk ke komputer
resepsionis, pergerakan berikutnya terjadi sepenuhnya di dalam koridor kantor—dari
meja resepsionis menuju ruang server di sebelahnya.
Karena data mencurigakan itu tidak pernah melintasi gerbang utama ke arah jalan raya
(internet luar), satpam gerbang depan akan buta total terhadap apa yang sedang terjadi.
Sebaliknya, pemantau lalu lintas lateral bertindak seperti kamera pengawas di lorong
antar-ruangan. Sistem inilah yang langsung membunyikan tanda bahaya ketika melihat
komputer resepsionis tiba-tiba mencoba mendobrak pintu ruang server rahasia di tengah
malam.
Dengan ini, seluruh peta peran pada bagian Pengguna Target dan Masalah Utama
dalam PRD Anda sudah tuntas dibedah:
1. NOC Tier 1 dan 2: Butuh alarm yang ringkas dan terkonsolidasi agar tidak panik saat
ratusan notifikasi meledak bersamaan.
2. Network Engineer dan Architect: Butuh konfigurasi deklaratif (cukup tulis hasil
akhir) dan audit otomatis agar terbebas dari kesalahan ketik manual dan celah
keamanan.
3. Security Operations Engineer: Butuh pengawasan koridor internal (lalu lintas
lateral) dan deteksi dini celah keamanan pada komponen sistem (firmware).
User prompt: Mari kita lanjutkan membedah Bab 3 Kebutuhan Fungsional pada PRD, mulai
dari teknologi pengumpulan data seperti gNMI, SNMP, dan eBPF.
Response: Tiga teknologi pengumpulan data ini hadir karena perangkat jaringan di
lapangan memiliki usia, kemampuan komputasi, dan letak anatomi yang sangat berbeda.
1. SNMP (Surat Kuesioner Klasik)
•
Analogi: Petugas pos yang mengirimkan formulir kuesioner kertas ke kantor cabang
lama setiap beberapa menit sekali, lalu menunggu balasan surat dikirim balik.
24 of 38 10/9/26, 09:37

Firefox about:blank
•
Mekanisme: Ini adalah bahasa tertua dan paling umum di dunia jaringan. Sistem
pusat mengirim pesan tanya-jawab baku untuk membaca angka tertentu di
perangkat.
•
Peran di PRD: Berfungsi sebagai jembatan bagi perangkat warisan, seperti router
lama, switch modular gedung lama, atau access point Wi-Fi tua yang belum
mendukung teknologi modern. Perangkat ini tidak bisa diajak bicara dengan
teknologi baru, sehingga protokol lama tetap wajib disediakan agar perangkat tua
tidak menjadi titik buta.
2. gNMI (Pipa Siaran Langsung / Streaming)
•
Analogi: Saluran live streaming resolusi tinggi yang terhubung tanpa henti antara
atlet dan layar monitor pelatih.
•
Mekanisme: Alih-alih menunggu ditanya, perangkat modern membuka satu saluran
pipa data berkecepatan tinggi ke sistem pemantau dan langsung menyemburkan
metrik perubahannya dalam hitungan pecahan detik.
•
Peran di PRD: Memenuhi kebutuhan observabilitas prediktif dengan latensi sub-
detik. Protokol ini dipakai pada perangkat keras modern berkecepatan tinggi di
pusat data untuk menangkap lonjakan trafik mikro sebelum memicu kemacetan.
3. eBPF (Kamera Sinar-X di Dalam Otak Server)
•
Analogi: Sensor mikroskopis yang disuntikkan langsung ke dalam dinding pembuluh
darah untuk memantau aliran darah secara langsung dari dalam organ tubuh, bukan
sekadar menempelkan termometer di kulit luar.
•
Mekanisme: Program mini yang disematkan langsung ke dalam inti sistem operasi
(kernel) komputer server. Teknologi ini membaca paket data tepat saat diproses oleh
prosesor dan memori server, tanpa membebani performa aplikasi.
•
Peran di PRD: Memberikan visibilitas mendalam di dalam server fisik maupun
lingkungan komputasi awan. Sistem bisa mengetahui aplikasi mana di dalam server
yang membuat jaringan lambat, bahkan sebelum data tersebut menyentuh kabel
fisik.
Target Penggunaan di
Teknologi Titik Pengamatan Karakteristik Utama
PRD
Luar perangkat Lambat, sederhana, Router dan switch
SNMP
(tanya-jawab berkala) standar purba model lama
Saluran keluar (siaran Sangat cepat, efisien, Router modern & tulang
gNMI
kontinu) sub-detik punggung data
25 of 38 10/9/26, 09:37

Firefox about:blank
Target Penggunaan di
Teknologi Titik Pengamatan Karakteristik Utama
PRD
Bagian dalam sistem Menembus batas kernel Server aplikasi, server
eBPF
operasi server secara langsung awan, dan kontainer
Mari kita uji pemahaman konsep ini lewat skenario pemilihan alat:
Di sebuah pusat data, ada tiga jenis perangkat yang harus dipantau:
1. Sebuah switch jaringan lama berumur 12 tahun yang tidak memiliki sistem operasi
modern.
2. Router utama berkecepatan tinggi yang menangani ribuan transaksi per detik dan
tidak boleh terlambat dideteksi sedetik pun jika ada gangguan.
3. Server komputer yang menjalankan puluhan aplikasi pendaftaran, di mana teknisi
ingin melihat aplikasi internal mana yang memakan kuota transmisi paling besar di
dalam server tersebut.
Berdasarkan fungsi masing-masing teknologi tadi, pasangkan teknologi mana (SNMP,
gNMI, atau eBPF) yang paling tepat untuk masing-masing dari ketiga perangkat tersebut?
User prompt: 1. snmp, 2. gnmi, 3. ebpf
Response: Ketiga pasangan tersebut tepat sasaran.
• Switch lama → SNMP: Perangkat warisan yang tidak memiliki prosesor modern
hanya mampu melayani metode kuesioner tanya-jawab klasik.
• Router utama → gNMI: Jalur tulang punggung berkecepatan tinggi wajib
menggunakan siaran data langsung tanpa jeda agar degradasi koneksi tertangkap
seketika.
• Server aplikasi → eBPF: Pemantauan internal aplikasi server membutuhkan sensor
yang membaca langsung dari dalam inti sistem operasi.
AIOps dan Analisis Akar Masalah
Setelah data berhasil dikumpulkan, tantangan berikutnya ada pada bagian kedua Bab 3
PRD: AIOps dan Analisis Insiden, khususnya fitur Peredam Banjir Alarm dan Analisis
Akar Masalah Otomatis.
Ketika terjadi gangguan fisik pada satu titik vital, ratusan perangkat lain di belakangnya
akan serentak kehilangan sinyal dan memicu ratusan alarm palsu. Jika sistem
26 of 38 10/9/26, 09:37

Firefox about:blank
menampilkan semua notifikasi itu mentah-mentah, tim teknisi akan tenggelam dalam
kepanikan.
Sistem cerdas mengatasi hal ini menggunakan konsep Peta Hubungan Sebab-Akibat
(Graf Dependensi).
Analogi Pipa Air Rumah Tangga: Bayangkan instalasi pipa air di sebuah rumah:
•
Pipa utama berasal dari meteran air di halaman depan.
•
Pipa tersebut bercabang ke keran wastafel dapur, keran kamar mandi, dan keran
mesin cuci.
Jika pipa utama di meteran depan pecah, air seketika berhenti mengalir ke wastafel dapur,
kamar mandi, dan mesin cuci.
•
Sistem Pemantau Bodoh: Membunyikan tiga alarm terpisah dan mengabarkan
bahwa keran dapur rusak, keran kamar mandi rusak, dan mesin cuci rusak. Teknisi
yang panik akan membuang waktu membongkar ketiga keran tersebut satu per
satu.
•
Sistem Berbasis Graf Dependensi (AIOps): Memiliki peta struktur hubungan induk
dan anak. Sistem tahu bahwa ketiga keran tersebut posisinya berada di bawah pipa
utama depan. Karena pipa utama terdeteksi pecah lebih dulu, sistem otomatis
membungkam keluhan dari ketiga keran dan hanya memunculkan satu perintah
kerja: "Perbaiki pipa utama di meteran depan."
Dengan cara ini, ribuan alarm turunan langsung diredam, dan teknisi langsung tertuju
pada akar masalah yang sebenarnya.
Mari kita uji logika peta hubungan sebab-akibat ini:
Di sebuah kantor cabang, susunan jalurnya adalah: Kabel Internet Penyedia Jasa →
Router Gerbang Kantor → Switch Pembagi di Lantai 1 → Komputer Kasir.
Tiba-tiba kabel yang menghubungkan Router Gerbang Kantor ke Switch Pembagi di
Lantai 1 putus tersandung.
Seketika layar ruang kendali menerima dua laporan:
1. "Komputer Kasir tidak bisa memproses pembayaran."
2. "Switch Pembagi di Lantai 1 kehilangan kontak dengan Router Gerbang Kantor."
Berdasarkan prinsip hubungan induk dan anak tadi, mana yang merupakan akar
masalah utama, dan mana yang sekadar dampak turunan?
27 of 38 10/9/26, 09:37

Firefox about:blank
User prompt: akar masalah pada switch yang berdampak pada komputer kasir
Response: Analisis Anda tepat sasaran. Gangguan pada koneksi menuju switch adalah
akar masalahnya, sedangkan matinya komputer kasir hanyalah dampak turunan karena
jalur suplainya terputus.
Di ruang kendali, sistem AIOps akan langsung mengunci laporan kasir agar tidak
membuat panik teknisi. Layar monitor hanya akan menerbitkan satu tiket tugas
perbaikan: "Periksa kabel penghubung dari Router ke Switch Lantai 1", sementara status
komputer kasir cukup diberi label abu-abu (terdampak), bukan merah (rusak).
Namun, ada satu pertanyaan mendasar: Dari mana sistem pemantau bisa tahu urutan
silsilah kabel tersebut sejak awal?
Jika sebuah gedung memiliki lima ribu kabel dan ratusan switch, teknisi tentu akan
kewalahan jika harus menggambar garis sambungan kabel satu per satu secara manual.
Di sinilah peran fitur berikutnya pada Bab 3 PRD: Penemuan Topologi Otomatis.
Cara Kerja Penemuan Topologi Otomatis
Analoginya: Pesta makan malam di ruangan gelap di mana orang-orang saling berjabat
tangan dan berkenalan:
•
Setiap kali bersalaman, seseorang berbisik: "Halo, saya Budi, tangan kanan saya
sedang memegang tangan kiri Ani."
•
Petugas pencatat di sudut ruangan mendengarkan bisikan perkenalan tersebut, lalu
mencatat di buku: "Budi bergandengan dengan Ani."
•
Setelah semua orang saling menyapa, petugas pencatat sudah memegang peta
denah lingkaran siapa menggandeng siapa secara lengkap tanpa perlu mengintip
ruangan.
Di dunia jaringan, perangkat keras melakukan hal yang persis sama melalui protokol
sapaan tetangga:
•
Switch akan mengirim bisikan kecil berkala ke kabelnya: "Halo, saya Switch Lantai 1
merek Cisco, identitas saya nomor 101."
•
Router di ujung kabel menerima bisikan itu dan meneruskannya ke aplikasi
pemantau pusat.
•
Aplikasi pemantau mengumpulkan seluruh bisikan tetangga tersebut, lalu
merajutnya menjadi peta pohon jaringan hidup di layar secara otomatis.
Jika ada kabel yang dicabut lalu dipindahkan ke colokan lain oleh teknisi, peta di layar
monitor akan langsung memperbarui jalurnya sendiri dalam hitungan detik.
28 of 38 10/9/26, 09:37

Firefox about:blank
Mari kita uji logika penemuan otomatis ini lewat skenario berikut:
Seorang teknisi baru memindahkan kabel komputer kasir dari Switch Lantai 1 ke Switch
Lantai 2 karena colokan lama longgar, tetapi ia lupa melaporkan pemindahan kabel
tersebut ke ruang kendali.
Apakah aplikasi pemantau modern akan tetap menggambar komputer kasir tersambung
ke Switch Lantai 1, atau petanya akan berubah sendiri? Bagaimana sistem
mengetahuinya?
User prompt: petanya akan berubah sendiri karena peta tersebut akan otomatis memperbarui
Response: Peta di layar monitor akan langsung memperbarui garisnya sendiri secara
otomatis.
Mekanisme di balik layar berjalan melalui koordinasi dua arah:
1. Pelepasan di Switch Lama: Begitu kabel dicabut dari Switch Lantai 1, sensor
colokan mendeteksi putusnya aliran sinyal listrik. Switch Lantai 1 seketika melapor ke
sistem: "Colokan nomor lima saya sekarang kosong."
2. Pengenalan di Switch Baru: Begitu kabel ditancapkan ke Switch Lantai 2, colokan
baru langsung dialiri arus data. Switch Lantai 2 membaca nomor identitas unik
komputer kasir tersebut, lalu melapor ke sistem: "Komputer kasir sekarang
tersambung di colokan nomor delapan saya."
3. Penyusunan Ulang Peta: Aplikasi pusat menerima dua kabar tersebut, menghapus
garis penghubung lama, dan menggambar garis baru menuju Switch Lantai 2 di
kanvas layar monitor secara seketika.
Penyimpanan Data: Graf dan Deret Waktu
Perubahan bentuk peta dan aliran angka kecepatan data yang terjadi ribuan kali per detik
ini membawa kita ke Bab 5 PRD: Arsitektur Komponen dan Teknologi.
Data jaringan tidak bisa ditampung dalam tabel biasa seperti lembar kerja Excel karena
sifat datanya terbagi menjadi dua bentuk yang sangat berlawanan:
1. Database Graf (Contoh: Neo4j)
•
Bentuk Data: Titik dan garis penghubung.
•
Peran: Menyimpan struktur hubungan fisik dan silsilah keluarga jaringan—siapa
tersambung ke siapa, kabel mana yang menjadi induk, dan rute mana yang menjadi
anak. Database ini sangat lihai mencari jalur tercepat dan memetakan dampak saat
satu titik putus.
29 of 38 10/9/26, 09:37

Firefox about:blank
2. Database Deret Waktu (Contoh: ClickHouse atau VictoriaMetrics)
•
Bentuk Data: Deretan angka yang ditempeli cap waktu detik demi detik.
•
Peran: Menyimpan rekaman riwayat performa seperti grafik suhu mesin, pemakaian
prosesor, dan kecepatan kilobyte data dari menit ke menit. Database ini dirancang
khusus agar tidak jebol saat dihujani jutaan catatan angka baru setiap detik.
Mari kita uji pembagian tugas kedua tempat penyimpanan ini:
Jika sistem pemantau ingin menjawab dua pertanyaan berikut:
1. "Apakah suhu router utama pernah melompat menembus 80°C pada hari Selasa jam dua
siang minggu lalu?"
2. "Jika kabel utama di ruang bawah tanah dipotong, sebutkan sepuluh switch mana saja
yang listrik datanya akan ikut padam?"
Manakah dari kedua pertanyaan tersebut yang tugas pencariannya diserahkan ke
Database Deret Waktu, dan mana yang diserahkan ke Database Graf?
User prompt: 1. database deret waktu, 2. database graf
Response: Kedua pasangan tersebut tepat sasaran.
• Pertanyaan 1 → Database Deret Waktu: Menelusuri rekaman angka metrik yang
terikat cap waktu (timestamp) untuk melihat fluktuasi suhu historis pada jam
tertentu.
• Pertanyaan 2 → Database Graf: Menelusuri keterhubungan antar-simpul (node dan
edge) untuk memetakan rantai kegagalan dan melihat perangkat mana saja yang
jalurnya terputus jika satu simpul mati.
Dengan pembagian arsitektur penyimpanan ini, fondasi logika dari seluruh komponen
utama PRD sudah terhubung:
1. Pengumpulan Data: SNMP untuk alat lama, gNMI untuk siaran sub-detik alat baru,
dan eBPF untuk melihat jeroan server.
2. Penyimpanan: Database deret waktu untuk angka performa, database graf untuk
bentuk denah kabel dan ketergantungan antar-alat.
3. Analisis Cerdas: Mesin AIOps membaca database graf untuk meredam banjir
notifikasi dan langsung menunjuk satu akar masalah.
4. Penyelamatan & Pemulihan: Mengalihkan rute saat kabel goyah, dan
memanfaatkan GitOps untuk mengunci serta memulihkan konfigurasi resmi secara
30 of 38 10/9/26, 09:37

Firefox about:blank
otomatis.
Bagian yang tersisa di dalam PRD kini berfokus pada tolok ukur kualitas sistem:
Kebutuhan Non-Fungsional (seperti ketahanan menampung 1 juta data per detik) dan
Metrik Keberhasilan (seperti MTTD dan MTTR).
User prompt: Kebutuhan Non-Fungsional
Response: Jika kebutuhan fungsional menjawab "apa saja fitur yang bisa dilakukan
aplikasi", maka kebutuhan non-fungsional menjawab "seberapa tangguh, cepat, dan
aman aplikasi tersebut saat bekerja".
Ibarat sebuah mobil: kebutuhan fungsionalnya adalah bisa maju, mundur, belok, dan
mengerem. Kebutuhan non-fungsionalnya adalah mobil tersebut harus mampu melesat
hingga 200 km/jam tanpa bergetar, memiliki bodi baja anti-peluru, dan tidak boleh mogok
meski dipakai melintasi gurun pasir selama 24 jam non-stop.
Di dalam PRD aplikasi Network Management System, kebutuhan non-fungsional dibagi
menjadi dua pilar utama:
1. Performa dan Skalabilitas (Kapasitas Menahan Beban Raksasa)
•
Menelan 1 Juta Metrik per Detik dengan Jeda <500 Milidetik:
◦
Analoginya: Pintu loket tiket konser akbar. Jika ada 1 juta penonton masuk
bersamaan setiap detik, loket tidak boleh macet atau membuat antrean
mengular. Pemeriksaan tiket harus selesai dalam waktu kurang dari setengah
kedipan mata (setengah detik).
◦
Di Jaringan: Ribuan switch dan router terus memuntahkan angka kecepatan,
suhu, dan paket data secara serentak. Sistem pemantau wajib mampu
menampung dan mencatat tsunami angka tersebut tanpa ada satu data pun
yang tercecer.
•
Pencarian Dasbor <1 Detik untuk Riwayat 30 Hari:
◦
Saat teknisi ingin melihat grafik performa jaringan sebulan ke belakang untuk
mencari pola kerusakan, grafik di layar harus langsung muncul seketika tanpa
tanda memuat (loading) yang berputar-putar lama.
•
Penskalaan Horizontal Tanpa Titik Lemah Tunggal (No Single Point of Failure):
◦
Analoginya: Mempekerjakan tim kuli angkut barang ketimbang satu orang
raksasa. Jika beban barang bertambah dua kali lipat, kita cukup menambah
jumlah kuli di tim tersebut. Dan yang terpenting: jika ada satu kuli yang
mendadak pingsan, kuli lainnya langsung membagi beban barang tersebut
31 of 38 10/9/26, 09:37

Firefox about:blank
sehingga pekerjaan tidak berhenti sedetik pun.
◦
Di Jaringan: Server pemantau dibuat berjejer secara paralel. Jika satu mesin
server pemantau mati terbakar, server rekannya di sebelahnya langsung
mengambil alih tugas pemantauan secara otomatis tanpa disadari oleh
pengguna.
2. Keamanan dan Kepatuhan (Benteng Pertahanan Sistem)
•
Hak Akses Granular (Role-Based Access Control / RBAC):
◦
Analoginya: Kartu akses kamar hotel bertingkat. Tamu biasa hanya bisa
membuka pintu kamarnya sendiri, staf kebersihan bisa membuka kamar di
lantai tiga, tetapi hanya kepala keamanan yang memegang kartu induk untuk
membuka ruang brankas dan ruang genset utama.
◦
Di Jaringan: Teknisi magang mungkin hanya diizinkan melihat layar grafik
status (hanya membaca), teknisi senior boleh menyetel rute jaringan, tetapi
hanya arsitek utama yang diizinkan mematikan atau menyalakan ulang router
inti perusahaan.
•
Enkripsi Menyeluruh (TLS 1.3 saat Mengalir & AES 256 saat Tersimpan):
◦
Analoginya: Surat rahasia negara. Saat kurir mengantar surat di jalan raya,
surat tersebut dimasukkan ke dalam koper baja antipeluru yang digembok (TLS
1.3). Begitu sampai di kantor tujuan, koper tersebut disimpan di dalam ruang
bawah tanah berpintu baja setebal satu meter (AES 256). Siapa pun yang
mencoba mencegat di jalan atau mencuri koper tidak akan bisa membaca
isinya.
•
Log Audit Tak Terubah (Immutable Audit Log):
◦
Analoginya: Kotak hitam (black box) di pesawat terbang yang ditulis
menggunakan pahatan batu karang. Tidak ada penghapus, tidak ada tombol
hapus (delete), dan tintanya tidak bisa luntur.
◦
Di Jaringan: Setiap kali ada teknisi yang menekan tombol setelan atau
mengubah satu angka saja, tindakan tersebut langsung terpahat permanen
beserta jam, menit, detik, dan nama pelakunya. Bahkan administrator tertinggi
pun tidak memiliki akses untuk menghapus rekaman riwayat tersebut. Ini
mencegah oknum nakal yang sengaja merusak jaringan lalu mencoba
menghapus jejak kejahatannya.
Mari kita uji pemahaman ini lewat satu skenario pengujian ketahanan:
Sebuah perusahaan besar sedang memantau ribuan cabang. Tiba-tiba salah satu dari lima
komputer server pemantau pusat mati total karena tersambar petir.
32 of 38 10/9/26, 09:37

Firefox about:blank
Berdasarkan prinsip Penskalaan Horizontal Tanpa Titik Lemah Tunggal, apa yang akan
terjadi pada layar pantauan teknisi di ruang operasi: apakah seluruh sistem pemantau
akan mati padam total, atau layar tetap menyala normal? Dan bagaimana nasib data yang
seharusnya diurus oleh server yang tersambar petir tadi?
User prompt: saya tidak tahu
Response: Layar monitor teknisi akan tetap menyala normal tanpa gangguan, dan
seluruh tugas pengolahan data dari server yang mati akan langsung diambil alih secara
otomatis oleh empat server yang tersisa.
Ibarat lima orang kasir di sebuah toserba yang sedang melayani antrean panjang pembeli.
Jika satu kasir mendadak pingsan, toserba tersebut tidak akan langsung menutup pintu
dan membubarkan pembeli. Keempat kasir yang masih berdiri akan segera membagi
antrean pelanggan dari kasir yang pingsan ke meja kasir mereka masing-masing.
Pelayanan belanja tetap berjalan lancar, hanya beban kerja tiap kasir yang tersisa sedikit
bertambah.
Di dalam sistem pemantau modern, mekanisme ini bekerja melalui tiga langkah otomatis:
1. Deteksi Cepat: Empat server yang sehat menyadari rekan mereka berhenti
mengirim sinyal detak jantung dalam hitungan milidetik.
2. Pengalihan Jalur: Komponen pengatur beban (load balancer) langsung
membelokkan kiriman data dari cabang-cabang yang tadinya dipegang oleh server
yang mati ke empat server yang masih hidup.
3. Pemberitahuan Internal: Layar ruang operasi tidak mengalami padam data,
melainkan hanya memunculkan satu baris catatan kecil di sudut layar: "Server nomor
lima mati, sisa empat server beroperasi menampung beban seratus persen."
Inilah alasan prinsip ini disebut tanpa titik lemah tunggal (no single point of failure). Tidak
boleh ada satu pun komponen yang jika rusak, bisa meruntuhkan keseluruhan sistem.
Dengan selesainya bab Kebutuhan Non-Fungsional, bab terakhir di dalam PRD ini adalah
Metrik Keberhasilan Operasional yang mengukur efektivitas sistem menggunakan dua
istilah: MTTD dan MTTR.
•
MTTD (Mean Time to Detect): Rata-rata waktu yang dibutuhkan sistem untuk
menyadari adanya kerusakan sejak detik pertama kerusakan itu terjadi.
•
MTTR (Mean Time to Resolve): Rata-rata waktu yang dibutuhkan tim teknisi untuk
memperbaiki kerusakan tersebut sampai jaringan pulih seratus persen.
Mari kita tinjau skenario ini:
33 of 38 10/9/26, 09:37

Firefox about:blank
Kabel internet kantor putus pada pukul 08.00 pagi.
•
Aplikasi pemantau baru membunyikan alarm pada pukul 08.10.
•
Teknisi selesai menyambung kabel dan internet kembali menyala pada pukul 08.40.
Berapa lama durasi waktu deteksi (MTTD) dan berapa lama durasi waktu perbaikan
(MTTR) pada kejadian tersebut?
User prompt: mttd = 10 menit, mttr = 30 menit
Response: Perhitungan durasi kejadian tersebut ditinjau dari dua titik pengukuran:
1. Waktu Deteksi: Dihitung dari saat insiden fisik pertama kali terjadi hingga sistem
membunyikan tanda bahaya:
08.10 − 08.00 = 10 menit
2. Waktu Resolusi: Dihitung dari saat teknisi menerima alarm hingga layanan kembali
beroperasi normal:
08.40 − 08.10 = 30 menit
Jika dihitung dari total durasi pemadaman jaringan sejak awal hingga tuntas, waktu
pemulihan menyeluruh memakan waktu 40 menit. Penghitungan 10 menit untuk deteksi
dan 30 menit untuk resolusi aktif teknisi adalah tepat.
Peran Fitur PRD dalam Memangkas Angka Tersebut
Angka 40 menit pemadaman di atas adalah potret nyata dari sistem pemantau tradisional
yang lambat. Sasaran utama dari PRD Network Management System modern adalah
memangkas kedua metrik tersebut secara radikal:
•
Streaming Telemetry Memangkas Deteksi: Mengganti metode polling sepuluh
menit dengan siaran telemetri sub-detik membuat waktu deteksi turun dari 10 menit
menjadi kurang dari 3 detik. Sistem langsung tahu adanya anomali saat detak
jantung pertama hilang.
•
AIOps dan Database Graf Memangkas Investigasi: Teknisi tidak perlu membuang
waktu 20 menit menyusuri ribuan alarm palsu dari perangkat cabang. Sistem
langsung menyajikan satu kartu tugas dengan penunjuk titik putus yang presisi.
•
Pengalihan Otomatis Menghilangkan Waktu Tunggu: Pengendali rute cadangan
langsung aktif dalam hitungan milidetik saat jalur utama mati. Bagi pengguna akhir,
gangguan layanan terasa mendekati nol detik meskipun kabel fisik baru selesai
disambung oleh teknisi pada pukul 08.40.
34 of 38 10/9/26, 09:37

Firefox about:blank
Seluruh bab fondasi PRD—mulai dari peran pengguna, arsitektur data, kebutuhan
fungsional, kebutuhan non-fungsional, hingga tolok ukur operasional—kini telah dibedah
secara utuh dari hulu ke hilir.
User prompt: Gabungkan seluruh pembahasan tadi menjadi satu dokumen Product
Requirement Document (PRD) yang rapi, profesional, dan siap dipresentasikan.
Response: Product Requirement
Document (PRD)
Next-Generation Network Management System (NMS)
1. Ringkasan Eksekutif dan Visi Produk
Sistem ini dirancang untuk menggantikan pemantauan reaktif berbasis polling tradisional
dengan platform observabilitas prediktif, terotomatisasi, dan cerdas. Platform melayani
infrastruktur jaringan gabungan fisik dan komputasi awan (hybrid) lintas merek perangkat
keras (multi-vendor) pada skala menengah hingga enterprise.
Sistem bertumpu pada tiga pilar operasional:
•
Melihat: Observabilitas menyeluruh dari transmisi fisik kabel hingga lapisan inti
sistem operasi server.
•
Mengatur: Pemulihan kendali mandiri dan automasi konfigurasi berbasis
rekonsiliasi status target.
•
Mengingatkan: Penapisan alarm cerdas berbasis graf relasi untuk menunjuk akar
masalah secara presisi.
2. Pengguna Sasaran dan Analisis Masalah
Peran
Masalah Utama Kebutuhan Solusi Sistem
Pengguna
Kelelahan notifikasi akibat Peringatan terkonsolidasi, isolasi
NOC Tier 1 dan banjir ribuan alarm saat kabel alarm turunan, serta kartu insiden
Tier 2 utama putus; proses triase tunggal yang menunjukkan titik
lambat. kegagalan fisik.
35 of 38 10/9/26, 09:37

Firefox about:blank
Peran
Masalah Utama Kebutuhan Solusi Sistem
Pengguna
Kesalahan ketik sintaks Validasi konfigurasi deklaratif,
Network
konfigurasi manual; minimnya pemetaan topologi otomatis, dan
Engineer &
visibilitas topologi hybrid lintas audit kepatuhan setelan perangkat
Architect
merek. berkala.
Ketiadaan pemantauan lalu Visibilitas lalu lintas internal antar-
Security
lintas data lateral antarsimpul; perangkat dan deteksi dini celah
Operations
inventarisasi kerentanan keamanan pada komponen sistem
Engineer
perangkat manual. (firmware).
3. Kebutuhan Fungsional
Modul 1: Penyerapan Data dan Observabilitas
•
Pengumpulan Data Warisan: Menyediakan protokol penarikan berkala berbasis
SNMP untuk router, switch lama, dan perangkat nirkabel yang belum mendukung
streaming modern.
•
Siaran Telemetri Sub-Detik: Mengintegrasikan gNMI (gRPC Network Management
Interface) untuk mengalirkan data kesehatan antarmuka dan lonjakan beban trafik
secara kontinu dengan jeda di bawah satu detik.
•
Observabilitas Lapisan Inti Server: Menjalankan sensor eBPF langsung di dalam
kernel sistem operasi guna memantau interaksi aplikasi server, latensi soket, dan
anomali koneksi internal tanpa membebani prosesor.
Modul 2: Pemetaan Topologi Dinamis dan Graf Ketergantungan
•
Penemuan Otomatis Multi-Protokol: Menemukan simpul dan relasi fisik antar-
perangkat secara mandiri menggunakan protokol pengenalan tetangga jaringan.
•
Penyusunan Pohon Ketergantungan: Mengonstruksi hierarki silsilah jaringan dari
tingkat gerbang internet, tulang punggung pusat data, switch distribusi, hingga ke
terminal kerja pengguna akhir secara real-time.
Modul 3: Manajemen Insiden dan Analisis Akar Masalah (AIOps)
•
Peredam Kepanikan Alarm: Saat simpul induk terputus, sistem membungkam
seluruh alarm keterputusan dari ratusan simpul turunan di bawahnya.
•
Penyajian Kartu Insiden Presisi: Menampilkan satu tiket kerja yang membedakan
antara perangkat yang mengalami kerusakan riil (root cause) dan perangkat yang
36 of 38 10/9/26, 09:37

Firefox about:blank
sekadar kehilangan sinyal pasokan (impacted).
Modul 4: Automasi Jaringan dan Rekonsiliasi GitOps
•
Sumber Kebenaran Tunggal (Single Source of Truth): Seluruh aturan rute dan
alokasi port disimpan sebagai dokumen teks deklaratif dalam repositori Git yang
terlacak riwayat versinya.
•
Pemulihan Mandiri Otomatis: Agen rekonsiliasi membandingkan konfigurasi aktif
perangkat dengan status target di repositori Git secara terus-menerus. Jika terjadi
ketidaksesuaian atau kekeliruan operator, konfigurasi perangkat ditimpa ulang ke
kondisi resmi dalam hitungan detik.
•
Logika Peredam Flapping: Menyediakan jeda waktu tunggu kestabilan sebelum
mengembalikan arus lalu lintas data ke jalur utama untuk mencegah data bolak-balik
saat kabel fisik masih goyah.
4. Arsitektur Data
Sistem membagi beban penanganan data ke dalam dua mesin penyimpanan
terspesialisasi:
[ Lapisan Perangkat & Server ]
│ (SNMP) │ (gNMI) │ (eBPF)
▼ ▼ ▼
[ Pipeline Penyerapan & Pemrosesan Data ]
│ │
▼ ▼
[ Database Deret Waktu ] [ Database Graf ]
- Metrik performa - Peta topologi fisik
- Riwayat kapasitas CPU - Relasi dependensi induk-anak
- Grafik utilisasi kabel - Jalur penjalaran kegagalan
•
Database Deret Waktu (VictoriaMetrics / ClickHouse): Mengelola miliaran titik
data numerik berstempel waktu, seperti fluktuasi temperatur, beban pemrosesan,
dan volume paket per detik.
•
Database Graf (Neo4j): Menyimpan model topologi jaringan dalam bentuk simpul
(nodes) dan relasi (edges) untuk mendukung analisis penelusuran dampak kegagalan
struktural secara instan.
5. Kebutuhan Non-Fungsional
Performa dan Skalabilitas
•
Daya Tampung: Mampu menerima dan mengolah minimal 1.000.000 titik metrik per
detik dengan latensi pemrosesan di bawah 500 milidetik.
37 of 38 10/9/26, 09:37

Firefox about:blank
•  Respons Kueri Dasbor: Waktu muat visualisasi riwayat performa 30 hari terakhir
tidak boleh melebihi 1 detik.
•
Ketersediaan Tinggi: Arsitektur terdistribusi horizontal tanpa titik kegagalan
tunggal (no single point of failure), dengan target ketersediaan layanan minimal
99,99%.
Keamanan dan Tata Kelola
•  Kontrol Akses Berbasis Peran (RBAC): Pemisahan hak akses berbutir halus
(granular) antara hak pantau, hak eksekusi perubahan konfigurasi, dan hak
administrator sistem.
•
Enkripsi Komprehensif: Menerapkan enkripsi TLS 1.3 untuk seluruh data yang
melintas di jaringan dan AES-256 untuk data yang tersimpan di media penyimpanan.
•  Log Audit Tak Terubah (Immutable Audit Trail): Setiap perubahan setelan jaringan,
penanganan alarm, dan akses pengguna dicatat secara permanen dan tidak dapat
dimanipulasi oleh akun mana pun.
6. Metrik Keberhasilan Operasional
Tolok Ukur
| Indikator Kinerja |     | Target Platform NMS Baru |
| ----------------- | --- | ------------------------ |
Tradisional
5 hingga 15 menit
Mean Time to
|     | (bergantung siklus | < 3 detik (berbasis streaming telemetry) |
| --- | ------------------ | ---------------------------------------- |
Detect (MTTD)
polling)
< 15 menit untuk penanganan manual,
Mean Time to
|     | 30 hingga 60 menit | sub-detik untuk pemulihan GitOps |
| --- | ------------------ | -------------------------------- |
Resolve (MTTR)
otomatis
≥ 85% alarm turunan berhasil
| Tingkat Reduksi | 0% (seluruh alarm  |                                        |
| --------------- | ------------------ | -------------------------------------- |
| Alarm Bising    | meledak bersamaan) | diredam oleh model graf                |
| Tingkat         |                    | Prediktif terhadap kehabisan kapasitas |
Reaktif (menunggu
| Keberhasilan |     | dan kenaikan suhu ekstrem sebelum |
| ------------ | --- | --------------------------------- |
laporan pengguna)
| Deteksi Dini |     | perangkat mati |
| ------------ | --- | -------------- |
38 of 38 10/9/26, 09:37
