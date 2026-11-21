# Tugas 9 (Pekan 9) — Cloud Delivery & Deployment Model

**Materi terkait:** Karakteristik esensial cloud computing (NIST SP 800-145), Deployment Model (Public/Private/Hybrid/Community), Service Model (IaaS/PaaS/SaaS).

> Bagian 2 mata kuliah ini murni **analisis & desain di atas kertas** — tidak ada biaya cloud sama sekali, karena tugas berhenti di tahap rekomendasi, bukan implementasi nyata (implementasi baru mulai Tugas 11).

## Studi Kasus

Pemerintah ingin membangun **Sistem Data Kependudukan** (nama, NIK, alamat, data biometrik) yang:
- Sangat **rahasia** (data pribadi warga negara, diatur UU Perlindungan Data Pribadi).
- Diakses oleh **ribuan instansi** (dukcapil daerah, kepolisian, bank untuk verifikasi KYC) dengan pola akses yang **tidak rata** (lonjakan saat musim pemilu/pendaftaran sekolah).
- Butuh **skalabilitas** agar tidak downtime saat lonjakan akses, tapi **tidak boleh** datanya berada di infrastruktur yang tidak terkendali penuh oleh negara.

## Tugas Kelompok

1. **Rekomendasikan Deployment Model** (Public / Private / Hybrid / Community Cloud) yang paling tepat untuk sistem ini. Jelaskan alasannya berdasarkan **karakteristik NIST** (on-demand self-service, broad network access, resource pooling, rapid elasticity, measured service) dikaitkan dengan kebutuhan kerahasiaan vs skalabilitas.

    Jawaban:

    Deployment Model yang paling tepat untuk sistem ini adalah Hybrid Cloud. Seluruh data inti kependudukan meliputi nama, NIK, alamat, hingga data biometrik disimpan di Private Cloud yang dikendalikan penuh negara (memenuhi UU Perlindungan Data Pribadi), sedangkan Public Cloud (dengan server di Indonesia) hanya dipakai untuk bagian yang tidak menyimpan data rahasia, seperti portal cek status, dan menjadi tempat kapasitas tambahan saat akses melonjak.

    Hybrid Cloud dipilih karena sistem data kependudukan memiliki dua kebutuhan yang saling bertabrakan, yaitu kerahasiaan dan skalabilitas. Sistem ini membutuhkan kerahasiaan tingkat tinggi karena NIK dan data biometrik merupakan data pribadi sensitif yang tidak boleh berada di infrastruktur di luar kendali penuh negara untuk mencegah terjadinya penyalahgunaan serta mematuhi UU Perlindungan Data Pribadi (UU PDP). Di sisi lain, skalabilitas tinggi dibutuhkan karena sistem diakses oleh ribuan instansi seperti Dukcapil daerah, kepolisian, dan perbankan untuk verifikasi KYC dengan lonjakan trafik drastis saat musim pemilu maupun pendaftaran sekolah sehingga layanan tidak boleh down. Dengan memisahkan penyimpanan data sensitif di Private Cloud dan mengalirkan beban puncak non-sensitif ke Public Cloud, negara tetap memegang kendali penuh atas data warga tanpa mengorbankan ketersediaan layanan.

    Alasan berdasarkan karakteristik NIST:
    | Karakteristik NIST | Kerahasiaan | Skalabilitas | Penerapan di Hybrid |
    |---|---|---|---|
    | On-demand self-service (pengguna menyiapkan layanan sendiri tanpa lewat petugas) | Akses ke data warga tidak boleh diberikan sembarangan. | Dengan ribuan instansi pengguna, proses pendaftaran dan penyiapan layanan secara manual akan menjadi hambatan, sehingga harus bisa dilakukan secara mandiri. | Instansi mendaftar dan mendapatkan akses uji coba (sandbox) secara mandiri. Akses ke data asli baru aktif setelah Dukcapil memverifikasi perjanjian kerja sama (PKS) dan memberikan otorisasi. |
    | Broad network access (layanan bisa diakses lewat jaringan dari berbagai tempat) | Jika data terlalu terbuka, berisiko menyebabkan kebocoran nasional. | Pengguna sistem, seperti bank, kepolisian, dan dukcapil daerah, tersebar di seluruh Indonesia, sehingga layanan harus dapat dijangkau melalui jaringan dari berbagai lokasi. | Akses luas tapi tetap terkontrol, jadi instansi masuk lewat API Gateway yang memeriksa identitas kedua pihak (mTLS) dan lewat jaringan khusus pemerintah. Hanya portal status warga yang terbuka ke internet, dan portal tersebut tidak menyimpan data rahasia. |
    | Resource pooling (sumber daya dipakai bersama oleh banyak pengguna) | Berbagi infrastruktur dengan pihak di luar kendali negara memperbesar risiko kebocoran data. | Kapasitas yang dipakai bersama dapat dimanfaatkan bergantian oleh banyak instansi, sehingga tidak ada kapasitas yang menganggur dan penggunaannya lebih efisien. | Pooling hanya di Private Cloud milik negara, dipakai bersama antarinstansi pemerintah, dengan isolasi per-instansi (kuota dan kunci terpisah). Data biometrik ditaruh di kelompok sumber daya yang terpisah. |
    | Rapid elasticity (kapasitas bisa naik-turun cepat sesuai kebutuhan) | Kapasitas tambahan tidak boleh mengorbankan kerahasiaan, sehingga data rahasia tidak boleh dipindahkan ke luar Private Cloud. | Akses dapat melonjak berkali lipat saat pemilu dan pendaftaran sekolah, sehingga kapasitas harus bisa ditambah dengan cepat. | Beban normal ditangani Private Cloud. Saat lonjakan, hanya bagian non-sensitif (portal cek status, cache, dan antrean) yang melebar ke Public Cloud, sementara database inti dilindungi cache, antrean, dan pembatasan laju (rate limiting). |
    | Measured service (pemakaian diukur dan dicatat) | Pencatatan pemakaian menunjukkan siapa mengakses NIK siapa, kapan, dan untuk tujuan apa, sehingga berfungsi sebagai alat kontrol kerahasiaan. | Data pemakaian membantu memprediksi kebutuhan kapasitas, terutama menjelang lonjakan, serta menghitung biaya pemakaian tiap instansi.| Setiap panggilan API dicatat, dan tiap instansi diberi batas jumlah permintaan (kuota). Pola yang tidak wajar, misalnya mengambil NIK secara berurutan dalam jumlah besar, akan terdeteksi lalu dibatasi. |

