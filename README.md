# Gestura - Terminal-Based Sign Language Translator for ASL

**Gestura** adalah aplikasi berbasis terminal (*Command Line Interface* / CLI) yang dirancang untuk mendeteksi dan menerjemahkan **American Sign Language (ASL)** secara *real-time* melalui kamera (*webcam*) menjadi teks dan keluaran suara (*Text-to-Speech*). 

Proyek ini dikembangkan sebagai tugas akhir mata kuliah **Pengolahan Citra Digital**, menggabungkan teknik ekstraksi fitur titik kunci tangan menggunakan **MediaPipe** dan klasifikasi pola menggunakan **Multi-Layer Perceptron (MLP)** serta **Long Short-Term Memory (LSTM)**.

---

## Fitur Utama

* **Deteksi Tangan Real-Time:** Memanfaatkan *MediaPipe Hands* untuk melacak 21 titik koordinat (*landmarks*) tangan secara akurat dan cepat.
* **Dukungan Alfabet Statis & Dinamis (A-Z):** Mampu membaca gestur alfabet statis (`A-I`, `K-Y`) maupun alfabet dinamis berbasis gerakan (`J` dan `Z`).
* **Pre-Processing Invariant:** Dilengkapi normalisasi geometris (*wrist centering* dan *scale normalization*) sehingga deteksi tetap stabil meskipun jarak dan posisi tangan berubah di depan kamera.
* **Trigger "Hands Down" Otomatis:** Mengumpulkan rangkaian huruf ke dalam array selama tangan terdeteksi, lalu otomatis menggabungkannya menjadi satu kata utuh saat tangan diturunkan (*Hands Down*).
* **Seamless Text-to-Speech (TTS):** Membacakan hasil terjemahan kata atau kalimat secara utuh dan lancar tanpa mengeja per huruf.
* **Pengembangan Vocabulary Reading (Tahap 2):** Arsitektur lanjutan berbasis LSTM untuk menerjemahkan frasa kosakata utuh dalam bentuk video gerakan.

---

## Teknologi yang Digunakan

* **Bahasa Pemrograman:** Python 3.9+
* **Computer Vision & Image Processing:** OpenCV (`opencv-python`), MediaPipe (`mediapipe`)
* **Machine Learning & Deep Learning:** Scikit-Learn / TensorFlow (Keras)
* **Pengolahan Data Numerik:** NumPy, Pandas
* **Text-to-Speech (TTS):** `pyttsx3` (Offline TTS)

---

## Struktur Direktori Proyek

```text
Gestura--Terminal-Based-Sign-Language-Translator-for-ASL/
├── data_training/
│   ├── asl_alphabets/        # Dataset gambar (A-I, K-Y) dan video (J, Z)
│   └── asl_vocabulary/       # Dataset video frasa kosakata (Tahap 2)
├── documents/
│   ├── gestura_SRS.md        # Dokumen spesifikasi kebutuhan sistem utama
│   └── pre-processing_SRS.md # Dokumen spesifikasi teknis modul pre-processing
├── src/
│   ├── preprocessing_utils.py # Fungsi ekstraksi landmarks & normalisasi geometris
│   ├── extract_dataset.py     # Skrip pembuatan dataset CSV dari folder data_training
│   ├── train_alphabet.py      # Skrip pelatihan model MLP untuk alfabet A-Z
│   └── main.py                # Aplikasi utama berbasis terminal (Live Detection & TTS)
├── models/                    # Direktori penyimpanan model terlatih (.h5 / .pkl)
├── requirements.txt           # Daftar pustaka Python yang dibutuhkan
├── LICENSE                    # Lisensi MIT
└── README.md                  # Dokumentasi utama proyek
```
---

## Cara Instalasi & Menjalankan (Clone)

Ikuti langkah-langkah berikut untuk menyalin (*clone*) dan menjalankan proyek ini di komputer lokal Anda:

### 1. Clone Repositori
Buka terminal atau *Command Prompt*, lalu jalankan perintah berikut:
```bash
git clone https://github.com
```

### 2. Masuk ke Direktori Proyek
```bash
cd Gestura--Terminal-Based-Sign-Language-Translator-for-ASL
```

### 3. Buat Virtual Environment (Direkomendasikan)
```bash
# Untuk Windows
python -m venv venv
venv\Scripts\activate

# Untuk macOS / Linux
python3 -m venv venv
source venv/bin/activate
```

### 4. Instal Dependensi / Pustaka
Pastikan semua pustaka yang dibutuhkan terinstal dengan menjalankan:
```bash
pip install opencv-python mediapipe numpy pandas scikit-learn tensorflow pyttsx3
```
*(Atau jika sudah ada file `requirements.txt`, jalankan: `pip install -r requirements.txt`)*

### 5. Jalankan Aplikasi
Pastikan *webcam* komputer Anda terhubung dan tidak sedang digunakan oleh aplikasi lain, lalu jalankan:
```bash
python main.py
```

---

## Cara Penggunaan Singkat

1. Jalankan program di terminal hingga jendela kamera terbuka.
2. Arahkan tangan ke kamera dan bentuk isyarat huruf **ASL** secara berurutan (misal: `H` -> `A` -> `L` -> `O`).
3. Sistem akan menyaring duplikasi huruf dan menyimpannya ke dalam memori sementara.
4. Turunkan tangan dari jangkauan kamera (**Hands Down**).
5. Aplikasi akan secara otomatis menggabungkan huruf menjadi `"HALO"` dan membacakannya melalui *speaker* komputer.
6. Tekan tombol **`q`** pada jendela kamera untuk keluar dari aplikasi.

---

## Lisensi

Proyek ini didistribusikan di bawah lisensi **MIT License**. Lihat file `LICENSE` untuk informasi lebih lanjut.
