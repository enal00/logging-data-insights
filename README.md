# Forestry Operations Excellence Analytics

> Dari potensi tegakan hasil survei hingga profitabilitas: sistem Business Intelligence untuk memantau recovery volume, efisiensi biaya, kualitas log, kepatuhan, dan keberlanjutan operasi kehutanan.

## Ringkasan proyek

Proyek ini adalah studi kasus portofolio yang dikembangkan dari pengalaman saya sebagai **Kepala Seksi GIS & Pemetaan** serta **Kepala Bagian Perencanaan** pada operasi kehutanan. Tujuannya adalah mengubah data lapangan yang sebelumnya tersebar menjadi dashboard Power BI yang mendukung keputusan operasional.

Dashboard menghubungkan seluruh siklus hidup satu koridor kerja, dari survei potensi sampai penerimaan revenue. Dengan demikian, manajemen dapat melihat bukan hanya berapa volume yang dihasilkan, tetapi juga di tahap mana volume berkurang, biaya meningkat, kualitas menurun, atau kewajiban operasional berisiko terlambat.

> **Catatan kerahasiaan:** seluruh data dalam proyek ini bersifat sintetis dan telah dianonimkan. Struktur proses bisnis, definisi metrik, dan konteks pengambilan keputusan mencerminkan pengalaman operasional nyata, tanpa mengungkap data perusahaan sebelumnya.

## Daftar isi

