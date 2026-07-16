# ⚖️ LAW AI — Chatbot Konsultasi Hukum Berbasis NLP

Chatbot konsultasi hukum berbahasa Indonesia yang menggabungkan **rule-based matching** dan **Machine Learning (TF-IDF + Logistic Regression)** untuk menjawab pertanyaan hukum masyarakat awam, termasuk yang menggunakan istilah informal sehari-hari (mis. "nyolong", "nipu").

> Proyek ini dikembangkan sebagai Ujian Akhir Semester (UAS) mata kuliah **Kecerdasan Buatan**.

---

## 📌 Daftar Isi

- [Latar Belakang](#-latar-belakang)
- [Fitur Utama](#-fitur-utama)
- [Arsitektur Sistem](#-arsitektur-sistem)
- [Dataset](#-dataset)
- [Text Preprocessing](#-text-preprocessing)
- [Hasil Evaluasi](#-hasil-evaluasi)
- [Contoh Penggunaan](#-contoh-penggunaan)
- [Keterbatasan & Rencana Pengembangan](#-keterbatasan--rencana-pengembangan)
- [Referensi](#-referensi)
- [Anggota Kelompok](#-anggota-kelompok)

---

## 📖 Latar Belakang

Akses terhadap informasi hukum di Indonesia masih menjadi tantangan bagi masyarakat awam. Bahasa hukum yang formal dan tersebar di berbagai peraturan (KUHP, KUHPerdata, UU ITE, UU Ketenagakerjaan, dll.) membuat masyarakat kesulitan memahami hak, kewajiban, maupun sanksi yang berlaku. Konsultasi langsung dengan praktisi hukum juga membutuhkan biaya dan waktu yang tidak selalu terjangkau.

**LAW AI** hadir untuk menjawab kebutuhan ini secara instan, menggunakan bahasa sehari-hari, sambil tetap merujuk pada dasar hukum yang benar.

**Target pengguna:**
- Masyarakat umum yang butuh edukasi hukum dasar
- Mahasiswa hukum/non-hukum sebagai alat bantu belajar
- Pelaku usaha kecil sebagai referensi awal kewajiban hukum

---

## ✨ Fitur Utama

- 🔎 **Pengenalan Peraturan Hukum** dari kalimat bebas pengguna (rule-based matching)
- 🧠 **Klasifikasi Intent** pertanyaan (Definisi, Sanksi, Hak, Kewajiban, Prosedur, Persyaratan, dll.) menggunakan TF-IDF + Logistic Regression
- 🔁 **Synonym Engine** — menerjemahkan istilah informal ("nyolong") ke istilah hukum baku ("pencurian")
- 🎯 **FAQ Engine** berbasis cosine similarity untuk mencari jawaban paling relevan
- 💾 Model tersimpan dalam format `.pkl` sehingga dapat dimuat ulang tanpa retraining

---

## 🏗️ Arsitektur Sistem

Chatbot dibangun dari lima komponen utama yang saling terhubung dalam satu pipeline:

| Komponen | Metode | Fungsi |
|---|---|---|
| **Legal Recognizer** | Rule-based (regex, longest match) | Mendeteksi nama peraturan hukum yang relevan |
| **Intent Recognizer** | TF-IDF + Logistic Regression | Mengklasifikasikan maksud pertanyaan ke 10 kategori intent |
| **Topic Recognizer** | Keyword matching | Mengenali topik hukum spesifik (mis. "Pencurian", "Wanprestasi") |
| **FAQ Engine** | TF-IDF + Cosine Similarity | Mencari jawaban FAQ paling relevan |
| **AI Legal Chatbot** | Orchestrator | Mengintegrasikan seluruh komponen menjadi satu alur jawaban |

**Alur pemrosesan:**

```
Input Pengguna
   → Deteksi Peraturan (Legal Recognizer)
   → Deteksi Topik (Topic Recognizer)
   → Klasifikasi Intent (Intent Recognizer)
   → Pencarian Jawaban FAQ (FAQ Engine)
   → Jawaban Akhir
```

---

## 📊 Dataset

Knowledge base disusun secara mandiri dalam `Knowledge_Base.xlsx`, terdiri atas 5 sheet:

| Sheet | Jumlah Baris | Kolom | Deskripsi |
|---|:---:|:---:|---|
| **Regulations** | 15 | 7 | Daftar peraturan perundang-undangan |
| **Articles** | 310 | 7 | Daftar pasal beserta topik & kata kunci |
| **Synonyms** | 160 | 3 | Pasangan kata baku & istilah informal |
| **FAQ** | 100 | 6 | Pasangan pertanyaan-jawaban per peraturan & intent |
| **Intent** | 100 | 3 | Contoh kalimat berlabel intent |

**Kategori peraturan** mencakup KUHP, KUHPerdata, UU ITE, UU Perlindungan Konsumen, UU Ketenagakerjaan, UU Perlindungan Data Pribadi, UU Lalu Lintas, UU Perkawinan, UU Hak Cipta, UU TPKS, UU Narkotika, UU Tipikor, UU Perlindungan Anak, UU Kesehatan, dan lainnya.

**Label intent:** `Definisi`, `Sanksi`, `Hak`, `Kewajiban`, `Prosedur`, `Persyaratan`, `Larangan`, `Pelaporan`, `Perizinan`, `Informasi Umum`.

✅ Seluruh sheet telah divalidasi — tidak ditemukan *missing value* maupun data duplikat.

---

## 🧹 Text Preprocessing

1. **Text Cleaning** — lowercase, hapus URL/HTML/angka/tanda baca, rapikan spasi
   `"Apa HUKUMAN bagi orang yang NYOLONG motor???"` → `"apa hukuman bagi orang yang nyolong motor"`
2. **Normalisasi Sinonim** — mengganti istilah informal dengan istilah hukum baku
   `"nyolong motor"` → `"pencurian motor"`
3. **Legal Dictionary & Alias** — 32 entri dictionary + 694 entri alias untuk deteksi peraturan
4. **Masking Istilah Hukum** — istilah hukum diganti token `<HUKUM>` agar model belajar dari pola struktur kalimat
5. **Vektorisasi TF-IDF** — representasi numerik untuk input model klasifikasi

---

## 📈 Hasil Evaluasi

Pengujian **Intent Recognizer** pada 15 data uji mencakup 7 kategori intent:

**Akurasi: 73,33% (11 dari 15 data uji benar)**

| Pertanyaan | Label Aktual | Prediksi | Status |
|---|:---:|:---:|:---:|
| Apa itu tindak pidana? | Definisi | Definisi | ✅ |
| Apa hukuman pencurian? | Sanksi | Sanksi | ✅ |
| Apa itu penipuan? | Definisi | Definisi | ✅ |
| Apakah chat WhatsApp bisa menjadi alat bukti? | Informasi Umum | Persyaratan | ❌ |
| Apa hak konsumen? | Hak | Kewajiban | ❌ |
| Apakah pekerja berhak mendapatkan THR? | Hak | Persyaratan | ❌ |
| Apa syarat sah perkawinan? | Persyaratan | Persyaratan | ✅ |

*Tabel lengkap tersedia pada laporan.*

**Temuan utama:**
- Kalimat tanpa istilah hukum eksplisit (tidak ter-*masking* `<HUKUM>`) lebih rentan salah klasifikasi.
- Pola kalimat serupa antar-intent (`Hak`, `Kewajiban`, `Persyaratan`) menyebabkan kesalahan karena data latih masih terbatas (±10 contoh/kelas).
- **Legal Recognizer** dan **FAQ Engine** tetap berfungsi andal pada seluruh kasus uji.

---

## 💬 Contoh Penggunaan

**Input:** `"Apa hukuman pencurian?"`

```
LEGAL   : Kitab Undang-Undang Hukum Pidana
TOPIK   : Pencurian
INTENT  : Sanksi
JAWABAN : Ancaman pidana pencurian diatur dalam KUHP dan
          bergantung pada jenis serta keadaan tindak pidananya.
```

Skor *similarity* jawaban FAQ pada uji end-to-end mencapai **1.0** (kecocokan sempurna).

---

## 🚧 Keterbatasan & Rencana Pengembangan

**Keterbatasan:**
- Data latih intent masih kecil (100 contoh untuk 10 kelas)
- Model rentan salah klasifikasi pada pola kalimat implisit/baru

**Rencana pengembangan:**
- 📈 Menambah variasi data latih intent (puluhan–ratusan contoh per kelas)
- 🤖 Eksperimen model lanjutan (SVM, IndoBERT)
- 🧩 Penanganan kalimat implisit tanpa bergantung penuh pada token `<HUKUM>`
- 🌐 Deployment ke web/WhatsApp/Telegram Bot agar dapat diakses masyarakat umum

---

## 📚 Referensi

1. Pedregosa, F., et al. (2011). Scikit-learn: Machine learning in Python. *Journal of Machine Learning Research*, 12, 2825-2830.
2. Ramos, J. (2003). Using TF-IDF to determine word relevance in document queries.
3. Salton, G., & Buckley, C. (1988). Term-weighting approaches in automatic text retrieval. *Information Processing & Management*, 24(5), 513-523.
4. Republik Indonesia. *Kitab Undang-Undang Hukum Pidana* (UU No. 1 Tahun 2023).
5. Republik Indonesia. *Undang-Undang Informasi dan Transaksi Elektronik* (UU No. 1 Tahun 2024).
6. Jurafsky, D., & Martin, J. H. (2023). *Speech and Language Processing* (3rd ed. draft). Stanford University.

---

## 👤 Anggota Kelompok

| Nama | NIM |
|---|---|
| Albar Hifdzi Alkariem | 2406092 |

---

<p align="center">Dibuat untuk memenuhi Ujian Akhir Semester mata kuliah Kecerdasan Buatan</p>
