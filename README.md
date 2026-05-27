---

```markdown
# DST Hidden Markov Model v7 — XAUUSD (Adaptive SL/TP)

**Indikator Trading untuk TradingView (Pine Script v6)**  
Menggunakan **Hidden Markov Model (HMM)** untuk mendeteksi regime pasar (Sideways, Bull, Bear, Volatile Bull, Volatile Bear) secara real-time, dilengkapi dengan **Adaptive Stop Loss (SL) & Take Profit (TP)** berbasis Confidence Entropy (`VCH`).

![Contoh Tampilan](https://via.placeholder.com/800x400?text=HMM+v7+Example)

---

## 📌 Fitur Utama

- ✅ **N-state HMM** (2–5 state, default 3: Side/Bull/Bear)
- ✅ **Observasi 4 dimensi** (return, ATR, slope, volume) — semuanya di-Z-score
- ✅ **Baum-Welch EM** untuk training ulang periodik & adaptif
- ✅ **Log-Viterbi** + **Duration Fix** (minimum durasi per state)
- ✅ **Confidence Entropy (VCH)** → mengukur keyakinan model (STRONG/MODERATE/WEAK)
- ✅ **Adaptive SL/TP** berbasis VCH:
  - STRONG → SL lebih ketat, TP lebih jauh
  - WEAK → SL lebih lebar, TP lebih dekat (proteksi modal)
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
  ```
  VCH = 1 - (H / log(N))
  H   = - Σ p(s) * log(p(s))
  ```
  - VCH ≥ 0.6 → STRONG
  - VCH < 0.35 → WEAK
  - sisanya MODERATE

### 4. Adaptive SL/TP (NEW in v7)
- **Base SL multiplier** berdasarkan regime (Bull/Bear/VBULL/VBEAR)
- **Adjustment factor** berdasarkan VCH tier
  ```
  SL_final   = base_sl × (tight/wide/1.0)
  RR_final   = RR_strong / RR_moderate / RR_weak
  TP_final   = SL_final × RR_final
  ```
- **Partial TP 1:1** (risk pertama)
- **Plot garis** di chart & tabel detail

### 5. Adaptive Retraining Trigger
Model dilatih ulang lebih cepat jika:
- Volatilitas shock (z_atr > threshold)
- Regime terlalu sering berubah (flip)
- Confidence collapse (VCH < 0.2)

---

## 📥 Cara Install di TradingView

1. Buka TradingView → **Pine Editor**
2. Buat **indicator baru**
3. Copy-paste seluruh kode `HMM_V7.txt`
4. Simpan dengan nama bebas (contoh: `HMM_v7_XAUUSD`)
5. Tambahkan ke chart (symbol XAUUSD atau lainnya)

> ⚠️ Indikator ini dirancang untuk **XAUUSD (gold)** tetapi bisa dicoba di pair lain dengan penyesuaian parameter.

---

## ⚙️ Parameter Utama

### HMM Core
| Parameter | Deskripsi |
|-----------|------------|
| `Jumlah Hidden States (N)` | 3 (default), 4, atau 5 |
| `Training Window (bars)` | 150 bar untuk training |
| `Baum-Welch Iterations` | 3 iterasi EM |
| `Min Duration Side/Bull` | 2 / 1 bar |

### Adaptive SL/TP
| Parameter | STRONG | MODERATE | WEAK |
|-----------|--------|----------|------|
| SL Adjustment | 0.75× | 1.0× | 1.4× |
| RR Ratio | 3.0 | 2.0 | 1.0 |
| VCH Threshold | ≥0.6 | 0.35–0.6 | <0.35 |

### Retraining
- Interval reguler: 20 bar
- Adaptive trigger: Vol shock (z_atr > 2.0), regime flip (>4x per 10 bar), confidence collapse (<0.20)

---

## 📊 Interpretasi Sinyal

| Regime | Arah Trading | SL | TP | Status |
|--------|--------------|----|----|--------|
| **SIDE (0)** | DON'T TRADE | - | - | Hindari |
| **BULL (1)** | LONG | di bawah close | di atas close | Ikuti naik |
| **BEAR (2)** | SHORT | di atas close | di bawah close | Ikuti turun |
| **VBULL (3)** | LONG (volatil) | lebih lebar | lebih jauh | Buy with care |
| **VBEAR (4)** | SHORT (volatil) | lebih lebar | lebih jauh | Sell with care |

### VCH Tier → Prioritas
- **STRONG** + Bull/Bear → HIGH PRIORITY
- **MODERATE** → TRADE WITH CARE
- **WEAK** → SKIP / WAIT (model ragu)

---

## 📈 Output Visual

### 1. Regime Ribbon
- Warna latar belakang berubah sesuai regime terbaru (IR)

### 2. Tabel Adaptive SL/TP
Menampilkan:
- Regime & VCH tier
- SL multiplier & adjustment
- Risk:Reward
- Harga SL/TP (Long & Short)
- Partial TP 1:1
- Status sinyal

### 3. Plot Garis di Chart
- **Garis merah** = SL
- **Garis hijau** = TP
- **Garis hijau muda** = Partial TP

### 4. Tabel Pendukung
- Transition Matrix (probabilitas pindah state)
- Stationary Distribution (A^n)
- Emission Parameters (μ dan σ tiap state)

---

## 🧪 Contoh Skenario Trading

**Skenario 1: Bull Strong**
- Regime = BULL, VCH = 0.75 (STRONG)
- Base SL Bull = 1.2× ATR
- SL final = 1.2 × 0.75 = 0.9× ATR (lebih ketat)
- RR = 3.0 → TP = 2.7× ATR
- Status = HIGH PRIORITY ✅

**Skenario 2: Bear Weak**
- Regime = BEAR, VCH = 0.20 (WEAK)
- Base SL Bear = 1.5× ATR
- SL final = 1.5 × 1.4 = 2.1× ATR (lebih lebar)
- RR = 1.0 → TP = 2.1× ATR
- Status = SKIP / WAIT ⚠️

---

## 🔄 Perbedaan v6 → v7

| Fitur | v6 | v7 |
|-------|----|----|
| SL/TP | Konstan (per regime) | **Adaptive** berdasarkan VCH |
| Tabel SL/TP | "DST KALKULASI" | **ADAPTIVE SL/TP** + adjustment factor |
| Plot garis | Tidak ada | **Ada** (SL/TP/Partial) |
| Risiko saat VCH rendah | Tetap ambil posisi | **SL lebih lebar, TP lebih dekat** (proteksi) |

---

## 📝 Catatan Penting

- Indikator ini bersifat **educational & experimental**
- **Tidak ada jaminan profit** — selalu gunakan manajemen risiko
- Performa terbaik pada **XAUUSD timeframe 1H–4H**
- Jangan gunakan di timeframe terlalu kecil (<15m) karena noise
- Pastikan data historis cukup (minimal 150 bar)

---

## 🛠️ Development & Debug

Jika ingin melihat proses training ulang:
- Aktifkan `show_ban = true`
- Perhatikan baris `Retrain #X (Y adp)  |  next: Zb`

Jika SL/TP tidak muncul:
- Pastikan regime bukan SIDE (0)
- Cek `show_sltp_plot = true`
- Periksa nilai `atr_sl` tidak nol

---

## 📄 Lisensi

MIT License — Bebas digunakan, dimodifikasi, dan didistribusikan.  
Dikembangkan untuk komunitas trading algorithmic.

---

## 🙏 Kredit

- DST (Developer)
- Berbasis Hidden Markov Model (Rabiner, 1989)
- Implementasi Pine Script v7

---