- [Masalah bisnis](#masalah-bisnis)
- [Tujuan analisis](#tujuan-analisis)
- [Konteks proses bisnis](#konteks-proses-bisnis)
- [Ruang lingkup data](#ruang-lingkup-data)
- [Data model](#data-model)
- [KPI utama](#kpi-utama)
- [Dashboard dan pertanyaan analitis](#dashboard-dan-pertanyaan-analitis)
- [Insight dan rekomendasi strategis](#insight-dan-rekomendasi-strategis)
- [Kerangka operational excellence](#kerangka-operational-excellence)
- [Tools dan kompetensi yang ditunjukkan](#tools-dan-kompetensi-yang-ditunjukkan)
- [Batasan studi kasus](#batasan-studi-kasus)
- [Peran saya](#peran-saya)

## Masalah bisnis

Operasi berjalan melalui survei, penebangan, skidding, hauling, shipping, penanaman, dan aktivitas compliance. Sebelum dashboard ini, evaluasi antar tahap sulit dilakukan secara terpadu. Akibatnya, beberapa pertanyaan penting tidak dapat dijawab cepat dan konsisten:

1. Berapa potensi tegakan yang berhasil menjadi volume siap jual?
2. Di tahap mana kehilangan volume dan nilai terbesar terjadi?
3. Apakah kenaikan revenue benar-benar menghasilkan profit yang lebih besar setelah seluruh biaya diperhitungkan?
4. Leader survei dan formasi frontman-operator mana yang paling sesuai untuk jenis pekerjaan dan kondisi operasi tertentu?
5. Bagaimana meningkatkan performa tanpa mengorbankan keselamatan, kualitas log, kelestarian, atau kepatuhan?
6. Apakah pelaporan eksternal, training, dan sertifikasi berjalan sesuai tenggat?

## Tujuan analisis

1. Mengkuantifikasi perjalanan potensi dari standing stock hingga revenue pada level Job ID dan pohon, sehingga manajemen dapat melihat berapa volume yang benar-benar menjadi penjualan.
2. Membentuk funnel conversion antar-tahap—stock ke TPn, TPn ke hauling, dan hauling ke shipping—untuk menemukan lokasi, penyebab, serta nilai kehilangan volume yang paling material.
3. Menganalisis hubungan revenue, total biaya, cost per m³, dan profit margin agar pertumbuhan penjualan tidak dinilai terpisah dari efisiensi biaya.
4. Menentukan kesesuaian leader dengan jenis survei berdasarkan score dan cost per km, serta mengidentifikasi formasi frontman-operator berkontribusi volume tinggi sebagai dasar alokasi, rotasi, dan pembinaan kerja.
5. Mengarahkan perbaikan operational excellence melalui pemantauan recovery, umur kayu, trimming, kualitas log, keselamatan, kelestarian, dan dampak lonjakan biaya hauling.
6. Menjaga visibilitas penanaman dan aktivitas compliance—certification, external reporting, training, serta external relation—agar kewajiban operasional terkendali dan selesai tepat waktu.

## Konteks proses bisnis

```text
Survey potensi
  └─ Standing stock, target/realisasi jalur, leader, biaya survei
          ↓
Harvest / TPn
  └─ Volume yang layak dipanen setelah seleksi kelestarian dan kelayakan lapangan
          ↓
Skidding dan Hauling
  └─ Pengangkutan log, biaya produksi, biaya hauling, dan recovery volume
          ↓
Shipping dan QC buyer
  └─ Trimming, umur kayu, volume akhir, harga per jenis, dan revenue
          ↓
Profit
  └─ Revenue dikurangi survey, harvest, skidding, hauling, shipping,
     planting, dan compliance cost
```

Penanaman, sertifikasi, training, external relation, dan external reporting dimodelkan sebagai kewajiban pendukung. Aktivitas ini tidak langsung menghasilkan revenue, tetapi tetap memengaruhi total biaya, legalitas, kelestarian, dan kesinambungan operasi. Dalam model, `compliance_fee_cost` telah mencakup biaya certification, external reporting, training, dan external relation.

## Ruang lingkup data

- **Periode studi kasus:** 2024–2027, berdasarkan tanggal shipping ketika invoice dan revenue diakui.
- **Batas siklus operasi:** *ET+1*; tanggal shipping dapat berada maksimal satu tahun setelah kegiatan dimulai dari tahap survei.
- **Unit operasi:** satu blok per tahun.
- **Sumber operasional:** tally sheet lapangan untuk survei, harvest, skidding, hauling, dan shipping; GPS sebagai pendukung posisi spasial setiap tally sheet.
- **Proses data:** input Excel, data cleaning dan transformasi dengan Power Query, data modeling serta perhitungan KPI dengan DAX di Power BI.
- **Mata uang:** Rupiah (IDR).

Survei dapat dimulai sebelum periode pengakuan revenue. Ini mencerminkan kondisi operasi yang sebenarnya: pekerjaan lapangan, penebangan, pengangkutan, dan shipping memiliki jeda waktu sebelum invoice dapat diterbitkan.

## Data model

Entitas inti proyek adalah **Job ID**, yaitu satu koridor kerja hasil survei. Model menggunakan pendekatan *accumulating snapshot* pada level Job ID untuk mengikuti aktivitas survei, harvest, hauling, planting, dan compliance sepanjang siklus operasi.

Revenue dimodelkan terpisah pada level **ID Tree**, karena satu Job ID dapat menghasilkan beberapa catatan pohon dan transaksi penjualan. Relasi one-to-many ini memungkinkan analisis volume, kualitas, trimming, dan revenue sampai ke tingkat pohon tanpa kehilangan konteks pekerjaan asalnya.

| Tabel | Grain / peran utama |
|---|---|
| `survey_km` | Satu catatan per Job ID untuk target dan realisasi jalur survei, leader, metode, dan luas cakupan. |
| `survey_cruising` | Potensi tegakan dan biaya survei per Job ID. |
| `harvest` | Aktivitas harvest, skidding, hauling, shipping, volume tiap tahap, dan biaya produksi per Job ID. |
| `tree_detail` | Satu catatan per pohon untuk volume, jenis, harga, revenue, dan komponen biaya terkait. |
| `trimming` | Hasil trimming dan buyer/vendor pada level pohon. |
| `planting` | Biaya, jumlah bibit, dan tanggal penanaman per Job ID. |
| `certification`, `external_reporting`, `inhouse_training`, `external_relation` | Monitoring compliance, audit, pelaporan, dan aktivitas pendukung; biaya aktivitas tersebut terkonsolidasi dalam `compliance_fee_cost`. |
| `date` | Dimensi kalender untuk analisis waktu. |
| `location` | Dimensi lokasi spasial titik tengah blok/tahun. |

<!-- Tambahkan screenshot data model Power BI yang telah diperbarui di sini. -->

## KPI utama

| KPI | Definisi bisnis |
|---|---|
| Cost per KM | Biaya survei yang diperlukan untuk menghasilkan satu kilometer jalur survei. |
| Leader score | Realisasi panjang jalur dibandingkan target kerja standar 2 km per hari. |
| Conversion rate | Rasio volume yang berhasil berpindah antar-tahap: standing stock → TPn, TPn → hauling, dan hauling → shipping. Conversion stock → TPn sengaja tidak diharapkan mencapai 100% karena sebagian tegakan tetap dipertahankan untuk kelestarian, serta ada seleksi diameter, jenis langka/indah/dilindungi, dan kelayakan medan. |
| Volume recovery | Perbandingan volume yang bertahan pada setiap tahap operasi; digunakan bersama conversion rate untuk membedakan seleksi yang direncanakan dari loss yang perlu dikendalikan. |
| Cost per m³ | Total biaya seluruh tahap dibagi volume produksi (bukan volume siap jual). |
| Log age | Selisih hari antara tanggal harvest dan shipping. |
| Trimming pass rate | Volume yang lolos QC buyer setelah trimming dibanding volume sebelum trimming. |
| Revenue | Nilai penjualan setelah QC internal dan trimming final oleh buyer. |
| Profit margin | Profit dibagi total revenue. |
| On-time reporting rate | Proporsi laporan eksternal yang diterbitkan pada atau sebelum tenggat. |

### Contoh logika DAX

Berikut contoh pola measure yang digunakan.

```DAX
Cost per KM = SUM(survey_cruising[survey_cost])/SUM(survey_km[length])

score_total = SUM(survey_km[length])/SUM(survey_km[length_target])

Survey-to-Harvest Conversion % =
DIVIDE ( [Total Volume TPn], [Total Standing Stock] )

Harvest-to-Hauling Conversion % =
DIVIDE ( [volume_tpn_hauling], [Total Volume TPn] )

cost = 
SUM('harvest'[prod_cost])+[compliance_fee_cost]+[Sum of survey_planting_cost]

cost/m3 = 
DIVIDE([cost], SUM('tree_detail'[volume_tree]))

log_ageD = 
VAR a = AVERAGE(harvest[log_age])
RETURN
FORMAT(a, "0.00") & " Days"

Trimming Pass Rate % =
DIVIDE ( [volume_tpn_ship], [volume_tpn_hauling] )

Total Revenue =
SUM ( [volume_tpn_ship] * [price]  * [Trimming Pass Rate % ])

profit = 
([Total Revenue] - [cost])
```


## Dashboard dan pertanyaan analitis

Dashboard dibangun dalam beberapa halaman agar setiap keputusan dapat ditelusuri kembali ke proses operasionalnya.

| Halaman | Pertanyaan yang dijawab |
|---|---|
| Survey overview | Berapa potensi, luas, panjang jalur, biaya, dan distribusi metode survei? |
| Survey performance | Leader mana yang paling sesuai untuk setiap tahap survei, berdasarkan score tinggi dan cost per km yang efisien? |
| Harvest & production | Berapa conversion dari stock ke TPn dan siapa kontributor volume terbesar? |
| Frontman-operator assignment | Kombinasi mandor dan operator mana yang menghasilkan volume tertinggi? |
| Volume attrition | Di tahap mana volume berkurang dari standing stock sampai shipping? |
| Cost & profit | Bagaimana perubahan revenue, biaya, dan profit margin dari waktu ke waktu? |
| Planting & compliance | Apakah penanaman, audit, training, dan external reporting berjalan sesuai target? |

<!-- Tambahkan screenshot halaman dashboard yang telah diperbarui.
Contoh nama file: assets/dashboard-overview.png, assets/survey-performance.png,
assets/volume-attrition.png, assets/cost-profit.png, assets/compliance.png -->

## Insight dan rekomendasi strategis

### 1. Conversion rate harus dibaca sesuai fungsi metode survei

PAK (*Penataan Areal Kerja*) dan Orientasi memiliki conversion rate dari standing stock ke TPn yang lebih rendah dibanding ITSP (*Inventarisasi Tegakan Siap Tebang*) dan PWH_BI (*Pembukaan Wilayah Hutan*). Hal ini bukan otomatis menunjukkan performa yang buruk. PAK dan Orientasi berfungsi sebagai survei perintis dengan cakupan luas, termasuk medan sulit dan area dengan akses terbatas.

**Aksi:** gunakan PAK dan Orientasi untuk membangun cakupan awal serta menemukan koridor potensial. Arahkan ITSP dan PWH_BI ke area yang telah teridentifikasi memiliki potensi dan kelayakan operasi lebih tinggi. Target survei harus dibedakan menurut metode, kondisi medan, dan pengalaman leader.

### 2. Kapasitas kerja leader memiliki batas praktis

Hubungan antara target kerja dan leader score menunjukkan bahwa penambahan beban tidak selalu meningkatkan pencapaian. Pada tingkat beban tertentu, score dapat menurun karena perhatian, mobilitas, dan koordinasi lapangan tersebar ke terlalu banyak penugasan.

**Aksi:** gunakan scorecard leader yang menggabungkan target attainment, cost/km, luas cakupan per km, standing volume per km, dan conversion rate berikutnya. Alokasikan pekerjaan lebih besar kepada leader dengan performa stabil; gunakan pembinaan intensif dan penyeimbangan beban bagi leader dengan tren menurun. dan dilakukan spesialisasi ketua regu dengan jenis pekerjaan nya 

### 3. Loss setelah TPn adalah peluang utama untuk menjaga recovery dan revenue

Conversion standing stock ke TPn sebesar **82,39%** mencerminkan seleksi yang memang direncanakan: sebagian tegakan dipertahankan untuk kelestarian, sementara pohon yang tidak memenuhi kriteria diameter, spesies, perlindungan, atau kelayakan medan tidak dilanjutkan ke harvest. Karena itu, funnel improvement tidak berfokus untuk memaksimalkan rasio ini sampai 100%.

Peluang perbaikan terbesar berada setelah TPn, ketika volume sudah memasuki rantai produksi. Recovery TPn ke hauling sebesar **96,87%** menunjukkan loss **3,13%** yang dapat berkaitan dengan kerusakan saat penarikan traktor di medan berat, log pecah, keterlambatan pengupasan yang meningkatkan risiko kerusakan, reject spesifikasi, loading dan unloading, serta pemanfaatan log untuk kebutuhan sarana-prasarana. Recovery hauling ke shipping sebesar **95,88%** menunjukkan pengaruh trimming, umur kayu, kontak dengan air, dan QC final pabrik atau buyer.

Evaluasi proses juga menunjukkan potensi ketidaksejajaran antara insentif produktivitas lapangan yang berfokus pada jumlah pohon dan kualitas log yang baru terlihat pada tahap hilir. Kondisi ini berisiko memperbesar trimming, reject, serta penurunan revenue yang terealisasi.

**Aksi:** terapkan pengawasan dan QC berlapis di setiap titik kritis: pemeriksaan awal di TPn, inspeksi selama skidding dan hauling, sorting berkelanjutan di TPK, serta verifikasi akhir sebelum shipping. Standarkan reason code untuk setiap kehilangan volume, lalu tampilkan loss berdasarkan volume dan nilai rupiah. Evaluasi tim harvest juga perlu memasukkan recovery volume, reject rate, trimming pass rate, dan kepatuhan QC—bukan hanya jumlah pohon yang diproses.

### 4. Heatmap volume mendukung penyusunan formasi kerja

Performa frontman dinilai lebih dahulu berdasarkan total volume, kemudian dipetakan bersama kontribusi operator. Analisis matriks ini membantu mengidentifikasi formasi mandor-operator dengan output volume tinggi tanpa menyederhanakan evaluasi menjadi penilaian individu semata.

**Aksi:** gunakan heatmap sebagai dasar rotasi formasi, program mentoring, dan pembinaan intensif. Sebelum menstandarkan suatu kombinasi, validasi kembali hasilnya terhadap jenis pekerjaan, medan, dan periode yang sama.

### 5. Lonjakan biaya hauling dapat menipiskan profit meskipun revenue naik

Pada awal 2026, kenaikan harga BBM terlihat bersamaan dengan lonjakan biaya produksi. Karena profit dihitung dari revenue dikurangi seluruh biaya, pertumbuhan revenue tidak selalu menghasilkan margin yang lebih besar ketika biaya meningkat lebih cepat.

**Aksi:** monitor tren hauling cost bersama revenue dan profit margin. Saat biaya meningkat, evaluasi jarak angkut, kebutuhan armada, urutan pengiriman, dan prioritas volume bernilai lebih tinggi. Karena data konsumsi BBM bersifat terbatas, evaluasi menggunakan total cost hauling, bukan liter per m³-km.

### 6. Compliance dan reporting adalah pengendali kelangsungan operasi

Keterlambatan external reporting dapat menciptakan risiko bagi kelancaran operasional dan hubungan dengan stakeholder. Oleh karena itu, monitoring kepatuhan bukan sekadar aktivitas administrasi, melainkan bagian dari pengendalian risiko operasi.

**Aksi:** gunakan status jatuh tempo, indikator laporan terlambat, dan daftar pemilik tindakan pada dashboard. Prioritaskan pengingat sebelum tenggat dan eskalasi untuk laporan yang mendekati atau melewati batas waktu.

## Kerangka operational excellence

| Horizon | Fokus tindakan | Hasil yang diharapkan |
|---|---|---|
| 0–30 hari | Standarkan reason code kehilangan volume, status pelaporan, dan QC awal TPn. | Data penyebab masalah dapat dibandingkan antar tahap. |
| 1–3 bulan | Terapkan scorecard leader dan formasi kerja berbasis konteks metode/medan. | Alokasi beban kerja dan pembinaan menjadi lebih terarah. |
| 3–6 bulan | Gunakan review funnel volume, trimming, biaya hauling, dan profit margin dalam rapat operasi rutin. | Keputusan bergerak dari reaktif menjadi berbasis indikator awal. |
| Berkelanjutan | Lakukan evaluasi pascapanen terhadap realisasi jalur, recovery, dan kualitas. | Pembelajaran operasi kembali ke perencanaan blok berikutnya. |

Prinsip ini sejalan dengan praktik *Reduced Impact Logging*: perencanaan sebelum tebangan, jalur ekstraksi yang terencana, pengendalian pelaksanaan, dan evaluasi pascapanen untuk mengurangi kehilangan kayu sekaligus menjaga kondisi hutan. Lihat [FAO](https://www.fao.org/4/x0595e/x0595e05.htm) dan [ITTO](https://www.itto.int/sustainable_forest_management/logging/) untuk konteks praktik tersebut.

## Tools dan kompetensi yang ditunjukkan

| Area | Implementasi dalam proyek |
|---|---|
| Excel | Penyimpanan dan konsolidasi input tally sheet lapangan. |
| Power Query | Data cleaning, standardisasi, transformasi, dan persiapan data. |
| Power BI | Dashboard interaktif, visualisasi KPI, funnel, trend, heatmap, dan monitoring. |
| DAX | Perhitungan KPI, time intelligence, conversion, cost, revenue, recovery, dan profitabilitas. |
| Data modeling | Model Job ID ke ID Tree, dimensi kalender, relasi aktivitas operasi, serta pemisahan grain Job dan pohon. |
| Business analysis | Penerjemahan proses kehutanan menjadi KPI, pertanyaan bisnis, insight, dan rekomendasi tindakan. |

## Struktur repositori yang disarankan

```text
.
├── README.md
├── NAS_DATA.pbix                 # Dashboard Power BI (data sintetis)
├── assets/
│   ├── dashboard-overview.png
│   ├── survey-performance.png
│   ├── volume-attrition.png
│   ├── cost-profit.png
│   ├── compliance.png
│   └── data-model.png
└── data/
    └── anonymized/               # CSV/Excel sintetis yang aman dipublikasikan
```


## Batasan studi kasus

- Data publik adalah data sintetis dan dianonimkan; angka tidak boleh ditafsirkan sebagai kinerja perusahaan asli.
- Dashboard digunakan untuk mendukung evaluasi dan pembinaan operasional, bukan sebagai satu-satunya dasar keputusan personal.
- Data konsumsi BBM per unit tidak tersedia untuk publik; analisis dampak BBM menggunakan total cost hauling.
- Hasil formasi mandor-operator berbasis volume perlu dibaca bersama kondisi medan, jenis pekerjaan, dan periode operasi.

## Peran saya

Saya memimpin penerjemahan kebutuhan operasi kehutanan menjadi kerangka analitik: mendefinisikan KPI, menata proses data, membangun model Power BI, menyusun dashboard, dan menghubungkan insight dengan rekomendasi bagi pengambilan keputusan strategis.

---

Jika Anda ingin mendiskusikan proyek ini atau peluang Data Analyst / Business Intelligence, silakan hubungi saya melalui profil GitHub atau LinkedIn.
