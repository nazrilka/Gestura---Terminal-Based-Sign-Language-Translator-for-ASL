# Software Requirements Specification (SRS)
## Modul: Image & Video Pre-Processing Pipeline (Project: Gestura)
**Target Bahasa Isyarat:** American Sign Language (ASL)  
**Konteks Akademis:** Mata Kuliah Pengolahan Citra Digital

---

## 1. Pendahuluan

### 1.1 Tujuan Dokumen
Dokumen ini berisi spesifikasi teknis khusus untuk **Modul Pre-Processing** pada proyek **Gestura**. Dokumen ini dibuat sebagai panduan kerja (*handover*) bagi pengembang yang bertanggung jawab mengolah dataset mentah (gambar dan video ASL) menjadi dataset numerik yang siap digunakan untuk tahap pelatihan (*training*) model *Multi-Layer Perceptron* (MLP).

### 1.2 Ruang Lingkup Modul
Modul ini bertugas mengambil data mentah dari direktori `dataset_alfabet` (A-Z), menerapkan teknik pengolahan citra digital dan ekstraksi fitur menggunakan **MediaPipe**, melakukan normalisasi geometris, menyelaraskan dimensi data statis dan dinamis, serta mengekspor hasilnya ke dalam satu file tabular berformat `.csv` atau `.npy`.

---

## 2. Spesifikasi Input (Data Mentah)

Sistem harus mampu membaca struktur direktori input dengan spesifikasi berikut:
*   **Lokasi Direktori Utama:** `dataset_alfabet/`
*   **Kelas Statis (24 Kelas: `A-I`, `K-Y`):**
    *   Format file: `.jpg`, `.jpeg`, atau `.png`.
    *   Jumlah sampel: Minimal 30 gambar per folder kelas alfabet.
*   **Kelas Dinamis (2 Kelas: `J`, `Z`):**
    *   Format file: `.mp4` atau `.avi`.
    *   Jumlah sampel: Minimal 30 video gerakan per folder kelas alfabet.

---

## 3. Kebutuhan Fungsional & Alur Pemrosesan (Pipeline)

Pengembang wajib mengimplementasikan urutan pemrosesan berikut secara berurutan:

### 3.1 Pre-Processing Spasial (Level Piksel)
*   **REQ-PP-01 (Pembacaan & Penyeragaman Ukuran):** Sistem membaca setiap citra/frame video menggunakan OpenCV dan mengubah resolusinya (*resizing*) menjadi dimensi standar `640x480` piksel agar beban deteksi seragam.
*   **REQ-PP-02 (Konversi Ruang Warna):** Sistem wajib mengonversi matriks warna citra dari ruang warna bawaan OpenCV (**BGR**) ke ruang warna **RGB** sebelum dimasukkan ke mesin deteksi MediaPipe.
*   **REQ-PP-03 (Peningkatan Kontras - Opsional/Direkomendasikan):** Jika citra memiliki pencahayaan redup, sistem menerapkan metode *CLAHE (Contrast Limited Adaptive Histogram Equalization)* pada kanal luminansi untuk memperjelas tepi kontur jari.

### 3.2 Ekstraksi Fitur (MediaPipe Hand Landmarks)
*   **REQ-PP-04 (Deteksi Titik Kunci):** Sistem menjalankan `mediapipe.solutions.hands` dengan parameter `static_image_mode=True` (untuk gambar) dan `False` (untuk video), serta `max_num_hands=1`.
*   **REQ-PP-05 (Ekstraksi Koordinat 3D):** Untuk setiap tangan yang terdeteksi, sistem mengambil **21 titik landmarks** yang masing-masing memiliki nilai koordinat `(x, y, z)`, menghasilkan **63 nilai fitur mentah** per frame.
*   **REQ-PP-06 (Penanganan Gagal Deteksi):** Jika MediaPipe gagal mendeteksi tangan pada suatu gambar atau frame tertentu (`results.multi_hand_landmarks is None`), data tersebut harus dilewati (*skip*) dan dicatat dalam *log* terminal agar tidak menghasilkan nilai kosong (`NaN`).

