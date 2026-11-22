# Tradesys Edge IchiFib

Web app sistem trading crypto **Ichimoku Cloud + Fibonacci** untuk **Binance Spot (long-only)**.
Satu file `index.html`: tanpa build, tanpa server, tanpa aplikasi berbayar. Data diambil langsung dari API publik Binance di browser.

Strategi disusun dari dua knowledge base:
- `ichimoku-framework-kb` (*Ichimoku Cloud Profit System*): urutan baca Kumo → ketebalan → Tenkan/Kijun → Chikou, serta setup TK Cross, Kumo Breakout, dan Trend Riding.
- `fibonacci-framework-kb` (*The Art of Fibonacci Trading*): retracement 38.2/50/61.8, konfirmasi price action, skor konfluensi ≥ 3/5, RR minimal 1:2, TP di extension.

> Aplikasi ini hanya memberi **sinyal dan alert**. Tidak ada order yang dikirim ke exchange dan tidak butuh API key Binance.

---

## Fitur

| Tab | Isi |
|---|---|
| **Dashboard** | BTC, Kumo BTC harian (risk-on/off), jumlah sinyal Ichimoku & Fibonacci, komposisi harga vs Kumo, chart dengan plan, **kartu Trading Plan**, checklist Ichimoku × Fibonacci (x/8), probabilitas historis, papan sinyal |
| **Scanner** | Watchlist atau semua pair USDT (top N volume). Kolom: Kumo, TK, Chikou, HTF, kedalaman Fib, sinyal, skor, entry/SL/TP. Filter: Sinyal / Ichimoku / Fibonacci / Di atas Kumo / Semua |
| **Chart** | Candlestick + **Kumo diproyeksikan 26 bar ke depan**, Tenkan, Kijun, Chikou, level Fibonacci kaki naik terakhir (zona 38.2–61.8 diarsir, extension 127.2/161.8), volume, struktur SMC, pola candle. Panel *Bacaan Ichimoku* dan *Bacaan Fibonacci* |
| **Backtest** | Kedua strategi, Ichimoku saja, atau Fibonacci saja; 1.000–3.000 bar; fee & slippage; hasil per setup |
| **Risiko** | Kalkulator ukuran posisi + aturan portofolio |
| **Alert** | Log sinyal, kirim sinyal tes / pesan tes ke Telegram, auto-scan 1 jam / 4 jam |
| **Pengaturan** | Parameter Ichimoku, Fibonacci, filter, risiko, endpoint Binance, Telegram |

Timeframe: **1H, 4H, 1D**. Bahasa **EN / ID**, tema **Terang / Gelap**. Harga live diperbarui tiap 15 detik.

---

## Logika strategi

### 1. Baca Kumo dulu (wajib, Bab 4 Ichimoku)
Setting **9-26-52**, pergeseran 26 (sama seperti TradingView).

| Cek | Bullish (boleh long) |
|---|---|
| Harga vs Kumo | Close di **atas** Kumo |
| Ketebalan Kumo | Dicatat dalam × ATR (tebal = support kuat, tipis = rawan fakeout) |
| Tenkan / Kijun | Tenkan di atas Kijun, Kijun naik |
| Chikou | Close **bebas** di atas harga dan Kumo 26 bar lalu |
| Kumo ke depan | Senkou A di atas Senkou B |
| Jarak ke Kijun | ≤ 2,5 ATR (lebih dari itu = terlalu jauh, jangan kejar) |

**Multi-timeframe** (1H→4H, 4H→1D, 1D→1W): setup Ichimoku wajib timeframe besar close **di atas** Kumo-nya; setup Fibonacci diblokir bila timeframe besar **di bawah** Kumo.
**BTC harian** di bawah/di dalam Kumo → sinyal diberi label risk-off (skor −10), atau diblokir jika diatur.

### 2. Setup Ichimoku
| Setup | Pemicu | Stop loss |
|---|---|---|
| **TK Cross** (Bab 6) | Tenkan memotong Kijun ke atas, **ditunda 1–2 candle**, harga tetap naik dan di atas Kumo, Kumo depan bullish, Kijun naik, Chikou bebas (kategori *Kuat*) | Di bawah Kijun − 0,3 ATR (pakai swing low 5 bar jika terlalu rapat) |
| **Kumo Breakout** (Bab 7) | Close pertama di atas Kumo + candle berikutnya close lebih tinggi (**2 candle searah**), body breakout ≥ 0,5 ATR, Tenkan > Kijun, Kumo depan bullish, Chikou bebas | Di bawah Kumo − 0,3 ATR |
| **Pullback Kijun** (Bab 8, trend riding) | ≥ 10 dari 12 close terakhir di atas Kumo, Kumo tidak tipis, Kijun naik; pullback menyentuh Tenkan/Kijun tanpa masuk jauh ke Kumo, lalu candle bullish memantul | Di bawah swing low 5 bar − 0,3 ATR |

