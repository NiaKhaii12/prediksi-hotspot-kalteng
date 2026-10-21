# Prediksi Kemunculan Hotspot H+1 Berbasis Zona Spasial di Kalimantan Tengah

Model klasifikasi biner untuk memprediksi kemunculan *hotspot* (titik panas) pada hari berikutnya (H+1) di zona-zona rawan di Provinsi Kalimantan Tengah, menggunakan Random Forest, XGBoost, dan *soft voting*.

> **Catatan:** target model adalah *deteksi hotspot satelit*, bukan kejadian kebakaran yang terverifikasi di lapangan. Hotspot adalah indikator anomali suhu permukaan; tidak setiap hotspot merupakan kebakaran.

---

## Deskripsi

Data hotspot satelit VIIRS dibersihkan dengan hanya mempertahankan deteksi ber-*confidence* *nominal* dan *high*, lalu dipotong sesuai batas administrasi provinsi. Hotspot periode latih (2021–2024) kemudian dikelompokkan menjadi **60 zona spasial** dengan **HDBSCAN** (metrik *haversine*). Hotspot periode uji ditugaskan ke zona latih terdekat dalam radius 5 km, sehingga zona tidak dibentuk ulang menggunakan data uji.

Untuk setiap kombinasi zona–hari disusun 34 kolom data yang mencakup:

- **Fitur meteorologi** dari lima titik grid NASA POWER, diinterpolasi ke pusat zona dengan *inverse distance weighting* (IDW).
- **Fitur turunan cuaca**: akumulasi hujan 3/7/14/30/60 hari, hari kering berturut-turut, rata-rata bergerak suhu dan kelembapan, serta *vapour pressure deficit* (VPD).
- **Riwayat hotspot zona**: status hari ini, jumlah hari ber-hotspot dalam 3/7/14/30 hari, dan jumlah hari sejak hotspot terakhir.
- **Aktivitas spasial**: aktivitas zona tetangga (radius 30 km) dan aktivitas tingkat provinsi.
- **Komponen musiman**: sinus dan kosinus hari dalam setahun.

Model yang dibandingkan adalah Random Forest, XGBoost, dan *soft voting* keduanya. Sebagai pembanding acuan digunakan *baseline persistence*, yang mengasumsikan kondisi besok sama dengan hari ini.

Data dibagi berdasarkan waktu untuk mencegah kebocoran informasi dari masa depan. Alur pengujiannya sebagai berikut:

1. Ablasi kelompok fitur
2. *Hyperparameter tuning* (`RandomizedSearchCV`, metrik PR-AUC)
3. Perbandingan skenario penanganan ketidakseimbangan kelas (tanpa penanganan, *class weight*, *undersampling*)
4. Penentuan *threshold* dari prediksi *out-of-fold*
5. Evaluasi pada data uji dan per tahun
6. *Walk-forward retraining* per kuartal
7. Analisis galat dan *feature importance*

## Tujuan

1. Membentuk zona spasial rawan hotspot di Kalimantan Tengah dari sebaran hotspot historis menggunakan HDBSCAN.
2. Membangun model Random Forest, XGBoost, dan *soft voting* untuk memprediksi kemunculan hotspot H+1 per zona berdasarkan fitur meteorologi, riwayat hotspot, aktivitas spasial, dan musim.
3. Mengukur kontribusi tiap kelompok fitur, pengaruh *hyperparameter tuning*, dan pengaruh strategi penanganan ketidakseimbangan kelas terhadap kinerja model, dengan PR-AUC sebagai metrik utama.
4. Menilai nilai tambah model dibandingkan *baseline persistence*, serta stabilitas kinerjanya antartahun dan antarkuartal (model statis vs. *walk-forward*).
5. Menganalisis pola kesalahan prediksi (alarm palsu dan kejadian terlewat) beserta fitur yang paling berpengaruh.

## Dataset