2. **Rekomendasikan Service Model** (IaaS / PaaS / SaaS) yang tepat untuk komponen-komponen berbeda dari sistem ini (boleh berbeda service model untuk komponen berbeda, mis. database inti vs portal publik untuk pengecekan status).

    Jawaban:

    Sistem ini sebaiknya menggunakan service model yang berbeda untuk setiap komponen. Semakin sensitif data yang ditangani, semakin rendah level abstraksi yang dipilih, supaya kendali negara lebih besar. Sebaliknya, semakin umum sebuah komponen dan semakin sedikit ia menyentuh data warga, semakin tinggi level abstraksinya.

    IaaS hanya menyediakan infrastruktur, sehingga pengelola mengurus sistem operasi sampai aplikasinya. Level abstraksinya paling rendah dan kendalinya paling besar. PaaS menyediakan platform siap pakai, sehingga pengelola cukup mengurus aplikasi dan konfigurasinya. Sementara itu, SaaS menyediakan aplikasi yang sudah jadi dan tinggal dipakai, sehingga level abstraksinya paling tinggi dan kendalinya paling kecil.

    Berikut komponen sistem dan service model yang direkomendasikan:
    | Komponen | Lokasi | Service Model | Alasan |
    |---|---|---|---|
    | Database inti kependudukan (termasuk biometrik) | Private Cloud | IaaS | Menyimpan data pribadi yang paling rahasia (dilindungi UU PDP), sehingga negara perlu mengendalikan penuh sistem operasi, pengamanan, enkripsi, dan kuncinya tanpa ketergantungan pada lapisan abstraksi pihak ketiga. |
    | API verifikasi NIK | Private Cloud | PaaS | Komponen ini dipakai ribuan instansi dan bank. Platform siap pakai memudahkan penambahan kapasitas saat akses melonjak, sementara tim cukup mengurus aplikasi dan konfigurasinya. |
    | Portal cek status warga | Public Cloud (server di Indonesia) | IaaS | Bebannya bisa melonjak tajam pada waktu tertentu dan hanya menampilkan status terbatas. Menggunakan IaaS di Public Cloud memberi tim kontrol penuh atas konfigurasi keamanan, caching/antrean, serta aturan auto-scaling infrastruktur tanpa menyimpan data rahasia. |
    | Email dan ticketing pegawai | Penyedia layanan luar (vendor SaaS) | SaaS | Aplikasi umum yang tidak berisi data kependudukan, jadi cukup memakai aplikasi siap pakai. Aplikasi hanya digunakan untuk koordinasi internal dan administrasi umum pegawai Dukcapil. Data pribadi warga tidak boleh dilampirkan atau diproses pada layanan ini agar tidak melanggar UU PDP. |

