# Tradesys Edge IchiFib

Web app sistem trading crypto **Ichimoku × Fibonacci** untuk **Binance Spot (long-only)**.
Ichimoku dan Fibonacci dipakai sebagai **satu strategi kombo yang tidak terpisah**, mengikuti *Panduan Trading Crypto: Ichimoku + Fibonacci* (versi crypto dari panduan forex Didi Trader Klinik), dengan dasar dari `ichimoku-framework-kb` dan `fibonacci-framework-kb`.

Satu file `index.html`: tanpa build, tanpa server, tanpa API key. Data dari API publik Binance di browser.

> Aplikasi ini hanya memberi **sinyal dan alert**. Tidak ada order yang dikirim ke exchange.

---

## Sistem kombo (BAB 3 panduan)

| # | Langkah | Aturan di aplikasi |
|---|---|---|
| 1 | **Filter pasar** | Altcoin: hanya long jika **BTC Daily close di atas awan**. BTC di dalam awan = tidak buka posisi altcoin baru. Volume 24 jam ≥ 50 juta USDT, spread ≤ 0,1% (bisa diubah). |
| 2 | **Tren** | Bias Daily, setup H4. Close H4 **di atas awan** dan Daily **tidak di bawah** awannya. Di dalam awan = no trade. |
| 3 | **Tarik Fibonacci** | Swing low → swing high terakhir yang **sudah terkonfirmasi** (pivot fraktal N = 5), wick ke wick, swing ≥ 3 × ATR(14). |
| 4 | **Konfluensi** | Fib **61,8%** dalam ≤ 0,5 × ATR dari **tepi awan** (Span A/B) atau **Kijun datar**. Zona entry = **61,8–78,6%**. Close di bawah 78,6% membatalkan swing. |
| 5 | **Konfirmasi** | Pada candle yang sudah close di zona: **Tenkan berbalik naik**, **close di atas Kijun**, **Chikou bebas** di atas harga & awan 26 bar lalu. Satu gagal = tunggu. |
| 6 | **Stop loss** | Di bawah tepi bawah awan − 0,5 × ATR. |
| 7 | **Target** | TP1 = swing high (0%), TP2 = 127,2%, TP3 opsional = 161,8% atau trailing Kijun. **Tolak jika R:R ke TP1 < 1:2 setelah fee.** |

**Tahap di papan kombo:**
- **Setup** — konfluensi 61,8% × awan/Kijun ada, menunggu pullback ke zona.
- **Di zona** — harga sudah di zona, menunggu konfirmasi Tenkan/Kijun/Chikou.
- **Sinyal** — semua aturan lolos di candle yang sudah close → trading plan lengkap.

Untuk tahap Setup/Di zona aplikasi menampilkan **rencana bersyarat** (entry, SL, TP, R:R) dan apa yang belum terpenuhi.

**Cek sinyal kuat (4 konfirmasi BAB 1):** close di atas awan, Tenkan di atas Kijun, Chikou bebas, awan depan hijau — ditampilkan x/4.

### Dua set aturan
| | Ketat (default) | Longgar |
|---|---|---|
| Sumber | Panduan apa adanya | Pelonggaran, **belum teruji** |
| R:R minimum diukur ke | TP1 | TP2 (127,2%) |
| Konfirmasi | Di candle yang close di zona | Boleh menyusul ≤ 6 bar setelah zona tersentuh |
| Close di atas Kijun | Wajib | Tidak wajib |

---

## Money management (BAB 5)

- **Qty = Modal × Risiko% ÷ (|Entry − SL| + (Entry + SL) × fee)**, fee = fee + slippage per sisi. Contoh panduan (5.000 USDT, risiko 1%, fee 0,10%, entry 93.800, SL 91.950) → **0,02456 BTC**; tanpa fee 0,02703 BTC.
- Risiko 1% per trade (maks 2%), total terbuka ≤ 3%, maks 3 posisi, altcoin searah BTC = satu eksposur.
- Partial **50% di TP1**, SL sisa ke **breakeven + fee**; sisa ke TP2 (atau 30% TP2 + 20% TP3, atau trailing Kijun).
- **Batas rugi harian −3%** → aplikasi mengunci sampai 07:00 WIB (sinyal Telegram dijeda).
- **Drawdown:** 10% → risiko 0,5% (otomatis), 20% → stop live, 30% → berhenti total.
- **Weekend:** risiko dipotong 50% (bisa diganti ke *tidak entry*).

## Timing (BAB 4)

Tab **Timing**: jam WIB sekarang, close H4 & Daily berikutnya, funding berikutnya, status weekend, jendela entry manual (07:00–23:00), jadwal rilis data AS/FOMC dari panduan, dan **profil volume & range per jam** yang dihitung dari data 1H Binance 3–12 bulan per koin.

