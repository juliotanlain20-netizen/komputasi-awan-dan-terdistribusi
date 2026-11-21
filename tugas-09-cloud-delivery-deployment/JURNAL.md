# Jurnal Proses — Tugas 9

## Diskusi deployment model
- Kelompok mempertimbangkan empat pilihan Deployment Model, yaitu Public Cloud, Private Cloud, Hybrid Cloud, dan Community Cloud.

- Public Cloud dipertimbangkan karena memiliki skalabilitas tinggi, tetapi kurang sesuai untuk menyimpan data kependudukan yang sangat sensitif karena kendali infrastruktur tidak sepenuhnya berada pada pemerintah.

- Private Cloud memberikan kendali dan keamanan data yang lebih besar, tetapi membutuhkan perencanaan kapasitas yang baik untuk menghadapi lonjakan akses dari ribuan instansi.

- Community Cloud memungkinkan penggunaan infrastruktur bersama oleh beberapa organisasi dengan kepentingan yang sama, tetapi kurang fleksibel untuk kebutuhan sistem kependudukan nasional.

- Akhirnya, kelompok memilih Hybrid Cloud karena menggabungkan keamanan Private Cloud dan fleksibilitas Public Cloud. Data inti kependudukan tetap disimpan dan diproses di Private Cloud, sedangkan Public Cloud digunakan secara terbatas untuk layanan non-sensitif yang membutuhkan skalabilitas tinggi.
## Diskusi service model per komponen
- Kelompok menentukan Service Model berdasarkan kebutuhan kendali, keamanan, dan pengelolaan setiap komponen sistem.

- IaaS dipilih untuk database inti kependudukan dan mesin pencocokan biometrik karena kedua komponen tersebut menangani data sangat sensitif sehingga membutuhkan kendali infrastruktur dan konfigurasi keamanan yang lebih besar.

- PaaS dipilih untuk Gerbang API, WAF, dan API verifikasi NIK karena memudahkan pengelolaan aplikasi, integrasi antarsistem, serta penyesuaian kapasitas ketika terjadi lonjakan akses.

- PaaS juga dipilih untuk portal cek status warga di Public Cloud karena mendukung pengelolaan aplikasi dan auto-scaling. Portal hanya menangani tampilan dan konten non-sensitif, sedangkan pengecekan data individual tetap diproses melalui Private Cloud.

- SaaS dipilih untuk email dan ticketing pegawai karena merupakan layanan pendukung administrasi yang tidak memerlukan pengelolaan infrastruktur sendiri. Layanan tersebut tidak boleh digunakan untuk menyimpan atau mengirim data pribadi warga.

## Log Penggunaan AI (Level 2)

> Wajib diisi sesuai kebijakan Level 2 di [`../RUBRIK-UMUM.md`](../RUBRIK-UMUM.md). Tulis "Tidak memakai AI" pada baris pertama jika memang tidak dipakai. Hanya untuk brainstorming ide/outline — bukan untuk kode/analisis/teks akhir.

| Tanggal | Tool AI | Prompt yang diberikan | Ringkasan saran/ide AI | Bagaimana diolah jadi tulisan/kode sendiri |
|---|---|---|---|---|
| 21/11/2026 | chatgpt | jelaskan  mengenai konsep Deployment Model dan Service Model dalam Cloud Computing, khususnya perbedaan Public Cloud, Private Cloud, Hybrid Cloud, dan Community Cloud serta penerapan IaaS, PaaS, dan SaaS pada suatu sistem pemerintahan | AI menjelaskan karakteristik masing-masing model cloud, memberikan pertimbangan dalam memilih model yang sesuai berdasarkan keamanan dan skalabilitas, serta menjelaskan penerapan IaaS, PaaS, dan SaaS pada berbagai komponen sistem. Selain itu, dibahas pula risiko penggunaan Hybrid Cloud, langkah mitigasinya, dan perbandingan dengan Public Cloud. | perbandingan model, nomor 3 dan 4 pada tugas, dan jurnal |
| 21/11/2026 | Claude | Jelaskan mengenai konsep karakteristik esensial cloud computing NIST dan kaitannya dengan kerahasiaan vs skalabilitas untuk sistem data kependudukan | AI menjelaskan kelima karakteristik NIST (on-demand self-service, broad network access, resource pooling, rapid elasticity, measured service) | Digunakan untuk memahami konsep NIST, lalu hasilnya dipakai sebagai dasar menyusun alasan pemilihan deployment model berdasarkan karakteristik NIST. |