3. Jelaskan **minimal 2 risiko spesifik** dari pilihan kalian dan mitigasinya (mis. jika Hybrid Cloud dipilih: risiko sinkronisasi data antara private & public segment).
4. Bandingkan singkat dengan **satu alternatif yang kalian tolak** — kenapa alternatif itu kurang tepat (mis. kenapa Public Cloud murni ditolak, atau kenapa Private Cloud murni dianggap kurang skalabel).

    ### Skenario Tandingan

    Skenario: Bagaimana jika instansi daerah tidak punya infrastruktur private cloud sendiri?

    Jawaban:

    Instansi daerah tidak perlu membangun private cloud sendiri. Private Cloud dalam sistem ini adalah platform nasional milik negara yang dipakai bersama antarinstansi pemerintah (resource pooling). Dukcapil daerah, kepolisian, dan instansi lain cukup mengakses layanan melalui API Gateway dengan identitas dua arah (mTLS) dan jaringan khusus pemerintah, sehingga data inti kependudukan tetap tersimpan di pusat dan tidak tersebar di server daerah.

    Pendekatan ini memiliki keuntungan, yaitu kualitas keamanan seragam, karena data tidak tersebar di banyak server daerah yang tingkat pengamanannya berbeda-beda. Biaya yang dikeluarkan juga lebih hemat, karena daerah tidak perlu membeli server dan merekrut tenaga ahli sendiri. Yang terakhir, akses tetap terkontrol, karena tiap instansi mendapat hak dan kuota sesuai kebutuhannya, dan setiap akses tercatat.

    Untuk daerah dengan jaringan kurang stabil, petugas memakai antrean luring (offline queue), yaitu permintaan disimpan sementara di perangkat dan dikirim otomatis ketika jaringan pulih. Data yang tersimpan sementara hanya data minimal dan terenkripsi, lalu dihapus setelah data terkirim. 

## Struktur Submission

```
tugas-09-cloud-delivery-deployment/
├── README.md      # Analisis lengkap (isi dengan poin 1-4 di atas)
├── JURNAL.md
└── bukti/          # (opsional) diagram pendukung
```

## Rubrik Penilaian (Tugas 9)

| Komponen | Bobot | Kriteria |
|---|---|---|
| Ketepatan rekomendasi deployment model | 30% | Argumen berbasis karakteristik NIST, bukan sekadar "private cloud lebih aman" tanpa detail |
| Ketepatan rekomendasi service model | 25% | Pemetaan komponen sistem ke IaaS/PaaS/SaaS masuk akal |
| Analisis risiko & mitigasi | 25% | Risiko spesifik ke konteks data kependudukan (bukan risiko generik) |
| Perbandingan dengan alternatif yang ditolak | 20% | Argumen penolakan jelas dan berdasar |

## Batasan Penggunaan AI (Level 2)

Kebijakan **Level 2 (AI Assisted Idea Generation & Structuring)** berlaku — lihat [`../RUBRIK-UMUM.md`](../RUBRIK-UMUM.md). Boleh bertanya ke AI gambaran umum karakteristik NIST/deployment model untuk brainstorming; **tidak boleh** meminta AI menuliskan rekomendasi/analisis akhir yang tinggal ditempel ke `README.md`. Catat pemakaian AI di "Log Penggunaan AI" pada `JURNAL.md`.

- Jawaban yang terasa seperti "template" (bisa dipakai untuk sistem cloud apa saja tanpa menyebut spesifik kependudukan/data pribadi/regulasi) akan dinilai rendah pada komponen ketepatan rekomendasi.
- `README.md` wajib membahas minimal satu skenario tandingan yang kalian buat sendiri (mis. "bagaimana jika instansi daerah tidak punya infrastruktur private cloud sendiri?") beserta jawabannya.