### 3.3 Pre-Processing Geometris (Normalisasi Koordinat)
Agar model tidak terpengaruh oleh posisi tangan di layar (*translation invariant*) maupun jarak tangan ke kamera (*scale invariant*), wajib dilakukan operasi matematika berikut pada 21 titik landmarks:
*   **REQ-PP-07 (Translasi Titik Pusat / Wrist Centering):** Menjadikan titik pergelangan tangan (Landmark index `0`) sebagai titik pusat `(0, 0, 0)`. Seluruh 20 titik lainnya dikurangi dengan koordinat titik ke-0:
    $$x'_i = x_i - x_0, \quad y'_i = y_i - y_0, \quad z'_i = z_i - z_0$$
*   **REQ-PP-08 (Normalisasi Skala):** Membagi seluruh nilai koordinat relatif di atas dengan nilai absolut maksimum dari titik terjauh terhadap pergelangan tangan. Hasil akhir koordinat wajib berada pada rentang **`-1.0` hingga `1.0`**.

### 3.4 Penyelarasan Dimensi Waktu (Temporal Alignment)
Karena kelas `J` dan `Z` berupa video gerakan sementara kelas lainnya berupa gambar diam, dimensi vektor fitur keduanya harus disamakan:
*   **REQ-PP-09 (Sampling Frame Dinamis - Kelas J & Z):** Sistem mengekstrak tepat **15 frame** yang terdistribusi secara merata (*uniform sampling*) dari awal hingga akhir setiap video gerakan `J` dan `Z`. Total fitur per video = $15 \text{ frame} \times 63 \text{ koordinat} = \mathbf{945 \text{ fitur}}$.
*   **REQ-PP-10 (Duplikasi Frame Statis - Kelas A-I, K-Y):** Untuk setiap gambar statis yang telah diekstrak menjadi 63 fitur ternormalisasi, sistem menduplikasi array 63 fitur tersebut sebanyak **15 kali** secara berurutan agar menghasilkan panjang vektor yang identik, yaitu **945 fitur**.

---

## 4. Spesifikasi Output (Deliverables ke Tahap Training)

Tahap *pre-processing* dinyatakan selesai apabila berhasil menghasilkan keluaran dengan kriteria berikut:

1.  **Format File:** Satu file dataset bersih bernama `dataset_asl_preprocessed.csv` (atau `.npy`).
2.  **Struktur Kolom & Baris:**
    *   **Jumlah Baris:** Total sampel valid (Target: $\approx 780$ baris dari $26 \text{ kelas} \times 30 \text{ sampel}$).
    *   **Jumlah Kolom:** Total **946 kolom**, yang terdiri dari:
        *   Kolom `0` s/d `944`: Nilai numerik *float* koordinat `(x, y, z)` yang sudah dinormalisasi (`-1.0` s/d `1.0`).
        *   Kolom `945` (atau kolom `label`): String karakter label kelas alfabet (`'A'` sampai `'Z'`).
3.  **Validasi Kebersihan Data:** Tidak boleh ada nilai `Null`, `NaN`, atau `Infinite` di dalam file output.
4.  **Fungsi Reusable (Penting untuk Live Testing):** Fungsi normalisasi geometris (`REQ-PP-07` & `REQ-PP-08`) wajib dibuat dalam bentuk fungsi Python terpisah (misal di dalam file `preprocessing_utils.py`) agar dapat dipanggil ulang secara *real-time* saat aplikasi terminal membaca kamera pengguna.

---

## 5. Lingkungan Pengembangan (Tech Stack)
*   **Bahasa:** Python 3.9+
*   **Library Wajib:** `opencv-python`, `mediapipe`, `numpy`, `pandas`
