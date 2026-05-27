# DST Hidden Markov Model v7 — XAUUSD (Adaptive SL/TP)

**Indikator Trading untuk TradingView (Pine Script v6)**  
Menggunakan **Hidden Markov Model (HMM)** untuk mendeteksi regime pasar (Sideways, Bull, Bear, Volatile Bull, Volatile Bear) secara real-time, dilengkapi dengan **Adaptive Stop Loss (SL) & Take Profit (TP)** berbasis Confidence Entropy (`VCH`).

---

## 📌 Fitur Utama

- ✅ **N-state HMM** (2–5 state, default 3: Side/Bull/Bear)
- ✅ **Observasi 4 dimensi** (return, ATR, slope, volume) — semuanya di-Z-score
- ✅ **Baum-Welch EM** untuk training ulang periodik & adaptif
- ✅ **Log-Viterbi** + **Duration Fix** (minimum durasi per state)
- ✅ **Confidence Entropy (VCH)** → mengukur keyakinan model (STRONG/MODERATE/WEAK)
- ✅ **Adaptive SL/TP** berbasis VCH
- ✅ **Tabel informatif**: Adaptive SL/TP, Transition Matrix, Emission, Stationary Distribution
- ✅ **Plot garis SL/TP** di chart utama
- ✅ **Adaptive Retraining Trigger** (vol shock, regime flip, confidence collapse)

---

## 🧠 Cara Kerja Singkat

### 1. Feature Engineering (D=4)
Setiap bar, dihitung 4 fitur yang dinormalisasi (Z-score):
- `z_ret`  → log return (close)
- `z_atr`  → ATR normalized
- `z_slope`→ kemiringan EMA (dinormalisasi dengan ATR)
- `z_vol`  → log volume

### 2. Hidden Markov Model
- **State** : Sideways (0), Bull (1), Bear (2), VBULL (3), VBEAR (4)
- **Emission** : Multivariate Gaussian (4 dimensi)
- **Transition** : Matrix ukuran N×N
- **Training** : Baum-Welch setiap interval tertentu (atau adaptif)

### 3. Regime Inference
- **Instantaneous Regime** (IR) → berdasarkan observasi terbaru
- **Viterbi** (dengan duration fix) → menghasilkan regime akhir `VR`
- **Posterior Forward-Backward** → probabilitas tiap state `VP`
- **Confidence Entropy (VCH)** :
