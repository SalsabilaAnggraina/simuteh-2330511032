# 🍃 SIMUTEH: Sistem Pemeriksaan Mutu Daun Teh Berbasis Citra
> **Laporan Praktikum Berkelanjutan Pertemuan 1–6 Mata Kuliah Computer Vision**  
> **Program Studi Teknik Informatika, Fakultas Sains dan Teknologi, Universitas Muhammadiyah Sukabumi (TA 2026/2027)**

---

## 📌 Identitas Peneliti & Repositori
* **Nama Lengkap**: Salsabila Anggraina Putri
* **NIM / Kelas**: 2330511032 / Teknik Informatika 7A
* **SEED Dataset**: `1032`
* **Dosen Pengampu**: Asriyanik, M.T.
* **Tautan Repositori**: [https://github.com/SalsabilaAnggraina/simuteh-2330511032](https://github.com/SalsabilaAnggraina/simuteh-2330511032)
* **Release Tags**: `v0.1-praproses` $\rightarrow$ `v0.2-tepi` $\rightarrow$ `v0.3-morfologi`

---

## 📝 1. Pendahuluan & Formulasi Masalah

### 💡 Ringkasan Kasus & Problem Statement
Penyortiran manual pucuk daun teh di nampan oleh kelompok tani Kabupaten Sukabumi sering terkendala ketidakseragaman penilaian dan kelambatan akibat variasi cahaya lokasi serta gejala penyakit cacar daun (*Exobasidium vexans*). **SIMUTEH** dibangun sebagai sistem berbasis visi komputer yang menerima foto daun teh di atas nampan dan menghasilkan klasifikasi kelas mutu (**Sehat**, **Bercak Ringan**, **Bercak Berat**) serta jumlah, luas ($\text{mm}^2$), dan persentase luas bercak secara cepat, objektif, dan transparan tanpa mengesampingkan keputusan akhir petugas.

### 🎯 Alur Pipeline SIMUTEH (`v0.1` – `v0.3`)
1. **Akuisisi & Praproses (`P1–P4`, `v0.1-praproses`)**: Standardisasi jarak pemotretan, koreksi pencahayaan (*gamma* otomatis & CLAHE adaptif kanal V), serta penapisan derau (*median* $3\times3$) melalui `src/praproses.py`.
2. **Deteksi Tepi (`P5`, `v0.2-tepi`)**: Evaluasi gradien dan operator tepi (Sobel, Prewitt, LoG, Canny) serta perbaikan ambang adaptif berbasis persentil magnitudo.
3. **Morfologi & Analisis Bercak (`P6`, `v0.3-morfologi`)**: Pembersihan citra biner bercak dengan operasi morfologi (*opening* & *closing*), pelabelan komponen terhubung, pengukuran geometri ($\text{px}$ dan $\text{mm}^2$), penyaringan bercak palsu, dan klasifikasi awal melalui `src/bercak.py`.

---

## 📐 2. Syarat Pemotretan & Ruang Warna Terbaik (P1–P2)

* **Hasil Lembar Hitung Akuisisi**: Pemotretan ideal dilakukan pada jarak **$20 - 30\text{ cm}$** tegak lurus di atas nampan dengan resolusi kerja lebar **$640\text{ px}$** (pada jarak $200\text{ mm}$, skala konversi adalah **$1\text{ piksel} = 0{,}4375\text{ mm}$** atau $0{,}1914\text{ mm}^2$).
* **Kanal Warna Terpilih**: Kanal **Hue (H)** pada HSV dan **Blue (B)** memberikan Kontras Bercak (KB) tertinggi pada tahap awal, sedangkan **Kanal Red (R)** terpilih untuk deteksi tepi dan segmentasi bercak karena klorofil daun menyerap cahaya merah secara homogen (menekan kontras tulang daun) sementara bercak kecokelatan memantulkan merah dengan kuat.

---

## 📊 3. Perbaikan Citra & Penapisan Spasial (P3–P4)

* **Perbaikan Pencahayaan**: Penerapan koreksi *gamma* otomatis dan **CLAHE pada Kanal V** (HSV) secara adaptif saat indeks ketidakrataan cahaya $> 0{,}65$ efektif mengeliminasi gradien bayangan lokal tanpa menggeser warna asli daun.
* **Penapisan Derau**: Filter **Median $3 \times 3$** terpilih sebagai penapis paling seimbang karena cepat ($0{,}08\text{ ms}$) dan mampu meredam derau impuls tanpa mengerosi detail bercak kecil kelas ringan ($dI$ terjaga di $37{,}10$).

---

## ⚙️ 4. Evaluasi Integrasi Praproses Final (`v0.1-praproses`)

Rangkaian praproses dalam `src/praproses.py` dengan konfigurasi `hasil/param_praproses.json` berhasil menaikkan Kontras Bercak (KB) pada seluruh kondisi pencahayaan:

| Kondisi Cahaya | KB Sebelum | KB Sesudah | Perubahan (%) | Catatan Ringkas Analisis |
| :--- | :---: | :---: | :---: | :--- |
| **bayangan** | $1.31$ | $2.71$ | $+106.9\%$ | CLAHE kanal V mengeliminasi gradien bayangan secara adaptif. |
| **gelap** | $1.14$ | $2.12$ | $+86.0\%$ | Filter median $3\times3$ meredam derau sensor dan menaikkan KB. |
| **kontras_rendah** | $0.58$ | $1.96$ | **$+237.9\%$** | **Lonjakan Tertinggi**: Memunculkan kontras bercak yang semula tersamar ($KB < 1$). |
| **normal** | $1.97$ | $3.62$ | $+83.8\%$ | **KB Akhir Tertinggi**: Pemisahan tepi bercak dari daun sangat tegas. |
| **terang** | $1.71$ | $2.44$ | $+42.7\%$ | Meredam pantulan silau (*glare*), meski peningkatannya terbatas akibat saturasi piksel. |

---

## 🔍 5. Deteksi Tepi & Keputusan Arsitektur (P5 — `v0.2-tepi`)

Evaluasi kuantitatif pada 40 citra berbercak (dengan toleransi jarak 2 piksel terhadap tepi acuan) menghasilkan temuan utama berikut:
* **Operator & Kanal Terbaik**: **Canny (`30/90`, $\sigma=2{,}0$) pada Kanal R sesudah praproses** menghasilkan skor $F_1$ tertinggi dan menekan deteksi tepi palsu pada tulang daun sehat dibandingkan kanal Gray.
* **Perbaikan Ambang Otomatis (`canny_persentil`)**: Ambang otomatis berbasis median intensitas gagal pada citra kontras rendah karena nilai median intensitasnya tinggi sementara gradien tepinya lemah. Penggunaan persentil magnitudo gradien Sobel ($q=95$, rasio $0{,}4$) terbukti jauh lebih stabil lintas kondisi cahaya.
* **Keputusan Pemakaian Tepi**: Deteksi tepi Canny **tidak digunakan** untuk mengisi wilayah segmentasi bercak karena garis batas selebar 1 piksel berisiko terbuka (*open contour*) pada bercak samar dan memperlambat komputasi. Segmentasi wilayah dilakukan menggunakan ambang intensitas dan operasi morfologi.

---

## 🧩 6. Operasi Morfologi, Pengukuran & Modul `bercak.py` (P6 — `v0.3-morfologi`)

### 🛠️ Rangkaian Pembersihan & Penyaringan
1. **Mask Awal (`utils_tepi.py`)**: Pemisahan daun dengan ambang Otsu pada kanal S (`mask_daun_awal`) dan kandidat bercak awal pada kanal R dengan ambang `median + 4.0 * MAD` (`bercak_awal`).
2. **Operasi Morfologi**: Kombinasi **Opening** lalu **Closing** menggunakan *structuring element* **Elips $5 \times 5$** untuk membuang titik derau dan tepi tipis tanpa menghapus bercak kecil, sekaligus menutup lubang di dalam bercak.
3. **Komponen Terhubung & Penyaringan (`konektivitas = 8`)**: Setiap kandidat diukur luas ($\text{px}$ & $\text{mm}^2$), keliling, kebulatan ($\frac{4\pi A}{K^2}$), dan rasio aspek, lalu disaring dengan aturan `luas_min >= 20 px`, `bulat_min >= 0.5`, dan `rasio_maks <= 1.8` untuk mengeliminasi sisa potongan tulang daun memanjang.

### 📈 Ringkasan Evaluasi `bercak.py` pada 60 Citra
* **Galat Absolut Rerata Persen Bercak**: `[isi nilai dari Kode 5.2]`%
* **Korelasi Persen Prediksi vs Acuan**: `[isi nilai dari Kode 5.2]`
* **Akurasi Klasifikasi Kelas Mutu**: `[isi nilai dari Kode 5.2]`
* **Kondisi Cahaya dengan Akurasi Terendah**: `[isi dari Kode 5.2]`

---

## 💻 7. Cara Menggunakan Modul `analisis_daun`

Pastikan direktori kerja berada di folder `notebook/` atau tambahkan jalur `src/` ke `sys.path`, lalu jalankan rangkaian `praproses` dan `analisis_daun`:

```python
import sys
import json
sys.path.append("src")

from utils_cv import baca
from praproses import praproses
from bercak import analisis_daun

# 1. Muat citra dan konfigurasi parameter final
img = baca("data/sintetis/citra/teh_041.png")
param_pra = json.load(open("hasil/param_praproses.json"))
param_bercak = json.load(open("hasil/param_bercak.json"))

# 2. Jalankan praproses (P1-P4) dilanjutkan analisis bercak (P5-P6)
img_bersih = praproses(img, param_pra)
hasil = analisis_daun(img_bersih, param_bercak)

# 3. Tampilkan ringkasan hasil pemeriksaan daun
print(f"Kelas Mutu    : {hasil['kelas']}")
print(f"Jumlah Bercak : {hasil['jumlah_bercak']} bercak")
print(f"Persen Bercak : {hasil['persen_bercak']}%")
for b in hasil["bercak"]:
    print(f"  - Bercak #{b['label']}: luas={b['luas_mm2']} mm² ({b['luas_px']} px), kebulatan={b['kebulatan']}")
```

---

## 📁 8. Struktur Repositori

```text
simuteh-1032/
├── data/
│   └── sintetis/                  # Dataset 60 citra sintetis (SEED 1032) - tidak diunggah ke GitHub
├── hasil/
│   ├── param_praproses.json       # Parameter final praproses (P1-P4)
│   ├── param_bercak.json          # Parameter final analisis bercak (P5-P6)
│   ├── p5_tabel_tepi.csv          # Hasil evaluasi 4 operator tepi pada 40 citra berbercak
│   ├── p5_f1_kondisi.png          # Grafik batang F1 tepi per kondisi cahaya
│   ├── p6_eksperimen_se.csv       # Hasil eksperimen k, bentuk, dan ukuran SE (60 citra)
│   └── p6_evaluasi_bercak.csv     # Hasil evaluasi akhir modul bercak.py (60 citra)
├── laporan/                       # Berkas Laporan PDF & Slide Presentasi PPTX
├── notebook/
│   ├── B1.ipynb .. B5.ipynb       # Notebook eksperimen Pertemuan