Semua setup dibatalkan jika jarak stop > 4 ATR.
**Plan:** TP1 = 1R (ambil 1/3, stop ke entry) · TP2 = 2R (ambil 1/3) · sisa 1/3 trailing di bawah Kijun; keluar saat **close di bawah Kijun** (bisa diganti ke *TK cross bearish* di Pengaturan).

### 3. Setup Fibonacci + price action
1. **Tarik Fib** dari swing low ke swing high terakhir yang **sudah terkonfirmasi** (fractal 5 bar), kaki ≥ 3 ATR, dan swing high harus **higher high** (tidak menarik Fib di pasar sideways).
2. Harga **menyentuh** 38.2 / 50 / 61.8% (78.6% opsional) lalu **close kembali di atasnya**.
3. **Konfirmasi candle wajib:** pin bar / bullish engulfing, atau pin bar sebelumnya + close di atas high-nya.
4. **Skor konfluensi ≥ 3 dari 5:** level Fib (+1), S/R (swing lama, Kijun, atau atap Kumo dalam 0,5 ATR) (+1), candle (+1), HTF searah (+1), volume ≥ rata-rata 20 bar (+1).
5. Harga tidak di bawah Kumo; kaki naik batal jika ada close di bawah 78.6%.
6. **Reward:risk ke extension 127.2% ≥ 1:2.**

**Plan:** SL di bawah low pullback − 0,3 ATR · TP1 = swing high (ambil 50%, stop ke entry) · TP2 = 127.2% (ambil 30%) · TP3 = 161.8% (20%, trailing di bawah setiap swing low baru).

### 4. Skor 0–100
- **Ichimoku:** posisi vs Kumo (15), Kumo depan (10), Tenkan > Kijun (10), kemiringan Kijun (10), Chikou (15), ketebalan Kumo (10), jarak sehat ke Kijun (15), HTF (10), volume (5).
- **Fibonacci:** skor konfluensi × 12 (maks 60), bobot level (61.8 = 15, 50 = 11, 38.2 = 8), di atas Kumo (10), Fib berhimpit Kijun/Kumo (10), RR (5).
- Jika beberapa setup muncul di satu pair, yang ditampilkan skor tertinggi.

### 5. Checklist Ichimoku × Fibonacci (x/8)
HTF di atas Kumo · harga di atas Kumo · Tenkan > Kijun & Kijun naik · Chikou bebas · Kumo depan bullish · pullback masuk zona 38.2–61.8 · level Fib berhimpit Kijun/Kumo/S-R · pola candle bullish. Informasi konfluensi, tidak mengubah sinyal.

### 6. Probabilitas & keyakinan
Dihitung dari mesin backtest yang sama pada 2.000 bar: % sinyal lampau yang mencapai TP1, TP2, atau kena SL dulu. Keyakinan = batas bawah Wilson 95% dari peluang TP1 (Tinggi ≥ 50%, Sedang ≥ 35%, sampel < 10 = "sampel sedikit"). Frekuensi masa lalu, **bukan jaminan**.

---

## Hasil uji (22 Nov 2026)

**Uji teknis (lulus):**
- **Look-ahead:** 1.555 titik di 5 pair 4H diuji ulang dengan data masa depan dipotong (termasuk 29 sinyal): **0 selisih**. Swing Fibonacci baru dipakai setelah terkonfirmasi 5 bar.
- **Invarian backtest:** 0 pelanggaran (urutan waktu, posisi tidak tumpang tindih, kerugian tidak melewati −1R kecuali gap).
- **Browser:** dijalankan di Chrome (desktop & HP, terang & gelap, EN & ID) dengan data Binance live dan mode demo, tanpa error JavaScript.

**Uji strategi dengan data Binance asli** (10 pair besar: BTC, ETH, SOL, BNB, XRP, ADA, AVAX, LINK, DOGE, LTC; 3.000 bar; fee 0,1% + slippage 0,05% per sisi; risiko 1%/trade; filter MTF aktif):

