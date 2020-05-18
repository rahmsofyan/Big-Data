# Tugas 7 - Implementasi Local big data Irish meter KNIME
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

## Lampiran
[File Model KNIME : Local big data Irish meter](https://hub.knime.com/knime/spaces/Examples/latest/10_Big_Data/02_Spark_Executor/09_Big_Data_Irish_Meter_on_Spark_only)   

## CRISP-DM
### Business Understanding
Listrik menjadi bagian yang tak terpisahkan dari kehidupan masyarakt modern saat ini. Hampir setiap peralatan penunjuang kehidupan manusia ditunjang oleh energi listrik. Tak ayal konsumsi listrik masyarakat semaking bertambah seiring waktu,salah satunya terjadi pada masyarkat Irlandia. Negara Irlandia menguji coba program *Smart Metering Electricity Customer Behaviour Trials*  untuk mengukur kebiasan penggunaan listrik oleh beberapa pelanggan setempat,yang nantinya akan diaplikasikan secara nasional. Dari ujicoba tersebut dilakukan studi yang nantinya dapat digunakan untuk mengetahui dampak dari konsumsi energi oleh masyarakat Irlandia.   
   

Salah satu studi yang perlu dilakukan adalah mengklasterisasi pelanggan berdasar data penggunaan listrik. Klaster-klaster pelanggan tersebut dapat digunakan sebagai acuan pemasaran produk dan strategi pengembangan produk.   
   
Pada kasus ini dilakukan studi dengan tujuan   
- Menentukan klaster dari pelanggan listrik mengggunakan algroritma K-Means dan PCA (Principal Component Analysts)

### Data Understanding
Dataset yang digunakan bersumber dari Irish Energy Meter dataset,yang merupakan kumpulan dari penggunaan listrik oleh beberapa pelanggan dari masyarakat Irlandia.   
Dataset terdiri dari 3 atribut :  
1. **MaterID** menunjukan ID pelanggan
2. **Enc_datetime** menunjukan waktu penggunaan
3. **Reading** menunjukan KW (kilowatt) per 30 menit   
   
Jumlah keseluruhan baris adalah 1226829 baris.   
Dibawah ini merupakan sample dari dataset :   
![sample data](assets/2.JPG)   

### Data Preparation
Pada tahap persiapan data, secara berurutan dilakukan ***loading* data kedalam SPARK**, **ekstraksi pada atribut *enc_datatime***, **Agregasi terhadap hasil ekstraksi**,dan terakhir **mengubah hasil agregasi dapam bentuk persentase**.Hasil akhir dalam bentuk presentasi tersebut yang akan digunakan sebagai data training pada model.
#### 1. Load Data 
Dataset awal yang disimpan dalam bentuk csv dibuka dengan **`Node File Reader`** ,dilanjutkan pembuatan local Spark Contect **`Node Create Local Big Data Environment`** ,lalu data di*-load* kedalam Spark* delam bentuk tabel baru. Proses *loading* data dilakukan di **`Metanode load data`**  yang berisikan **`Node Table Creator`**  untuk menyimpan dataset kedalam tabel,dan **`Node DB Loader `** untuk loading data kedalam Spark Context.   
   
Berikut skema keseluruhan load data :   
<img src="assets/3.1.1.JPG" height="200">   
Berikut skema **metanode Loaddata** :  
<img src="assets/3.1.2.JPG" height="200">   
Proses keseluruhan :   
![prosesload](assets/3.1.3.gif)   
#### 2. Ekstraksi Atribut enc_datetime
Pada Dataset,atribut enc_datatime yang merupakan waktu penggunaan masih dalam berbentuk pengkodean integer sehingga perlu diekstraksi untuk mendapatakan waktu asli yang dapat diagregasi pada tahap selanjutnya.   
   
Pada enc_datetime, 3 digit awal menunjukan jumlah hari diawali dari 1 Januari 2009.   
Misalnya : 00501 berarti waktu penggunaan pada 5 Januari 2009.   
Sedangkan 2 digit akhir menunjukan jumlah 30 menit diawali dari pukul 00:00.   
Misalnya : 00501 berati waktu penggunaan pada pukul 00:30.   

Query dilakukan di dalam spark context, dimana pada konfigurasi KNIME spark context sebelumnya menggunakan kerangka Kerja Hive (*default*) untuk query dan analis datanya. Selanjutnya digunakan Spark SQL sebagai pengganti Hive untuk query dan analisis data dengan **`Node Hive to Spark`** . Spark SQL akan melakukan operasinya kompleksnya dalam memori dan eksekusi volume data dalam bentuk SparkDataFrame/RDD, sehingga performa lebih cepat dan efisien.   

Proses ekstraksi dilakukan dalam 4 tahapan dimana setiap tahapan dilakukan secara berurutan dan hasil ekstraksi pada tahap sebelumnya akan digunakan untuk diekstrak kembali pada tahap selanjutnya.Ekstraksi dilakukan dengan **`Node Spark SQl`.**
   
**Ekstraksi yang dilakukan antara lain :**   
**1.Ekstraksi tanggal dan pukul**   
Ekstraksi pada tahap pertama ini menghasilkan tanggal penggunaan yang disimpan dalam **atribut eventdate** dan pukul akhir penggunaan dalam atribut **my_time**.   
Berikut query sql yang digunakan :   
```
SELECT 

meterid,
enc_datetime,
reading as kw30,
date_add(cast('2008-12-31' as timestamp), cast(substr(enc_datetime, 1, 3) as int)) as eventDate,
concat(
 substr(concat("00", cast(cast((cast(substr(enc_datetime, 4) as int) * 30 / 60) as int) %24 as string)),-2,2),":", 
 substr(concat("00", cast(cast(cast(substr(enc_datetime, 4) as int) * 30 % 60 as int) as string)), -2, 2)
) as my_time

FROM #table# t1
```
   
**2.Ekstraksi jam,hari,minggu,bulan,dan tahun**   
Ekstraksi Kedua ini menghasilkan jam dalam **hour**,hari dalam **dayofweek**,minggu dalam **week** ,bulan dalam **month**,dan tahun dalam **year**. Hasil ekstraksi ini berasal dari ekstraksi yang pertama.
Berikut query sql yang digunakan :   
```
SELECT 

meterid,
kw30,
eventDate,
year(eventDate) as year,
month(eventDate) as month,
weekofyear(eventDate) as week,
date_format(eventDate, 'EEEE') as dayOfWeek,
hour(my_time) as hour

FROM #table# t1
```
   
**3.Ekstraksi kategori jenis hari**   
Ekstraksi ketiga ini menghasilkan kategori jenis hari dalam atibut **dayclassifier** ,BD untuk hari kerja , WE untuk hari non-kerja atau akhir pekan. Hasil ekstraksi ini berasal dari ekstraksi yang kedua.  
Berikut query sql yang digunakan :   
```
SELECT *, 
CASE 
WHEN dayOfWeek in ('Saturday','Sunday') 
      THEN 'WE' 
      ELSE 'BD' 
END as dayClassifier

from #table#
```

    
**4.Ekstraksi segmen 24jam sehari**   
Ekstraksi terakhir ini menghasilkan segmentasi 24 jam sehari dalam atibut **daysegment** . Terdapat 5 segmen untuk 24 jam sehari antari lain "7-9","9-13","13-17","17-21",dan "21-7" . Hasil ekstraksi ini berasal dari ekstraksi yang ketiga.   
Berikut query sql yang digunakan :   
```
SELECT meterID, kw30, eventDate, year, month, week, dayOfWeek, dayClassifier, hour,
CASE 
WHEN hour >=7 AND hour <9 THEN '7-9'
WHEN hour >=9 AND hour <13 THEN '9-13' 
WHEN hour >=13 AND hour <17 THEN '13-17' 
WHEN hour >=17 AND hour <21 THEN '17-21' 
WHEN hour >=21 OR hour <7 THEN '21-7'  
								 
END as daySegment

from #table#
```
   
Dari ekstraksi atribut enc_datetime didapatkan 9 fitur baru antara lain :   
**`kw30,eventDate,year,month,week,dayOfWeek,dayClassifier,hour,daySegment`**   
Berikut skema keseluruhan ekstraksi Atribut enc_datetime :   
<img src="assets/3.2.1.JPG" height="200">   
Berikut skema **metanode Atribut enc_datetime** :  
<img src="assets/3.2.2.JPG" height="200">   
Proses keseluruhan :  
![prosesload](assets/3.2.3.gif)   
Tabel sampel hasil ekstraksi terakhir :
![prosesload](assets/3.2.4.JPG)   
    

#### 3. Agregasi terhadap hasil ekstraksi
Sebelum dilakukan agregasi untuk mempercepat operasi maka data masukan dari Spark SQL diubah menjadi *persistent* RDD dengan level operasi memory only dengan **`Node Persistant Spark Dataframe/RDD`**. Data RDD disimpan dalam JVM ,sedangkan untuk komputasi di dalam memori. Sehingga memungkinkan jika data dimasa depan berasal dari turunan yang sama dari data yang sedang dikomputasi maka komputasi akan lebih cepat. 
   
Hasil Ekstraksi sebelumnya akan dilakukan agregasi kedalam 7 kelompok agregasi,yaitu :   
**1.Total penggunaan atau *total usage*.** 
Pada kelompok ini dilakukan operasi *sum* kwh berdasar **`materID`**.Hasil dari operasi disimpan dalam atribut baru,**`totalKW`**.   
<img src="assets/3.3.1.JPG" height="100">   
   
**2.Rata-rata penggunaan per tahun atau *usage by year***. 
Pada kelompok ini dilakukan operasi *sum* kwh berdasar **`materID`** dan **`year`**.Hasil tersebut dirata-rata berdasar **`materID`** dan disimpan dalam atribut baru **`avgYearlyKW`**.   
<img src="assets/3.3.2.JPG" height="100">   
   
**3.Rata-rata penggunaan per bulan atau *usage by month***.   
Pada kelompok ini dilakukan operasi *sum* kwh berdasar **`materID`** ,**`month`**,dan **`year`**.Hasil tersebut dirata-rata berdasar **`materID`** dan disimpan dalam atribut baru **`avgMonthlyKW`**.   
<img src="assets/3.3.3.JPG" height="100">   
   
**4.Rata-rata penggunaan per minggu atau *usage by week***.   
Pada kelompok ini dilakukan operasi *sum* kwh berdasar **`materID`** ,**`week`**,dan **`year`**.Hasil tersebut dirata-rata berdasar **`materID`** dan disimpan dalam atribut baru **`avgWeeklyKW`**.   
<img src="assets/3.3.4.JPG" height="100">   
   
**5.Rata-rata penggunaan per masing-masing hari atau *usage by days of week***. 
Pada kelompok ini dilakukan operasi *sum* kwh berdasar **`materID`** ,**`week`**,**`daysofweek`**,dan **`year`**.Hasil tersebut dirata-rata berdasar **`materID`** dengan *pivoting* pada **`daysofweek`**  ,dan disimpan dalam atribut baru **`avgMonday`**,**`avgTuesday`**,**`avgWednesday`**,**`avgThursday`**,**`avgFriday`**,**`avgSaturday`**,atau **`avgSunday`**.   
<img src="assets/3.3.5.JPG" height="100">   
   
**6.Rata-rata penggunaan per hari atau *usage by day***.   
Pada kelompok ini dilakukan operasi *sum* kwh berdasar **`materID`** ,**`eventDate`**.Hasil tersebut dirata-rata berdasar **`materID`** dan disimpan dalam atribut baru **`avgWDaily`**.   
<img src="assets/3.3.6.JPG" height="100">   
   
**7.Rata-rata penggunaan per masing-masing segmen jam atau *usage by day segment***.   
Pada kelompok ini dilakukan operasi *sum* kwh berdasar **`materID`** ,**`evenDate`**,dan **`daySegment`**.Hasil tersebut dirata-rata berdasar **`materID`** dengan *pivoting* pada **`daySegment`**  ,dan disimpan dalam atribut baru **`avg_7to9`**,**`avg_9to13`**,**`avg_13to17`**,**`avg_17to21`**,atau **`avg_21to7`**.   
<img src="assets/3.3.7.JPG" height="100">   
   
**8.Rata-rata penggunaan per masing-masing jenis hari *usage by day classifier***.   
Pada kelompok ini dilakukan operasi *sum* kwh berdasar **`materID`** ,**`year`**,**`month`**,**`week`**,dan **`dayclassifier`**.Hasil tersebut dirata-rata berdasar **`materID`** dengan *pivoting* pada **`dayClassifier`**  ,dan disimpan dalam atribut baru **`avg_BD`**,atau **`avg_WE`**.
<img src="assets/3.3.8.JPG" height="100">   
   
**9.Rata-rata penggunaan per jam atau *usage by hour*.**   
Pada kelompok ini dilakukan operasi *sum* kwh berdasar **`materID`** ,**`eventDate`**,dan **`hour`**.Hasil tersebut dirata-rata berdasar **`materID`** dan disimpan dalam atribut baru **`avgHourly`**.   
<img src="assets/3.3.9.JPG" height="100">   

Pada agregasi di atas operasi *sum* dan rata-rata berdasar atribut tertentu di atas dengan **`node Spark GroupBy`**,sedangkan untuk 
rata-rata pivoting dengan **`node Spark Pivot`**.
Hasil untuk masing-masing agregasi di atas dilakukan join dengan **`node Spark Joiner`**. Dari hasil join tersebut didapatkan 20 atribut fitur dari dataset antara lain :   
**`totalKW,avgYearlyKW,avgMonthlyKW,avgWeeklyKW,avgMonday,avgTuesday,avgWednesday,avgThursday,avgFriday,avgSaturday,avgSunday,avgDaily,avg_7to9,avg_9to13,avg_13to17,avg_17to21,avg_21to7,avg_BD,avg_WE,avgHourly`**.

Berikut skema **metanode Aggregations and time series** :  
<img src="assets/3.3.10.JPG" height="200">   
Salah satu proses dari Agregasi :usage by days of week : 
![prosesload](assets/3.2.11.gif)   
Tabel sampel hasil agreagasi terakhir :
![prosesload](assets/3.2.12.JPG)   
     
### Modeling
### Evaluation  
### Deployment   