| Dataset | Sumber | Variabel yang dipakai | Ukuran |
|---|---|---|---|
| `hotspot_kalimantan_tengah.csv` | [NASA FIRMS](https://firms.modaps.eosdis.nasa.gov/), VIIRS 375 m *(TODO: sebutkan satelit S-NPP / NOAA-20 / NOAA-21 dan tanggal unduh)* | `acq_date`, `latitude`, `longitude`, `confidence` | 178.732 baris × 15 kolom → 172.129 titik setelah filter *confidence* dan pemotongan batas provinsi |
| `meteorologi_kalimantan_tengah.csv` | [NASA POWER](https://power.larc.nasa.gov/), data harian *(TODO: tanggal unduh)* | `T2M` (suhu 2 m), `RH2M` (kelembapan relatif 2 m), `PRECTOTCORR` (curah hujan terkoreksi), `WS10M` (kecepatan angin 10 m) | 10.555 baris × 6 kolom; 5 titik × ±2.111 hari |
| `Kalimantan Tengah.geojson` | *(TODO: sumber data batas administrasi)* | Poligon batas provinsi | — |
| Panel zona–hari (hasil olahan) | Turunan dari ketiga data di atas | 34 kolom fitur dan target | 126.600 baris (60 zona × ±2.110 hari); kelas positif 6,75% |

**Titik meteorologi (koordinat grid NASA POWER, bukan stasiun BMKG):**

| Lokasi | Lintang | Bujur |
|---|---|---|
| Palangka Raya | -2,21 | 113,92 |
| Sampit | -2,53 | 112,95 |
| Pangkalan Bun | -2,68 | 111,62 |
| Muara Teweh | -0,95 | 114,89 |
| Kuala Kapuas | -3,00 | 114,39 |

Nilai hilang NASA POWER (kode `-999`, 15 nilai per variabel) diisi dengan interpolasi per titik.

**Pembagian data (berbasis waktu):**

| Bagian | Periode | Keterangan |
|---|---|---|
| Latih | 2021–2024 | 87.600 baris; hari terakhir 2024 dibuang karena targetnya jatuh di 2025 |
| Validasi silang | Fold 1: latih ≤2022 → validasi 2023; Fold 2: latih ≤2023 → validasi 2024 | Dipakai untuk ablasi, *tuning*, pemilihan skenario *imbalance*, dan *threshold* |
| Uji | 2025 – data terakhir 2026 | *(TODO: cocokkan tanggal awal dan akhir dengan file CSV)* |

## Keterbatasan

- Zona HDBSCAN hanya mencakup **67%** hotspot periode uji. Sisanya, 33%, berada di luar zona sehingga tidak ikut diprediksi.
- Data meteorologi berasal dari produk grid NASA POWER, bukan observasi stasiun. Resolusinya jauh lebih kasar daripada ukuran zona.
- Proporsi kelas positif berbeda tajam antartahun uji: 3,0% pada 2025 dan 21,0% pada 2026. Metrik uji gabungan karena itu banyak dipengaruhi 2026, padahal data tahun tersebut belum lengkap. Lihat evaluasi per tahun.

## Struktur Notebook (`Emel.ipynb`)

| Tahap | Isi |
|---|---|
| 1 | Setup, import library, baca data |
| 2 | Cleaning data meteorologi dan fitur turunan |
| 3 | Cleaning hotspot dan pemotongan batas provinsi |
| 4 | Fungsi pendukung dan parameter awal |
| 5 | Zonasi HDBSCAN, fitur cuaca tambahan, peta |
| 6 | Panel zona–hari, target H+1, split waktu |
| 7 | Ablasi kelompok fitur |
| 8 | *Hyperparameter tuning* RF dan XGBoost |
| 9 | Penanganan *imbalance* |
| 10 | Model final, *threshold*, evaluasi uji |
| 11 | *Walk-forward retraining* |
| 12 | Analisis galat dan interpretasi |

## Kebutuhan

```
pandas numpy geopandas shapely scikit-learn xgboost hdbscan scipy matplotlib joblib
```

Notebook dijalankan di Google Colab. Letakkan ketiga file data di `/content/`. Hasil antara di-*cache* ke Google Drive (`PAKAI_DRIVE = True`) atau ke `/content/cache`.
