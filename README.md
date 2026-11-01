# 🍃 SIMUTEH: Sistem Pemeriksaan Mutu Daun Teh Berbasis Citra
> **Ringkasan Laporan Praktikum Evaluasi Pertemuan 1–4 Mata Kuliah Computer Vision**  
> **Program Studi Teknik Informatika, Fakultas Sains dan Teknologi, Universitas Muhammadiyah Sukabumi (TA 2026/2027)**

---

## 📌 Identitas Peneliti & Repositori
* **Nama Lengkap**: Salsabila Anggraina Putri
* **NIM / Kelas**: 2330511032 / Teknik Informatika 7A
* **SEED Dataset**: `1032`
* **Dosen Pengampu**: Asriyanik, M.T.
* **Tautan Repositori**: [https://github.com/SalsabilaAnggraina/simuteh-1032](https://github.com/SalsabilaAnggraina/simuteh-1032)
* **Release Tag**: `v0.1-praproses`

---

## 📝 1. Pendahuluan & Formulasi Masalah

### 💡 Ringkasan Kasus & Problem Statement
Penyortiran manual pucuk daun teh di nampan oleh kelompok tani Kabupaten Sukabumi sering terkendala ketidakseragaman penilaian dan kelambatan akibat variasi cahaya lokasi serta gejala penyakit cacar daun (*Exobasidium vexans*)[cite: 34]. **SIMUTEH** dibangun sebagai sistem berbasis visi komputer yang menerima foto daun teh di atas nampan melalui kamera *smartphone* dan menghasilkan klasifikasi kelas mutu (**Sehat**, **Bercak Ringan**, **Bercak Berat**) serta persentase luas bercak secara cepat, objektif, dan terstandarisasi tanpa mengesampingkan keputusan akhir petugas[cite: 34].

### 🎯 Ringkasan Tujuan Kegiatan
1. Merumuskan rencana akuisisi citra daun teh sesuai batas jarak dan resolusi pemotretan[cite: 34].
2. Menentukan kanal warna optimal untuk memisahkan bercak dari permukaan daun[cite: 34].
3. Mendiagnosis dan memperbaiki gangguan pencahayaan berbasis histogram dan CLAHE adaptif[cite: 34].
4. Mengevaluasi penapis spasial untuk meredam derau tanpa merusak detail bercak kecil[cite: 34].
5. Mengintegrasikan seluruh praproses ke modul `src/praproses.py` dan konfigurasi JSON terstandar[cite: 34].
6. Menerapkan etika *Responsible Computer Vision* dan nilai AIK (Siddiq & Amanah)[cite: 34, 59].

---

## 📐 2. Syarat Pemotretan & Ruang Warna Terbaik (P1-P2)

* **Hasil Lembar Hitung Akuisisi**: Pemotretan ideal dilakukan pada jarak **$25 - 30\text{ cm}$** tegak lurus di atas nampan dengan resolusi minimum **$640 \times 480\text{ px}$**[cite: 39]. Rekomendasi ini menjamin diameter bercak terkecil ($5\text{ mm}$) tetap terwakili oleh minimal 10 piksel[cite: 38, 39].
* **Kanal Warna Terpilih**: Kanal **Hue (H)** pada HSV dan kanal **Blue (B)** pada BGR memberikan nilai Kontras Bercak (KB) tertinggi[cite: 41, 58]. Kanal H terbukti paling konsisten mengisolasi informasi warna biologis bercak kecokelatan dari daun hijau di seluruh kondisi pencahayaan[cite: 41, 58].

---

## 📊 3. Perbaikan Citra & Penapisan Spasial (P3-P4)

* **Perbaikan Pencahayaan**: Penerapan **CLAHE pada Kanal V** (HSV) dengan `clipLimit=2.0` dan `tileGridSize=(8,8)` terbukti paling efektif mengeliminasi gradien bayangan lokal tanpa merusak warna asli daun[cite: 49, 58].
* **Penapisan Derau**: Filter **Median $3 \times 3$** terpilih sebagai penapis paling seimbang. Sangat cepat ($0.08\text{ ms}$) dan mampu mengeliminasi derau impuls secara sempurna tanpa mengerosi detail bercak kecil kelas ringan ($dI$ terjaga di $37.10$)[cite: 52, 54, 58].

---

## ⚙️ 4. Evaluasi Integrasi Praproses Final (P5)

Rangkaian praproses yang diintegrasikan ke dalam `src/praproses.py` dengan parameter terkonfigurasi `hasil/param_praproses.json` berhasil menaikkan Kontras Bercak (KB) pada seluruh kondisi pencahayaan[cite: 55, 58]:

| Kondisi Cahaya | KB Sebelum | KB Sesudah | Perubahan (%) | Catatan Ringkas Analisis |
| :--- | :---: | :---: | :---: | :--- |
| **bayangan** | $1.31$ | $2.71$ | $+106.9\%$ | CLAHE kanal V mengeliminasi gradien bayangan secara adaptif[cite: 55]. |
| **gelap** | $1.14$ | $2.12$ | $+86.0\%$ | Filter median $3\times3$ meredam derau sensor dan menaikkan KB[cite: 55]. |
| **kontras_rendah** | $0.58$ | $1.96$ | **$+237.9\%$** | **Lonjakan Tertinggi**: Memunculkan kontras bercak yang semula tersamar ($KB < 1$)[cite: 55]. |
| **normal** | $1.97$ | $3.62$ | $+83.8\%$ | **KB Akhir Tertinggi**: Pemisahan tepi bercak dari daun sangat tegas[cite: 55]. |
| **terang** | $1.71$ | $2.44$ | $+42.7\%$ | Meredam pantulan silau (*glare*), meski peningkatannya terbatas akibat saturasi piksel[cite: 55, 56]. |

---

## 📁 5. Struktur Repositori

```text
simuteh-1032/
├── data/
│   └── sintetis/              # Dataset 60 citra sintetis (SEED 1032)
├── hasil/                     # CSV evaluasi, grafik PNG, & param_praproses.json
├── laporan/                   # Berkas Laporan PDF & Slide Presentasi PPTX
├── notebook/                  # Notebook eksperimen Colab (B1 s.d. B5)
├── src/                       # Modul Python (buat_dataset_sintetis.py, utils_cv.py, praproses.py)
├── README.md                  # Ringkasan eksekutif reprodusibilitas
└── requirements.txt           # Dependensi pustaka Python
