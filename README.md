- VCH ≥ 0.6 → STRONG
- VCH < 0.35 → WEAK
- sisanya MODERATE

### 4. Adaptive SL/TP
- **Base SL multiplier** berdasarkan regime
- **Adjustment factor** berdasarkan VCH tier
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
3. Copy-paste seluruh kode `Hidden_Markov_Model.pine`
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
| Risiko saat VCH rendah | Tetap ambil posisi | **SL lebih lebar, TP lebih dekat** |

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