---

## Fitur lain
Dashboard (KPI, chart, plan, checklist 10 butir dari lampiran panduan, probabilitas historis), Scanner (tahap, konfluensi, volume, filter BTC), Chart (Kumo diproyeksikan 26 bar, Tenkan/Kijun/Chikou, Fib dengan zona 61,8–78,6% dan titik tepi awan yang berhimpit), Backtest (preset ketat/longgar, Ichimoku 9/26/52/26 vs 20/60/120/30, filter BTC on/off, walk-forward 70/30, rugi beruntun terpanjang), Risiko, Alert Telegram, Pengaturan. Bahasa EN/ID, tema terang/gelap.

---

## Hasil uji (22 Nov 2026) — dibaca dulu

**Teknis (lulus):**
- **Look-ahead:** 5.742 titik di 6 pair H4 dihitung ulang dengan data masa depan dipotong (termasuk data Daily & BTC): **0 selisih**. Awan yang dibandingkan dengan harga di bar t dihitung dari bar t − 26; pivot baru dipakai setelah N bar; eksekusi di open candle berikutnya.
- **Invarian backtest:** 0 pelanggaran.
- **Browser:** live, demo, dan uji "mundur waktu" ke bar sinyal historis (BTC 4H, 4 Sep 2026) di desktop & HP, tanpa error.

**Strategi** — 10 pair (BTC, ETH, SOL, BNB, XRP, ADA, AVAX, LINK, DOGE, LTC), **H4 3 tahun** (Sep 2023 – Nov 2026) dan **D1 hingga 8 tahun**, fee 0,1% + slippage 0,05% per sisi:

| Varian | H4 trade | D1 trade |
|---|---|---|
| Ketat (panduan), 9/26/52/26 | **0** | **0** |
| Ketat, 20/60/120/30 | 0 | 0 |
| Ketat, tanpa filter BTC | 0 | 0 |
| Longgar | 4 (−0,33R rata-rata) | 2 (−0,14R) |

**Kenapa hampir tidak ada sinyal:** saat harga berada di 61,8–78,6%, harga hampir selalu **di bawah Kijun** (Kijun ≈ titik tengah 26 bar, dekat level 50%) dan **Chikou masih tertahan** harga 26 bar lalu (sekitar swing high), sehingga konfirmasi langkah 5 jarang lolos. Ketika lolos, **SL di bawah seluruh awan** membuat R:R ke TP1 rata-rata hanya ~0,5, jauh di bawah syarat 1:2.

**Kesimpulan:** aturan panduan saat ini belum bisa divalidasi (target panduan ≥ 100 trade). Aplikasi tetap berguna sebagai **watchlist bertahap** (Setup → Di zona) dengan checklist dan rencana bersyarat, tapi parameter perlu direvisi lalu diuji ulang sebelum dipakai live. Mode longgar juga belum punya sampel yang cukup.

---

## Cara upload (GitHub Pages)
1. Repo → **Settings → Pages** → *Deploy from a branch*, branch `main`, folder `/ (root)` → **Save**.
2. Buka `https://haedar-collab.github.io/ichimoku-fibonacci-framework/`.
3. Di HP: Chrome → **⋮ → Add to Home screen**.

Data aplikasi disimpan dengan prefix `tsif_` di localStorage.

## Setup Telegram
1. **@BotFather** → `/newbot` → salin token. Kirim pesan ke bot, buka `https://api.telegram.org/bot<TOKEN>/getUpdates`, salin chat ID.
2. **Pengaturan → Data dan Telegram**: isi token & chat ID, centang *Kirim sinyal live baru* (opsional: *Kirim juga setup yang masuk zona*).
3. Tab **Alert** → *Kirim pesan tes* / *Kirim sinyal tes*.

Token hanya disimpan di browser. Untuk API key exchange (jika nanti dipakai): tanpa izin withdraw, whitelist IP.

## Struktur kode
- `ENGINE-START … ENGINE-END`: indikator, `ichiCore()`, `pivots()`, `ichiAt()`, `fibAt()`, `comboCore()` (7 langkah kombo), `evalAt()`, `backtest()` (sizing dengan fee, partial 50%, BE + fee, walk-forward), `htfStates()`.
- Data: `getKlines()` (hanya candle close), `attach()` (Daily & filter BTC), `universe()` (volume 24 jam, stablecoin dikeluarkan).

*Bukan nasihat keuangan. Ikuti tahapan validasi di panduan (backtest ≥ 3 tahun, walk-forward, forward test 4–8 minggu, live dengan risiko 0,25–0,5% dulu).*
