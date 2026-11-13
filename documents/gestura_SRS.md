# Software Requirements Specification (SRS)
## Project:

---

## 1. Pendahuluan

### 1.1 Tujuan Dokumen
Dokumen ini mendefinisikan spesifikasi kebutuhan perangkat lunak untuk proyek akhir semester bernama **Gestura**. Dokumen ini ditujukan sebagai panduan pengembangan teknis bagi pengembang dan acuan penilaian evaluasi akademis.

### 1.2 Cakupan Produk
**Gestura** adalah aplikasi berbasis terminal (CLI) yang berfungsi untuk membaca bahasa isyarat tangan menggunakan kamera (webcam), menerjemahkannya ke dalam bentuk teks, dan mengonversinya menjadi ucapan suara (*Text-to-Speech*). Sistem dikembangkan menggunakan Python dengan memanfaatkan MediaPipe untuk ekstraksi fitur koordinat tangan dan algoritma Machine Learning (MLP & LSTM) untuk klasifikasi objek.

### 1.3 Definisi, Akronim, dan Singkatan
*   **SRS:** *Software Requirements Specification*
*   **CLI:** *Command Line Interface* (Aplikasi Terminal)
*   **MLP:** *Multi-Layer Perceptron* (Neural Network Sederhana)
*   **LSTM:** *Long Short-Term Memory* (Arsitektur RNN untuk data sekuensial)
*   **TTS:** *Text-to-Speech* (Konversi teks ke suara)
*   **Landmarks:** 21 titik koordinat kunci pada tangan yang dideteksi oleh MediaPipe.

---

## 2. Deskripsi Umum

### 2.1 Perspektif Produk
Gestura berjalan sebagai aplikasi mandiri (*standalone*) pada lingkungan terminal. Aplikasi mengakses perangkat keras kamera pengguna secara *real-time*, memproses input video, menjalankan model prediksi, dan mengeluarkan output audio melalui *speaker* sistem komputer.

### 2.2 Fitur Utama Perangkat Lunak
Pengembangan proyek ini dibagi menjadi dua tahapan besar:
1.  **Tahap 1.1 (Fokus Utama):** Pembacaan Alfabet Bisindo/ASL (A-Z) berbasis gambar statis dan sekuensial video, penampungan teks, dan konversi ke ucapan vokal secara *seamless*.
2.  **Tahap 2.2 (Pengembangan):** Pembacaan Kosakata (*Vocabulary*) berbasis video penuh untuk frasa utuh yang berdurasi panjang.

### 2.3 Karakteristik Pengguna
Pengguna aplikasi ini adalah dosen penguji, rekan mahasiswa, atau masyarakat umum yang ingin menerjemahkan gerakan isyarat alfabet/kata tertentu secara langsung lewat komputer tanpa memerlukan antarmuka grafis yang berat (GUI).

### 2.4 Batasan-Batasan Proyek
*   Aplikasi hanya berjalan di lingkungan Command Line Interface (CLI) / Terminal.
*   Aplikasi membutuhkan masukan eksternal berupa Kamera/Webcam yang berfungsi dengan baik.
*   Akurasi model sangat dipengaruhi oleh konsistensi ekstraksi koordinat tangan oleh MediaPipe.

---

## 3. Kebutuhan Fungsional (Functional Requirements)

### 3.1 Tahap 1.1: Pembacaan Alfabet (A-Z)
*   **FR-1.1.1 (Pre-processing):** Sistem harus dapat membaca 30 dataset gambar per kelas untuk alfabet statis (A-I, K-Y) dan dataset video untuk alfabet dinamis (J, Z).
*   **FR-1.1.2 (Feature Extraction):** Sistem harus mengekstrak 21 landmarks tangan $(x, y, z)$ menggunakan MediaPipe dan melakukan normalisasi skala data koordinat.
*   **FR-1.1.3 (Data Alignment):** Sistem harus menyamakan panjang data fitur untuk alfabet statis dan dinamis ke dalam format matriks konstan (misal: duplikasi frame untuk statis agar selaras dengan data gerakan J dan Z).
*   **FR-1.1.4 (Training Model):** Sistem harus mampu melatih data koordinat hasil ekstraksi menggunakan algoritma *Multi-Layer Perceptron* (MLP).
*   **FR-1.1.5 (Real-time Prediction):** Sistem harus dapat mengklasifikasikan gerakan alfabet secara langsung (*live prediction*) dari tangkapan kamera melalui terminal.
*   **FR-1.1.6 (Debouncing & Buffer):** Sistem harus menyaring input huruf yang terdeteksi berulang agar tidak terjadi duplikasi karakter yang sama dalam satu runtutan waktu diam.
*   **FR-1.1.7 (Trigger Hands Down):** Sistem harus mendeteksi kondisi ketika tangan diturunkan atau tidak lagi tertangkap oleh kamera (`Hands Down`).
*   **FR-1.1.8 (Text-to-Speech Seamless):** Saat kondisi `Hands Down` terpenuhi, sistem harus menggabungkan seluruh huruf yang tertampung di dalam array menjadi satu string kata utuh, mengirimkannya ke pustaka TTS, dan mengosongkan kembali isi array penampung.

### 3.2 Tahap 2.2: Pengembangan Vocabulary Reading
*   **FR-2.2.1 (Vocab Pre-processing):** Sistem harus mampu memotong dan menyelaraskan dataset video berformat `.mp4` menjadi sekuens frame konstan.
*   **FR-2.2.2 (Temporal Training):** Sistem harus melatih data runtutan waktu koordinat dari `dataset_vocabulary` menggunakan algoritma *Long Short-Term Memory* (LSTM) atau BiGRU.
*   **FR-2.2.3 (Phrase To Speech):** Sistem harus langsung menerjemahkan hasil klasifikasi sekuens gerakan video menjadi satu kalimat frasa utuh (tanpa mengeja per huruf) dan membacanya via TTS saat tangan diturunkan.

---

## 4. Kebutuhan Non-Fungsional (Non-Functional Requirements)

### 4.1 Performa (Performance)
*   Proses *inference* / prediksi alfabet menggunakan model MLP harus berjalan secara *real-time* dengan waktu jeda (latency) di bawah 100 milidetik per frame.
*   Pemicuan fungsi TTS setelah kondisi `Hands Down` terjadi harus merespons dalam waktu kurang dari 1 detik.

### 4.2 Ranah Lingkungan Perangkat Lunak (Software Environment)
*   **Bahasa Pemrograman:** Python 3.9+
*   **Pustaka Pengolahan Gambar:** OpenCV (`opencv-python`)
*   **Pustaka Ekstraksi Fitur:** MediaPipe (`mediapipe`)
*   **Pustaka Machine Learning:** Scikit-Learn atau TensorFlow/Keras
*   **Pustaka Audio:** `pyttsx3` (Offline TTS) atau `gTTS` (Online TTS)

### 4.3 Portabilitas & Memori
*   Aplikasi harus dapat dijalankan lintas platform (Windows, macOS, Linux) melalui eksekusi terminal atau *command prompt*.
*   Model klasifikasi harus berukuran ringan agar tidak membebani penggunaan RAM komputer saat proses pemindaian kamera sedang aktif.
