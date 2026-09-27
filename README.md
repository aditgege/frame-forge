# FRAMEFORGE

Editor animasi aset isometrik berbasis browser. Ambil **satu PNG transparan**
berisi beberapa frame, lalu FRAMEFORGE memecahnya, merapikan anchor, mengecek
loop, dan mengeluarkan `spritesheet.png` + `anim.json` + `preview.gif` yang
siap dipakai engine.

```
split → trim → anchor → loop → sheet
```

- **Nol dependensi.** Satu file `index.html`, tanpa build step, tanpa npm.
- **Nol server.** Semua proses berjalan di browser (Canvas + WebAssembly-free JS murni).
- **Data tidak keluar dari mesin.** Tidak ada `fetch`, tidak ada upload.

---

## Cara pakai

### 1. Jalankan

**Cara tercepat — tanpa server.** Cukup buka `index.html` di browser
(Chrome/Edge/Firefox modern). Aplikasinya satu file tanpa `fetch`/ES module,
jadi `file://` sudah cukup.

```bash
# klik dua kali index.html, atau:
start index.html          # Windows
open index.html           # macOS
xdg-open index.html       # Linux
```

**Cara dengan `npx serve .`** — direktori lokal dilayani lewat HTTP statis.
Berguna kalau mau sering reload lewat DevTools, atau menguji dari perangkat
lain di jaringan yang sama.

```bash
# butuh Node.js — cek dengan: node -v
npx serve .
# → Local:   http://localhost:3000
# → Network: http://192.168.x.x:3000   (untuk tes di HP)
```

Buka URL yang ditampilkan terminal, lalu `Ctrl+C` untuk menghentikan server.
Opsi berguna:

```bash
npx serve . -l 8080        # ganti port
npx serve . --no-clipboard # jangan salin URL ke clipboard
```

Alternatif tanpa Node.js:

```bash
python -m http.server 8080 # lalu buka http://localhost:8080
php -S localhost:8080
```

> Kalau hanya butuh menguji, `file://` sudah cukup — `npx serve` tidak
> diwajibkan.

Kalau mau cepat melihat alurnya tanpa file apa pun, klik **demo bersih** atau
**demo drift** — keduanya membuat PNG contoh di dalam browser.

### 2. Siapkan PNG input

| Kebutuhan | Nilai |
| --- | --- |
| Format | PNG, **straight alpha** (bukan premultiplied) |
| Background | transparan — **cutout tidak perlu** |
| Isi | 2–8 frame arranged `strip H`, `strip V`, atau `grid` |
| Jarak antar frame | boleh nol, atau pisahkan dengan celah transparan |

Muat file lewat **Pilih PNG…** atau langsung **seret ke jendela mana saja**.

### 3. Cek hasil split

Panel kiri → **Pembagian frame**. Default `auto` menebak jumlah frame dari celah
transparan antar frame. Kalau tebakan meleset (frame saling menempel), isi
manual salah satu dari:

- **Jumlah frame** → pembagian rata (butuh `Layout` strip/grid)
- **Kolom / Baris** → khusus `Layout: grid`
- **Ukuran frame (W × H)** → memotong tetap per sel
- **Gap antar frame** → jarak dalam piksel antar sel

Status deteksi dan catatan rencana potongan tampil di kotak `detected`.

### 4. Atur kanvas & anchor

Preset cepat: `auto · bbox`, `tile 128×64`, `objek 128×160`, `karakter`,
`gedung 256×320`. Preset hanya mengisi angka — mengedit angka secara manual
otomatis melepas preset.

- **Mode anchor** — `bottom-center` (default, kaki menyentuh `y = H - sink`) atau
  `center`.
- **Sink** — sisa ruang di bawah anchor, default `6` px.
- **Skala seragam** — seluruh frame di-scale dengan faktor yang sama; kalau
  konten tidak muat, aplikasi auto-fit turunkan semuanya secara seragam
  (tidak pernah hanya sebagian frame yang di-scale).
- **Resample** — `nearest` (pixel art) atau `bilinear`.

### 5. Verifikasi

Bar bawah menampilkan **checklist §19** — split · anchor · loop · transparansi.
Buka/tutup dengan chevron di kanan. Status `ok` / `peringatan` / `gagal`
ikut menentukan warna di pill status topbar.

Lima tab pratinjau:

| Tab | Isi |
| --- | --- |
| **Animasi** | playback frame sesuai kanvas final |
| **Onion** | semua frame ditumpuk 50% + garis anchor — mendeteksi kaki yang bergeser |
| **Edge 400%** | tepi alpha 4× di atas terang/gelap — mendeteksi halo |
| **Sheet** | spritesheet horizontal, semua frame bersebelahan |
| **Sumber** | PNG asli, belum dipecah |

