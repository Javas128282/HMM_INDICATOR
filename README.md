### Konsep Dasar Model
Model ini mendefinisikan **3 hidden states**:
- **State 0 = SIDEWAYS** (konsolidasi, tidak ada tren kuat)
- **State 1 = BULL** (tren naik)
- **State 2 = BEAR** (tren turun)

Setiap state memiliki karakteristik perilaku harga yang berbeda, yang dimodelkan melalui distribusi probabilitas.

### Observation (Input Model)
Yang dijadikan input observasi bukan harga mentah, melainkan **Dynamic Z-Score** dari log-return.  
Rumusnya:  
`log(close / close[1])` → dihitung rata-rata dan standar deviasi dalam window (default 20 bar) → kemudian dinormalisasi menjadi Z-Score.  

Keunggulan menggunakan Z-Score:
- Stasioner (tidak terpengaruh level harga)
- Unitless (bisa dibandingkan antar periode)
- Lebih mendekati distribusi normal (asumsi Gaussian emission)

### Cara Kerja Utama (Setiap Bar Baru)

Model bekerja dalam beberapa tahap penting yang dijalankan pada `barstate.islast`:

#### 1. Baum-Welch EM Algorithm (Training)
Ini adalah bagian inti dan paling berat. Model melakukan **learning** parameter menggunakan algoritma **Baum-Welch** (Expectation-Maximization):

- **E-Step**: Menghitung probabilitas state tersembunyi menggunakan Forward-Backward algorithm (dengan scaling untuk menghindari underflow).
- **M-Step**: Mengupdate tiga parameter utama:
  - **π** (initial state probabilities)
  - **A** (Transition Matrix) — probabilitas berpindah antar state
  - **μ dan σ** (Emission parameters) — mean dan standar deviasi Gaussian untuk setiap state

Proses ini diulang sebanyak `bw_n` iterasi (default 5) dengan **warm-start**, artinya parameter dari bar sebelumnya digunakan sebagai titik awal, sehingga tidak perlu belajar dari nol setiap kali.

#### 2. State Reordering (Disambiguation)
Setelah training, model memeriksa nilai μ (mean) dari ketiga state. Model otomatis mengatur ulang label agar:
- State 1 selalu memiliki μ tertinggi → **Bull**
- State 2 selalu memiliki μ terendah → **Bear**
- State 0 menjadi Sideways

Ini penting agar label state tidak "terbalik" antar bar.

#### 3. Viterbi Decoding
Setelah parameter ter-update, model menggunakan **Viterbi Algorithm** untuk mencari urutan state tersembunyi yang paling mungkin (Maximum A Posteriori). Hasilnya adalah **VR** (Viterbi Regime) — regime yang paling optimal untuk seluruh window observasi.

#### 4. Posterior Probability (γ)
Model juga menghitung probabilitas posterior untuk setiap state pada bar terakhir menggunakan Forward-Backward lagi. Nilai ini digunakan sebagai **Confidence Level** (VC).

### Komponen Pendukung Lainnya

- **Instantaneous Regime (IR)**: Perhitungan cepat hanya berdasarkan observasi saat ini (tanpa sequence), digunakan untuk regime ribbon dan warning.
- **Debounce (min_hold)**: Mencegah terlalu sering berganti regime dengan mewajibkan state sama selama beberapa bar.
- **Stationary Distribution**: Dihitung dengan memangkatkan Transition Matrix berkali-kali untuk melihat probabilitas jangka panjang.

### Visualisasi & Output
Script menampilkan banyak informasi:
- Regime Ribbon + Sideways Warning
- Banner utama dengan confidence
- Transition Matrix
- Emission Parameters (μ & σ per state)
- Stationary Distribution
- Kalkulasi SL/TP berdasarkan regime

---

**Kesimpulan Cara Kerja:**

Setiap bar close baru, script mengambil 60 bar terakhir, melatih ulang HMM menggunakan Baum-Welch, menyesuaikan label state, menjalankan Viterbi untuk mendapatkan regime terbaik, dan menghitung tingkat kepercayaan. Semua parameter (transition probability, distribusi emission) terus di-update secara adaptif sesuai kondisi pasar terkini.

Model ini jauh lebih sophisticated dibandingkan indikator regime sederhana yang hanya menggunakan moving average atau threshold harga. Ia benar-benar "belajar" pola perilaku Z-Score di setiap regime.
