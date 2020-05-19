# EAS - Analisis Penjualan shampoo
Nama Mahasiswa : Rahma Sofyantoro | NRP : 05111640000117  

## Daftar Isi  
- [Lampiran](#lampiran)   
- [CRISP-DM](#crisp-dm)   
      [Business Understanding](#business-understanding)   
      [Data Understanding](#data-understanding)   
      [Data Preparation](#data-preparation)   
      [Modeling](#modeling)   
      [Evaluation](#evaluation)   
      [Deployment](#deployment)  

## dataset
[sales-of-shampoo-over-a-three-ye.csv](dataset/sales-of-shampoo-over-a-three-ye.csv)   

## Lampiran
[Analisis_3.knwf](lampiran/Analisis_3.knwf)   


## CRISP-DM
### Business Understanding
 Penjualan produk shampoo yang tidak stabil menjadikan produsen shampoo perlu melakukan perencanaan yang baik dalam menjalankan bisnisnya. Berangkat dari hal tersebut dilakukan studi untuk memprediksi penjualan shampoo selanjutnya. Hasil prediksi ini diharapkan dapat digunakan sebagai acuan kebijakan pemasaran atau penentuan harga.  
   
Pada kasus ini dilakukan studi dengan tujuan   
- Memprediksi penjualan samphoo bulanan

### Data Understanding
Dataset yang digunakan bersumber dari [sales-of-shampoo-over-a-three-ye.csv](dataset/sales-of-shampoo-over-a-three-ye.csv)   
, kumpulan penjualan shampoo selama 3 tahun.
Dataset terdiri dari 3 atribut :  
1. **Month** menunjukan bulan terjadinya penjualan
2. **Sales of shampoo over a three year period** jumlah penjualan shampoo 
   
Jumlah keseluruhan 38 baris.   
Dibawah ini merupakan sample dari dataset :   
<img src="assets/2.1.JPG" height="200">   

### Data Preparation
Pada tahap persiapan data, secara berurutan ***loading* data kedalam Spark**, **Pengubahan field Month ke bentuk standar Date SQL**,dan ***Sorting* atribut tanggal**. Data hasil persiapan ini akan menjadi data training dari model.   
#### 1. Loading Data kedalam Spark
Dataset awal yang disimpan dalam bentuk csv dibuka dengan **`Node File Reader`** ,dilanjutkan pembuatan local Spark Contect dengan **`Node Create Local Big Data Environment`** ,lalu data di-*load* kedalam Spark delam bentuk tabel baru. **`Node Table Creator`**  untuk menyimpan dataset kedalam tabel,dan **`Node DB Loader `** untuk loading data kedalam Spark Context.   
   
Berikut skema keseluruhan load data :   
<img src="assets/3.1.1.JPG" height="200">   

Proses keseluruhan :   
![prosesload](assets/3.1.2.gif)   
   
#### 2. Pengubahan field DATE ke bentuk standar SQL
Sebelum pengubahan field,masukan yang masih berupa Hive diubah kedalam Spark SQL. Spark SQL sebagai pengganti Hive untuk query dan analisis dat dengan **`Node Hive to Spark`** . Spark SQL akan melakukan operasi kompleks dalam memori dan mengeksekusi volume data dalam bentuk SparkDataFrame/RDD, sehingga performa lebih cepat dan efisien.  

Field pada **Month** masih berformat "tahun ke-bulan",sedangkan untuk standar SQL menggunakan yyyy-mm-dd,sehingga data perlu diubah
menjadi format standar. 
   
Pengubahan dilakukan dengan query menggunakan **`node Spark SQL`**.
Mengambil nila-nilai bulan,dan tahun ke dengan regex lalu disimpan dalam atribut **month**, dan **year** . Setalah itu dilakukan query kembali ,untuk menggabungkan tahun,dan bulan yang sudah dalam bentuk angka ke standar SQL. Angka tahun yang terlalu kecil bernilai *invalid* ,untuk itu perlu dilakukan panambahan tahun.Selain itu juga diberikan default tanggal 1 untuk masing-masing data sebagai *default*.Data hasil *concating* yang masih berbentuk string dikonversi tipe datanya menjadi Datetime dengan fungsi date.  Atribut **newDate** sebagai pengganti atribut lama, **Month** dengan field-field yang sesuai standar SQL.  
Berikut query sql yang digunakan :   
```
SELECT 	
			regexp_extract(`Month`,'([0-9]+)\-([a-zA-Z]+)',1) as year,
 			regexp_extract(`Month`,'([0-9]+)\-([a-zA-Z]+)',2) as month,
 			`Sales of shampoo over a three year period`
 			
FROM #table#
```
   
Query selanjuntya :   
Berikut query sql yang digunakan :   
```
SELECT 	date(concat('199',`year`,"-",
CASE
	WHEN `month` = 'Jan' THEN '1'
	WHEN `month` ='Feb' THEN '2'
	WHEN `month` ='Mar' THEN '3'
	WHEN `month` ='Apr' THEN '4'
	WHEN `month` ='May' THEN '5'
	WHEN `month` ='Jun' THEN '6'
	WHEN `month` ='Jul' THEN '7'
	WHEN `month` ='Aug' THEN '8'
	WHEN `month` ='Sep' THEN '9'
	WHEN `month` ='Oct' THEN '10'
	WHEN `month` ='Nov' THEN '11'
	WHEN `month` ='Dec' THEN '12'
END,"-1")) AS newDate,`Sales of shampoo over a three year period`
FROM #table#
```
Berikut skema Pengubahan field DATE ke bentuk standar SQL :   
<img src="assets/3.2.1.JPG" height="200">   

Proses keseluruhan :   
![prosesload](assets/3.2.2.gif)   

#### 3. Sorting
Data masukan selanjutnya akan disorting sesuai urutan atribut **newDate** secara *ascending menggunakan **`Node Spark Sorter`**.
Setelah itu data akan diubah dari bentuk Spark ke tabel KNIME untuk dilakukan *lagging* di tahap modeling. Pengubahan Spark ke tabel Knime menggunakan **`node Spark to Table`**.

Berikut skema sorting :   
<img src="assets/3.2.1.JPG" height="200">   

Proses keseluruhan :   
![prosesload](assets/3.3.2.gif)   

### Modeling
Data masukan dipartisi menjadi data training dan data testing dengan rasio 80%:20%. Pengambilan data dilakukan secara berurutan atau *Take from Top* agar hasil partisi data tetap berurutan. Urutan data ini sangat penting untuk prediksi selanjutnya.   
   
Prediksi data berbasis *time series*  memerlukan beberapa data secara berurut untuk setiap prediksinya atau labelnya. Sedangkan Data masukan hanya memiliki 2 kolom yaitu **newDate** dan **Sales of shampoo over a three year period**,yang berarti hanya memiliki satu urutan data. Untuk menambah data seri digunakan **`Node Lag Column`**. Masing-masing baris dari data masukan akan dikorelasikan dengan baris dibawahnya secara berurut untuk membentuk satu data seri dengan membuat salinan kolom data input yang dipilih dan menggeser sel ke bawah ke sejumlah langkah tertentu.
   
Berikut paramet **`Node Lag Column`** :   
<img src="assets/4.1.JPG" height="200">   

Lag bernilai 6 menunjukan akan ada 6 baris salinan  dari atribut **Sales of shampoo over a three year period**, sedangkan Lag interval bernilai 1 menunjukan jarak data bergeser kebawah sebanyak 1. Hal ini juga dapat diartikan bahwa prediksi produksi listrik pada bulan tertentu berdasar 6 bulan sebelumnya.

Data hasil **lagging** akan dilatih dalam algoritma Linear Regression menggunakan node **`node Linear Aggression learner`** dengan label data adalah atribut **Sales of shampoo over a three year period**.

Berikut paramater dari *Linear Regression learner* :   
<img src="assets/4.2.JPG" height="200">   

Proses keseluruhan :   
![prosesload](assets/4.3.gif)   

### Evaluation  
Pada tahap ini dilakukan evaluasi terhadap model yang sudah dibuat sebelumnya dengan data testing.

Berikut Skema Evaluasi Model :   
<img src="assets/5.1.JPG" height="200">   
   
Hasil evaluasi sebagai berikut :   
<img src="assets/5.2.JPG" height="100">   

Hasil dari error matrics di atas menunjukan bahma Mean absolute error yaitu 75.572 dan Root mean squared error 77.582, simpangan  cukup jauh dari nilai yang asli. Model tidak berhasil untuk melakukan prediksi yang cukup akurat, hal ini mungkin dikarekanan jumlah dataset yang kecil.

### Deployment   
Pada tahap deployement model prediksi akan disimpan dalam bentuk PMML menggunakan **`node PMML Writter`**.
Sedangkan hasil prediksi yang telah dikurangi kolumnya dengan **`node Colum filter`** sehingga tinggal menyisakan  akan disimpan dalam bentuk CSV.
![prosesload](assets/6.gif)   