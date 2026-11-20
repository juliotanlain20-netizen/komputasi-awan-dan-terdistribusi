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

    Deployment model yang paling tepat untuk sistem ini adalah Hybrid Cloud. Data inti kependudukan disimpan di Private Cloud milik negara, sedangkan Public Cloud (server di Indonesia) hanya dipakai untuk bagian yang tidak menyimpan data rahasia, seperti portal cek status, dan menjadi tempat kapasitas tambahan saat akses melonjak.

    Hybrid Cloud dipilih karena sistem data kependudukan memiliki dua kebutuhan yang saling bertabrakan, yaitu kerahasiaan dan skalabilitas. Sistem ini membutuhakn kerahasiaan karena NIK dan data biometrik adalah data pribadi yang sangat sensitif dan tidak boleh berada di infrastruktur yang tidak dikendalikan penuh oleh negara. Sedangkan, skalabilitas dibutuhkan karena akses akan melonjak saat pemilu dan pendaftaran sekolah, dan sistem tidak boleh down. Hybrid Cloud memenuhi kedua kebutuhan tersebut.

    Alasan berdasarkan karakteristik NIST:
    | Karakteristik NIST | Kerahasiaan | Skalabilitas | Penerapan di Hybrid |
    |---|---|---|---|
    | On-demand self-service (pengguna menyiapkan layanan sendiri tanpa lewat petugas) | Akses ke data warga tidak boleh diberikan sembarangan. | Dengan ribuan instansi pengguna, proses pendaftaran dan penyiapan layanan secara manual akan menjadi hambatan, sehingga harus bisa dilakukan secara mandiri. | Instansi mendaftar dan mendapat akses uji coba (sandbox) secara mandiri. Akses ke data asli baru aktif setelah Dukcapil memverifikasi perjanjian kerja samanya. |
    | Broad network access (layanan bisa diakses lewat jaringan dari berbagai tempat) | Jika data terlalu terbuka, berisiko menyebabkan kebocoran nasional. | Pengguna sistem, seperti bank, kepolisian, dan dukcapil daerah, tersebar di seluruh Indonesia, sehingga layanan harus dapat dijangkau melalui jaringan dari berbagai lokasi. | Akses luas tapi tetap terkontrol, jadi instansi masuk lewat API Gateway yang memeriksa identitas kedua pihak (mTLS) dan lewat jaringan khusus pemerintah. Hanya portal status warga yang terbuka ke internet, dan portal tersebut tidak menyimpan data rahasia. |
    | Resource pooling (sumber daya dipakai bersama oleh banyak pengguna) | Berbagi infrastruktur dengan pihak di luar kendali negara memperbesar risiko kebocoran data. | Kapasitas yang dipakai bersama dapat dimanfaatkan bergantian oleh banyak instansi, sehingga tidak ada kapasitas yang menganggur dan penggunaannya lebih efisien. | Pooling hanya di Private Cloud milik negara, dipakai bersama antarinstansi pemerintah, dengan isolasi per-instansi (kuota dan kunci terpisah). Data biometrik ditaruh di kelompok sumber daya yang terpisah. |
    | Rapid elasticity (kapasitas bisa naik-turun cepat sesuai kebutuhan) | Kapasitas tambahan tidak boleh mengorbankan kerahasiaan, sehingga data rahasia tidak boleh dipindahkan ke luar Private Cloud. | Akses dapat melonjak berkali lipat saat pemilu dan pendaftaran sekolah, sehingga kapasitas harus bisa ditambah dengan cepat. | Beban normal ditangani Private Cloud. Saat melonjak, hanya bagian yang tidak menyimpan data sensitif (portal, cache, antrean) yang melebar ke Public. Database inti dilindungi dengan cache, antrean, dan pembatasan laju. |
    | Measured service (pemakaian diukur dan dicatat) | Pencatatan pemakaian menunjukkan siapa mengakses NIK siapa, kapan, dan untuk tujuan apa, sehingga berfungsi sebagai alat kontrol kerahasiaan. | Data pemakaian membantu memprediksi kebutuhan kapasitas, terutama menjelang lonjakan, serta menghitung biaya pemakaian tiap instansi.| Setiap panggilan API dicatat, dan tiap instansi diberi batas jumlah permintaan (kuota). Pola yang tidak wajar, misalnya mengambil NIK secara berurutan dalam jumlah besar, akan terdeteksi lalu dibatasi. |

2. **Rekomendasikan Service Model** (IaaS / PaaS / SaaS) yang tepat untuk komponen-komponen berbeda dari sistem ini (boleh berbeda service model untuk komponen berbeda, mis. database inti vs portal publik untuk pengecekan status).
3. Jelaskan **minimal 2 risiko spesifik** dari pilihan kalian dan mitigasinya (mis. jika Hybrid Cloud dipilih: risiko sinkronisasi data antara private & public segment).
4. Bandingkan singkat dengan **satu alternatif yang kalian tolak** — kenapa alternatif itu kurang tepat (mis. kenapa Public Cloud murni ditolak, atau kenapa Private Cloud murni dianggap kurang skalabel).

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
