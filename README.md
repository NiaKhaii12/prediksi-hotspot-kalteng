Deskripsi

Penelitian ini membangun model klasifikasi biner untuk memprediksi kemunculan hotspot pada hari berikutnya (H+1) di zona-zona rawan di Provinsi Kalimantan Tengah. Data hotspot satelit VIIRS dibersihkan terlebih dahulu: hanya deteksi dengan confidence nominal dan high yang dipakai, lalu titiknya dipotong sesuai batas administrasi provinsi. Setelah itu, data hotspot periode latih (2021–2024) dikelompokkan dengan HDBSCAN (metrik haversine) menjadi 60 zona spasial. Hotspot periode uji dimasukkan ke zona latih terdekat dalam radius 5 km, sehingga zona tidak dibentuk ulang dengan data uji.

Untuk setiap kombinasi zona dan hari, disusun 34 kolom data. Isinya adalah fitur meteorologi dari lima titik grid NASA POWER yang diinterpolasi ke pusat zona dengan inverse distance weighting, fitur turunan cuaca (akumulasi hujan, hari kering berturut-turut, vapour pressure deficit), riwayat hotspot zona, aktivitas zona tetangga dan provinsi, serta komponen musiman. Model yang dibandingkan adalah Random Forest, XGBoost, dan soft voting keduanya. Pembanding acuannya adalah model persistence, yang mengasumsikan kondisi besok sama dengan hari ini.

Data dibagi berdasarkan waktu. Data latih mencakup 2021–2024, dengan validasi silang per tahun (validasi 2023 dan 2024), dan data uji mencakup 2025–2026. Pembagian ini mencegah informasi masa depan bocor ke proses pelatihan. Pengujian dilakukan berlapis: ablasi kelompok fitur, hyperparameter tuning, perbandingan skenario penanganan ketidakseimbangan kelas (tanpa penanganan, class weight, undersampling), penentuan threshold dari prediksi out-of-fold, evaluasi per tahun, walk-forward retraining per kuartal, dan analisis galat.

Tujuan
Membentuk zona spasial rawan hotspot di Kalimantan Tengah dari sebaran hotspot historis menggunakan HDBSCAN.
Membangun model Random Forest, XGBoost, dan soft voting untuk memprediksi kemunculan hotspot H+1 per zona berdasarkan fitur meteorologi, riwayat hotspot, aktivitas spasial di sekitar zona, dan musim.
Mengukur kontribusi tiap kelompok fitur (ablasi), pengaruh hyperparameter tuning, dan pengaruh strategi penanganan ketidakseimbangan kelas terhadap kinerja model, dengan PR-AUC sebagai metrik utama.
Menilai apakah model memberi nilai tambah dibandingkan baseline persistence, serta seberapa stabil kinerjanya antartahun dan antarkuartal (model statis dibandingkan walk-forward).
Menganalisis pola kesalahan prediksi (alarm palsu dan kejadian terlewat) beserta fitur yang paling berpengaruh.
