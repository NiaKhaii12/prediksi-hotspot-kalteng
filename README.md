# Prediksi Hotspot Kebakaran Hutan dan Lahan di Kalimantan Tengah
# Deskripsi
Proyek ini merupakan implementasi machine learning untuk memprediksi potensi kemunculan hotspot pada grid wilayah Kalimantan Tengah untuk satu hari berikutnya (H+1). Model dikembangkan dengan mengintegrasikan data hotspot satelit dan data meteorologi serta mempertimbangkan karakteristik spasial, temporal, dan ketidakseimbangan kelas.

# Tujuan
- Menganalisis pola persebaran hotspot secara spasial dan temporal.
- Mengidentifikasi karakteristik historis hotspot dan kondisi meteorologi.
- Memprediksi potensi kemunculan hotspot pada setiap grid berukuran 10 × 10 km.
- Membandingkan performa Random Forest, XGBoost, dan Soft Voting dengan prioritas pada pendeteksian hotspot.

# Dataset
Proyek menggunakan dua sumber data utama:
- Data hotspot: deteksi hotspot satelit NASA FIRMS.
- Data meteorologi: data meteorologi harian NASA POWER, meliputi temperatur udara (T2M), kelembapan relatif (RH2M), curah hujan (PRECTOTCORR), dan kecepatan angin (WS10M).

Data yang digunakan mencakup periode 2021 hingga 9 Oktober 2026.  