| TF | Strategi | Trade | Win | Profit factor | Ekspektansi | t-stat |
|---|---|---|---|---|---|---|
| 4H | Ichimoku | 134 | 38% | 0,83 | −0,07R | −0,85 |
| 4H | Fibonacci | 131 | 33% | 0,87 | −0,10R | −0,71 |
| 1D | Ichimoku | 166 | 34% | 1,73 | +0,26R | 1,68 |
| 1D | Fibonacci | 158 | 40% | 1,34 | +0,21R | 1,53 |

Per setup di 1D: Pullback Kijun 149 trade, +0,31R (t 1,78); Fib 38.2 44 trade, +0,49R (t 2,02); Fib 61.8 66 trade, +0,31R; Fib 50 48 trade, −0,18R. TK Cross dan Kumo Breakout jarang muncul (13 dan 4 trade di 1D), sampelnya terlalu kecil untuk dinilai.

**Kesimpulan:** di **4H** kedua strategi belum punya edge setelah biaya. Di **1D** keduanya positif, tetapi belum signifikan secara statistik (t < 2, kecuali Fib 38.2 yang tipis di atas 2 dengan 44 trade). Exit "TK cross bearish" memberi hasil hampir sama dengan exit "close di bawah Kijun". Pakai **1D** sebagai timeframe utama, forward test dengan nominal kecil, dan jangan jadikan sinyal otomatis.

**Batasan:**
- Long-only: saat pasar di bawah Kumo (seperti saat diuji, 20 dari 20 pair watchlist di bawah Kumo 4H), aplikasi memang tidak memberi sinyal.
- Auto-scan hanya berjalan selama halaman terbuka.
- Backtest dan probabilitas tidak memakai filter BTC risk-off, hanya filter MTF per pair.

---

## Cara upload (GitHub Pages)

1. Login ke GitHub → **New repository**, misalnya `ichimoku-fibonacci-framework`, set **Public**.
2. **Add file → Upload files**: upload `index.html` dan `README.md` → **Commit changes**.
3. **Settings → Pages** → Source *Deploy from a branch*, branch `main`, folder `/ (root)` → **Save**.
4. Tunggu 1–2 menit, buka `https://<username>.github.io/ichimoku-fibonacci-framework/`.
5. Di HP: buka di Chrome → menu **⋮ → Add to Home screen**.

Pengaturan, alert, dan token disimpan dengan prefix `tsif_` di localStorage, jadi tidak bentrok dengan Tradesys Edge Dual di domain `haedar-collab.github.io` yang sama.

---

## Setup Telegram

1. Chat **@BotFather** → `/newbot` → salin **bot token**.
2. Kirim pesan ke bot, buka `https://api.telegram.org/bot<TOKEN>/getUpdates`, salin `"chat":{"id": ...}`.
3. **Pengaturan → Data dan Telegram**: isi token & chat ID, centang *Kirim sinyal live baru ke Telegram*, **Simpan dan scan ulang**.
4. Tab **Alert**: *Kirim pesan tes* dan *Kirim sinyal tes*.

Pesan berisi entry, SL, TP1–TP3 dengan persen, ukuran posisi, plan manajemen, bacaan Ichimoku (Kumo, TK, Chikou, Kijun), level Fib, skor konfluensi, checklist, dan probabilitas historis.

**Keamanan:** token hanya di localStorage browser, tidak ikut ter-upload. Jangan menulis token di `index.html`.

---

## Struktur kode (`index.html`)

- `ENGINE-START … ENGINE-END`: indikator, `ichiCore()` (Tenkan, Kijun, Senkou A/B, Kumo), `pivots()` (swing terkonfirmasi), `ichiAt()`, `fibAt()`, `evalAt()` (semua setup), `backtest()`, `htfStates()`, SMC & pola candle, data demo. Bisa dipakai ulang di Node.js untuk scheduler.
- Data layer: `api()`, `getKlines()` (hanya candle yang sudah close), `universe()` (stablecoin & leveraged token dikeluarkan).
- UI: render tiap tab, chart SVG, kalkulator risiko, alert, pengaturan.

*Bukan nasihat keuangan. Uji setiap strategi dengan data historis dan forward test dengan nominal kecil sebelum dipakai dengan modal sungguhan.*
