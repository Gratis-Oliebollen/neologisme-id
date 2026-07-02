# Sistem Pencetus Kandidat Neologisme Istilah Teknis (EN→ID)

Proyek riset Natural Language Processing yang dikembangkan selama program
Summer Course di Saga University, Jepang, dan dipakai sebagai bahan
konversi mata kuliah **CIF62340 — Pemerolehan Informasi dan Penambangan
Teks** (Program Studi Teknik Informatika, FILKOM Universitas Brawijaya).

## Deskripsi proyek

Sistem ini menerima sebuah istilah teknis berbahasa Inggris (misalnya
`"string"`) beserta definisinya, lalu menghasilkan **daftar kandidat kata
bahasa Indonesia** yang secara semantik berkaitan dengan istilah tersebut.
Kandidat ini dimaksudkan sebagai **bahan pertimbangan** bagi perencana
istilah (terminology planner) atau pengguna lain dalam memilih atau
merumuskan padanan/neologisme yang tepat — bukan sebagai jawaban tunggal
yang otomatis benar.

Pipeline menggabungkan:
- **FastText** (`cc.id.300.vec`, korpus Common Crawl bahasa Indonesia)
  untuk pencarian kandidat berbasis kemiripan vektor kata.
- **Sentence-Transformer multilingual**
  (`paraphrase-multilingual-MiniLM-L12-v2`) untuk menilai kemiripan makna
  antara kandidat dan definisi istilah, termasuk lintas bahasa
  Inggris–Indonesia.

## Struktur pipeline

1. Input istilah teknis Inggris + definisi.
2. Ekstraksi kata kunci dari definisi (stopword removal manual).
3. Terjemahan kata kunci ke Indonesia (kamus manual, `EN_ID_DICT`).
4. Pencarian kandidat kata Indonesia via FastText (`most_similar`).
5. Penilaian (scoring) setiap kandidat terhadap definisi penuh
   menggunakan Sentence-Transformer.
6. Evaluasi kuantitatif:
   - **Recall@K (exact match)** — apakah istilah resmi (gold standard)
     muncul tepat di antara top-K kandidat.
   - **Semantic Relevance Score** — kedekatan makna kandidat terhadap
     istilah resmi, dengan baseline kontrol sebagai pembanding.

Penjelasan lengkap setiap tahap, termasuk pendekatan yang sempat dicoba
dan ditinggalkan (lihat bagian Lampiran di notebook), didokumentasikan
langsung di dalam `notebooks/Neologisme_v3.ipynb`.

## Cara menjalankan

Notebook ini dirancang untuk **Google Colab** (memanfaatkan `wget` untuk
mengunduh model FastText dan dukungan GPU opsional). Langkah:

1. Buka `notebooks/Neologisme_v3.ipynb` di Google Colab.
2. Jalankan seluruh sel secara berurutan dari atas
   (`Runtime → Run all`, atau `Restart session and run all` untuk
   memastikan tidak ada state tersisa dari sesi sebelumnya).
3. Tidak ada berkas eksternal yang perlu diunggah manual — model
   FastText diunduh otomatis di sel awal (±1.1 GB, memerlukan koneksi
   internet yang stabil).

Estimasi waktu jalan: sekitar 5–10 menit di CPU Colab gratis (lebih cepat
jika GPU T4 diaktifkan melalui `Runtime → Change runtime type`).

## Dependensi

Lihat [`requirements.txt`](./requirements.txt). Library utama:
`gensim`, `torch`, `transformers`, `sentence-transformers`.

## Data acuan

- **Definisi istilah** (5 istilah uji: array, browser, compiler, mouse,
  string) ditulis manual untuk keperluan riset ini, bukan disalin dari
  kamus berhak cipta.
- **Gold standard** (padanan resmi) bersumber dari KBBI Daring dan/atau
  situs PASTI (Padanan Istilah, Badan Bahasa Kemdikbud).

## Hasil evaluasi (ringkasan)

Pada pengujian 5 istilah:

- **Recall@5 (exact match): 0%** — sistem belum berhasil menempatkan
  istilah resmi tepat di top-5 kandidat untuk kelima istilah uji.
- **Semantic Relevance Score rata-rata: 0,82**, dibandingkan baseline
  kata tak-berhubungan yang berkisar **0,28–0,40** — menunjukkan kandidat
  yang dihasilkan signifikan lebih dekat secara makna terhadap istilah
  resmi dibanding kata acak, meski belum mereproduksi istilah resmi
  secara tepat.

Penjelasan lebih lanjut mengenai temuan ini, termasuk analisis error dan
keterbatasan metodologis yang ditemukan selama pengembangan (anisotropy
pada IndoBERT base, bias leksikal pada skema scoring awal, serta
sirkularitas evaluasi yang sempat terjadi dan telah diperbaiki), tersedia
di bagian Lampiran pada notebook.

## Keterbatasan & pengembangan lanjutan

- Kamus terjemahan kata kunci (`EN_ID_DICT`) disusun manual dengan
  cakupan terbatas pada 5 istilah uji; belum scalable untuk istilah
  dalam jumlah besar.
- Daftar stopword Inggris disusun manual dan ringkas, belum
  selengkap daftar standar seperti NLTK.
- Filter pola linguistik (afiksasi, kalke) yang direncanakan di awal
  riset belum diimplementasikan pada versi ini.
- Model FastText mewarisi noise ejaan dari korpus web mentah
  (misalnya kesalahan ketik yang konsisten muncul sebagai kandidat).

## Lisensi

Lihat [`LICENSE`](./LICENSE).

## Kontak

Dustin — mahasiswa Teknik Informatika, Universitas Brawijaya
(program exchange Saga University).
