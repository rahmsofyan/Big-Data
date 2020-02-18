# Tugas 1 - ETL menggunakan KNIME
Nama Mahasiswa : Rahma Sofyantoro | NRP : 05111640000117

## Daftar Isi
- [Dataset](#Dataset)
- [1. Business Understanding](#1-business-understanding)
- [2. Data Understanding](#2-data-understanding)
- [3. Data Preparation](#3-data-preparation)
- [4. Modeling](#4-modeling)
- [5. Evaluation](#5-evaluation)
- [6. Deployment](#6-Deployment)

## Dataset
[The Human Freedom Index (HFI)](https://www.cato.org/human-freedom-index-new) menyajikan tingkat kebebasan manusia secara umum , dipahami sebagai tidak adanya kendala koersif,dengan skala 0 hingga 10 (lebih menujukan baik).
Didalamnnya menggunakan 79 indikator berbeda dari kebebasan personal dan ekonomi di bidang-bidang berikut:

- Aturan hukum
- Keamanan dan Keselamatan
- Gerakan
- Agama
- Asosiasi, Majelis, dan Sosial Sipil
- Ekspresi dan Informasi
- Identitas dan Relasi
- Ukuran Pemerintahan
- Sistem Hukum dan Hak Milik
- Akses ke komiditas yang memiliki nilai stabil
- Kebebasan untuk Berdagang Secara Internasional
- Peraturan Kredit, Perburuhan, dan Bisnis

HFI mencakup 162 negara hingga 2017.  
Di dalam dateset terdapat 120 kolom,dengan 1620 record data 
[Download Dataset]()

## 1. Business Understanding
Dari dataset tersebut dapat dilakukan analisis mengenai :
1. Tingkat persebaran kebebasan di seluruh dunia.
1. Negara - negara dengan tingkat kebebasan tertinggi dan terendah untuk masing-masing indikator
1. Korelasi positif dan negatif antar indikator
1. Negara-negara yang mengalami perubahan signifikan masing-masing indikator dari tahun ke tahun
## 2. Data Understanding
Dari dataset HFI diketahui :
- Dataset terdiri 1620 baris dan 120 kolom.  
- Terdapat 76 Indikator/Kolom (33 dari *personal freedom*,43 dari *economic freedom*) berbeda yang ditunjukan pada Gambar 1).  
- Terdapat 4 kolom indentitas lokasi dan waktu yang berkaitan (*year*,ISO_CODE,*countries*,*region*).  
- Setiap kategori indikator memiliki nilai rata-rata dari indikator yang termasuk bagiannya.
- Setiap subkategori indikator memiliki nilai rata-rata dari indikator yang termasuk bagiannya.
- Kategori *Personal Freedom* menganai peraturan hukum dan keamanan terhadap individu, dan kebebasan individu untuk berserikat.
- Pada Kategori *Personal Freedom* terdapat sub-kategori antara lain *Legal Protection and Security* mengenai Peraturan Hukum ,dan Sekuritas Keamanan, *Spesific Personal Freedoms* yang mengenai hubungan individu dengan komunitas atau kelompok-kelompok di dalamnya.
- Kategori *Economic Freedom* mengenai bantuan pemerintah, hak dan legalitas kepemilikan individu, komoditas ,perdaganan internasional, dan regulasi yang mengatur hal-hal yang berkaitan dagan bisnis,ketenagakerjaan,dan kredit.
## 3. Data Preparation
## 4. Modeling
## 5. Evaluation
## 6. Deployment