### 6. Unduh

- Tombol **`Unduh semua .zip`** di topbar — paket lengkap:

```
<nama>_anim.zip
└── <nama>/
    ├── frames/frame_00.png … frame_NN.png
    ├── spritesheet.png
    ├── anim.json
    ├── preview.gif
    ├── preview_onion.png
    └── preview_edge.png
```

- Tombol individual di panel **Unduhan** untuk `spritesheet.png`, `anim.json`,
  `preview.gif`, `preview_onion`, `preview_edge`, dan **salin JSON** ke clipboard.
- Klik ikon unduh pada thumbnail filmstrip untuk mengambil satu frame.

---

## Pintasan keyboard

| Tombol | Aksi |
| --- | --- |
| `Spasi` | putar / jeda |
| `←` `→` | mundur / maju 1 frame |
| `1` – `5` | ganti tab pratinjau |
| `+` `-` | zoom |
| `F` | fit ke layar |
| scroll | zoom di posisi kursor |
| klik + seret | pan |
| klik ganda | fit |

---

## Format `anim.json`

```jsonc
{
  "name": "fire_loop",
  "frame_w": 128,
  "frame_h": 64,
  "count": 6,
  "anchorX": 64,
  "anchorY": 58,
  "fps": 10,
  "loop": true,
  "pingpong": false,
  "frames": [
    { "x": 0,   "y": 0, "w": 128, "h": 64 },
    { "x": 128, "y": 0, "w": 128, "h": 64 }
  ],
  "meta": {
    "source": "fire_strip.png",
    "layout": "strip-h",
    "anchor_mode": "bottom-center",
    "sink": 6,
    "scale": 1,
    "resample": "nearest",
    "alpha_threshold": 12,
    "generator": "frameforge-web (isometric-game-skills §18)"
  }
}
```

`frames[i].x` adalah offset di dalam `spritesheet.png`. `anchorX`/`anchorY`
adalah titik jangkar sprite di dalam satu frame — pakai ini untuk menempatkan
sprite tepat di atas tile.

---

## Pipeline internal

1. **Split** — `planCells()` menebak layout memakai run piksel non-transparan;
   fallback ke pembagian rata / ukuran tetap.
2. **Trim** — `bboxOf()` memotong setiap frame ke kotak konten di atas
   `Ambang alfa`.
3. **Normalisasi** — semua frame ditempatkan ke kanvas dengan ukuran identik
   (potongan integer) pada anchor yang sama.
4. **Loop check** — MAD antara frame bersebelahan dibandingkan dengan
   frame terakhir → frame 0. Lompatan besar = peringatan, bukan error.
5. **Artefak** — sheet, onion, edge, dan GIF (encoder GIF89a + LZW + median-cut
   palet ditulis dari nol di dalam file).

---

## Struktur proyek

```
.
├── index.html   # seluruh aplikasi (HTML + CSS + JS, satu file)
└── README.md
```

Tidak ada modul terpisah yang perlu dicompile. Kalau ingin memecah,
`index.html` punya tiga bagian yang jelas: `<style>` (CSS),
`<body>` (markup), `<script>` (logika).

---

## Troubleshooting

**"Belum ada file"** — muat PNG lebih dulu; tanpa input tidak ada pipeline.

**Split salah jumlah frame** — pindah `Layout` dari `auto` ke `strip H`/`strip V`/`grid`,
lalu isi jumlah frame atau ukuran frame manual.

**Kaki sprite bergeser antar frame** — buka tab **Onion**. Garis anchor
horizontal akan memotong kaki di frame berbeda → normalkan sumber, atau naikkan
`Ambang alfa` jika ada bayangan transparan yang ikut terpotong.

**Halo putih di tepi sprite** — buka tab **Edge 400%**. Jika tepi memiliki
semi-transparan yang menyatu dengan background, bersihkan alpha di sumber;
`Ambang alfa` hanya untuk bbox, bukan untuk fighting/halo removal.

**GIF terlihat berkedip** — encoder GIF memakai palet terkuantisasi
(median-cut, maks 255 warna) dan transparansi biner. Untuk kualitas warna
penuh, pakai `spritesheet.png` + `anim.json`, bukan GIF.

**Gagal build ZIP di browser lama** — butuh `TextEncoder`, `Blob`, dan
`createImageBitmap`. Chrome/Edge/Firefox versi terbaru sudah mendukung.

---

## Lisensi

Alat internal — bebas dipakai dan dimodifikasi.
