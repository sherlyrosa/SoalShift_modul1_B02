

# Laporan Praktikum SoalShift_modul1_B02

 1. Anda diminta tolong oleh teman anda untuk mengembalikan filenya yang telah dienkripsi oleh seseorang menggunakan bash script, file yang dimaksud adalah nature.zip. Karena terlalu mudah kalian memberikan syarat akan membuka seluruh file tersebut jika pukul 14:14 pada tanggal 14 Februari atau hari tersebut adalah hari jumat pada bulan Februari.  
	`Hint: Base64, Hexdump`

	**Jawab :**
	```sh
	for foto in *
		do
		    base64 -d "$foto" | xxd -r > "$foto-tmp"
		    rm "$foto"
		    mv "$foto-tmp" "$foto"
		done
	```
	> See : [full code](../blob/master/LICENSE)
	
	**Penjelasan :**
	 1. Lakukan ekstrak file nature.zip dan buka folder hasil ekstraksi.
	 2. Lakukan *decode* file-file yang ter-*encode* menggunakan `base64`.
		  ```console
		  $ base64 -d [nama file] > [output file]
		  ```
		  Akhirnya kita berhasil men-*decode* file tersebut, namun file tersebut tertulis dalam bentuk hexa, sehingga tidak dapat terbuka sebagai gambar seperti semestinya.
	3. Maka kita perlu merubah file hexa tersebut ke dalam binary file, kita dapat menggunakan *command* `xxd`.
		```sh
		$ xxd -r [input file] > [output file]
		``` 	
		*Yeyhh!!* Akhirnya kita mendapatkan foto yang seharusnya.
	4. Namun karena file tersebut terlalu banyak dan tidak memungkinkan untuk melakukan cara tersebut satu persatu. Maka kita harus melakukan cara tersebut dengan cara *looping* dalam file bash.
		```sh
		#!/bin/bash
		for foto in *
			do
				base64 -d "$foto" | xxd -r > "$foto-tmp"
			    rm "$foto"
			    mv "$foto-tmp" "$foto"
			done
		```
		Dengan cara tersebut, setelah melakukan ekstraksi, kita menjalankan file bash tersebut di dalam folder nature. Maka tiap foto di-*decode* lalu di-rename menjadi `[nama foto]-tmp.jpg` lalu foto yang ter-*encode* dihapus menggunakan *command* `rm` dan diganti oleh foto yang sudah kita *decode* menggunakan *command* `mv`.
	5. Setelah file bash selesai, kita dapat menjalankannya menggunakan `cron`. Buka `crontab` :
		```sh
		$ crontab -e
		```
	6. Buat *command* `cron` di `crontab` pada saat pukul 14:14 pada tanggal 14 Februari.
		```sh
		14 14 * 2 5 ~/Desktop/SoalShift_modul1_B02/soal1.sh
		```
	7. Selesai~ pastikan ketika menjalankan file bash tersebut, file tersebut berdampingan dengan file nature.zip
	> See : [full code](../blob/master/LICENSE)
		
		
2. Anda merupakan pegawai magang pada sebuah perusahaan retail, dan anda diminta
untuk memberikan laporan berdasarkan file `WA_Sales_Products_2012-14.csv`.
Laporan yang diminta berupa:  
	a. Tentukan negara dengan penjualan(*quantity*) terbanyak pada tahun 2012.  
    b. Tentukan tiga product line yang memberikan penjualan(*quantity*) terbanyak pada soal poin a.  
	c. Tentukan tiga *product* yang memberikan penjualan(*quantity*) terbanyak berdasarkan tiga product line yang didapatkan pada soal poin b.
    
	**Jawab :**  
	a. Negara dengan penjualan(*quantity*) terbanyak pada tahun 2012 :
	```sh
	$ awk -F "," '{if($7 == "2012") a[$1]+=$10}END{for(v in a)print a[v]" - "v}' WA_Sales_Products_2012-14.csv | sort -nr | head -1 | awk -F "-" '{print $2}'	
	```
	b. Tiga *product line* yang memberikan penjualan(*quantity*) terbanyak pada soal poin a :
	```sh
	$ awk -F "," '{if($7=="2012" && $1=="United States") a[$4]+=$10} END {for(v in a)print v}' WA_Sales_Products_2012-14.csv | sort -nr | head -3
	```
	c. Tiga produk yang memberikan penjualan(*quantity*) terbanyak berdasarkan tiga *product line* yang didapatkan pada soal poin b :
	```sh
	$ awk -F "," '{if ($7=="2012" && $1=="United States") a[$4]+=$10} END {for(v in a)print a[v]" - "v}' WA_Sales_Products_2012-14.csv | sort -nr | head -3 | awk -F "-" '{print $2}'	
	```
	
	**Penjelasan :**  
	a. Negara dengan penjualan(*quantity*) terbanyak pada tahun 2012 :
	1. Kita akan menggunakan `awk` untuk melakukan *searching*. Gunakan `option` `-F` karena file yang akan kita searching menggunakan *delimiter* berupa `,` sedangkan `awk` *default* menggunakan  ` ` *(space)*.
	2. Pertama-tama `print` semua daftar pada tahun 2012. Dalam file ini tahun berada pada kolom ke-7. 
		```sh
		$ awk -F "," '{if($7 == "2012") print}' WA_Sales_Products_2012-14.csv
		```
		Maka akan keluar seluruh penjualan pada tahun 2012.
	3. Setelah itu, hitung jumlah penjualan pada tiap negara dengan memasukkannya ke dalam *array*, `a[$1]+=$10`. Dalam kasus ini, penjualan(*quantity*) berada di kolom 10 dan nama negara berada pada kolom 1.
	4. Lalu, lakukan iterasi(*looping*) untuk mencetak(print) hasilnya. 
		```sh
		$ awk -F "," '{if($7 == "2012") a[$1]+=$10}END{for(v in a)print a[v]" - "v}' WA_Sales_Products_2012-14.csv
		```
		Maka akan keluar, seluruh jumlah penjualan(*quantity*),`a[v]`, beserta nama negaranya,`v`.
	5. Lakukan *sorting* secara *descending*(dari nilai terbesar ke terkecil) dengan command `sort -nr`
		```sh
		$ awk -F "," '{if($7 == "2012") a[$1]+=$10}END{for(v in a)print a[v]" - "v}' WA_Sales_Products_2012-14.csv | sort -nr	
		```
	6. Terakhir ambil satu nilai tertinggi saja menggunakan command `head -1`. dan cetak(*print*) nama negaranya saja menggunakan `awk`.
		```sh
		$ awk -F "," '{if($7 == "2012") a[$1]+=$10}END{for(v in a)print a[v]" - "v}' WA_Sales_Products_2012-14.csv | sort -nr | head -1 | awk -F "-" '{print $2}'	
		```
		Maka hasil yang ditemukan ialah :
		```
		 United States
		```
	b. Tiga  _product line_  yang memberikan penjualan(_quantity_) terbanyak pada soal poin a :
	1. Masih menggunakan syntax `awk` seperti poin a. Namun dalam kasus ini, kita hanya memperlukan mencetak(*print* negara yang bersangkutan saja sesuai hasil poin a, yaitu `United States`, maka kita akan mengubah kondisi menjadi `if($7=="2012" && $1=="United States")`.
	2. Sekarang kita perlu menghitung penjualan *produk line* pada negara tersebut, menggunakan cara yang sama namun dalam kasus ini kolom *produk line* berada pada kolom 4.
		```sh
		$ awk -F "," '{if ($7=="2012" && $1=="United States") a[$4]+=$10} END {for(v in a)print a[v]" - "v}' WA_Sales_Products_2012-14.csv
		```
		Maka akan keluar seluruh *produk line* pada penjualan negara tersebut.
	3. Lalu *sort* secara *descending*(urut dari terbesar ke terkecil), menggunakan *command* `sort -nr`.
		```sh
		$ awk -F "," '{if ($7=="2012" && $1=="United States") a[$4]+=$10} END {for(v in a)print a[v]" - "v}' WA_Sales_Products_2012-14.csv | sort -nr
		```
	4. Terakhir `print` tiga teratas menggunakan command `head -3` dan cukup `print` nama _produk line_ nya saja menggunakan `awk`.
		```sh
		$ awk -F "," '{if ($7=="2012" && $1=="United States") a[$4]+=$10} END {for(v in a)print a[v]" - "v}' WA_Sales_Products_2012-14.csv | sort -nr | head -3 | awk -F "-" '{print $2}'
		```
		Maka akan tercetak tiga _produk line_ teratas dari poin a, yaitu :
		``` 
		 Personal Accessories
		 Camping Equipment
		 Outdoor Protection
		```
	c. Tiga produk yang memberikan penjualan(_quantity_) terbanyak berdasarkan tiga  _product line_  yang didapatkan pada soal poin b : 
	1. Masih menggunakan syntax `awk` seperti pada poin sebelumnya, namun kali ini menggunakan syntax lebih banyak, dengan menyantumkan _produk line_ pada kolom 4. Lalu, `print` produk pada kolom 6.
		```sh
		$ awk -F "," '{if($7=="2012" && $1=="United States" && ($4=="Personal Accessories" || $4=="Camping Equipment" || $4=="Outdoor Protection")) a[$6]+=$10} END {for(v in a)print a[v]" - "v}' WA_Sales_Products_2012-14.csv
		```
	2. Setelah itu lakukan *sort* secara *descending*(dari yang terbesar ke terkecil) dan munculkan tiga produk teratas.
		```sh
		$ awk -F "," '{if($7=="2012" && $1=="United States" && ($4=="Personal Accessories" || $4=="Camping Equipment" || $4=="Outdoor Protection")) a[$6]+=$10} END {for(v in a)print a[v]" - "v}' WA_Sales_Products_2012-14.csv | sort -nr | head -3
		```
	3. Terakhir `print` namanya saja menggunakan `awk`.
		```sh
		$ awk -F "," '{if($7=="2012" && $1=="United States" && ($4=="Personal Accessories" || $4=="Camping Equipment" || $4=="Outdoor Protection")) a[$6]+=$10} END {for(v in a)print a[v]" - "v}' WA_Sales_Products_2012-14.csv | sort -nr | head -3 | awk -F "-" '{print $2}' 
		``` 
		Maka tiga produk teratas dari poin b ialah : 
		```sh
		 Zone
		 TrailChef Water Bag
		 Single Edge
		```
	> See : [full code](../soal2.sh)

3. Buatlah sebuah script bash yang dapat menghasilkan password secara acak
sebanyak 12 karakter yang terdapat huruf besar, huruf kecil, dan angka. Password
acak tersebut disimpan pada file berekstensi .txt dengan ketentuan pemberian nama
sebagai berikut:  
a. Jika tidak ditemukan file password1.txt maka password acak tersebut
disimpan pada file bernama password1.txt  
b. Jika file password1.txt sudah ada maka password acak baru akan
disimpan pada file bernama password2.txt dan begitu seterusnya.  
c. Urutan nama file tidak boleh ada yang terlewatkan meski filenya
dihapus.  
d. Password yang dihasilkan tidak boleh sama.  
**Jawab :**
	```sh
    #!/bin/bash

    number=1
    suffix=1
    while test -e "password$suffix.txt";
	    do
		    ((++number))
		    suffix="$number"
		done
    fname="password$suffix.txt"
    randomnum=$(</dev/urandom tr -dc A-Z-a-z-0-9 | head -c12)
    echo "$randomnum" > "$fname"
	```  
	> See : [full code](../soal3.sh)  
	
    **Penjelasan :**  
    1. Inisial variabel :
	    ```sh
	    number=1
	    suffix=1
	    ```
	    `suffix=1` karena nama file pasti dimulai dari password1.txt  
	    `number=1` sebagai temporary untuk memeriksa file yang ada nantinya.
    2. Lakukan *looping* menggunakan `while` ketika `test -e "password$suffix"` , untuk memeriksa apakah file itu ada atau tidak, jika file tersebut ada maka _looping_ akan berjalan dengan menambahkan +1 kedalam `suffix` hingga ke file yang belum ada.
	    ```sh
	    while test -e "password$suffix.txt"
	        do
                ((++number))
                suffix="$number" 
	        done
	    ```
    3. Buat nama file yang belum ada.
	    ```sh
	    fname="password$suffix.txt" 
	    ```
    4. Random string A-Z, a-z, dan 1-9 menggunakan `/dev/urandom` dengan `tr`, lalu ambil bagian depan saja menggunakan `head -c12`.
	    ```sh
	    randomnum=$(</dev/urandom tr -dc A-Z-a-z-0-9 | head -c12)
		```
    5. cetak password acak tersebut kedalam file teks.
	    ```sh
	    echo "$randomnum" > "$fname"
	    ```
    
4. Lakukan backup file syslog setiap jam dengan format nama file “jam:menit tanggal-bulan-tahun”. Isi dari file backup terenkripsi dengan konversi huruf (string manipulation) yang disesuaikan dengan jam dilakukannya backup misalkan sebagai berikut:  
	a. Huruf b adalah alfabet kedua, sedangkan saat ini waktu menunjukkan pukul 12, sehingga huruf b diganti dengan huruf alfabet yang memiliki urutan ke 12+2 = 14.  
    b. Hasilnya huruf b menjadi huruf n karena huruf n adalah huruf ke empat belas, dan seterusnya.  
    c. setelah huruf z akan kembali ke huruf a  
    d. Backup file syslog setiap jam.  
    e. dan buatkan juga bash script untuk dekripsinya.  
    **Jawab :** 
    > Bash file. See: [full code](../soal4.sh)
    ```sh
    #!/bin/bash
    lowercase=(a b c d e f g h i j k l m n o p q r s t u v w x y z)
    uppercase=(A B C D E F G H I J K L M N O P Q R S T U V W X Y Z)
    tgl=`date +"%d"`
    bln=`date +"%m"`
    thn=`date +"%Y"`
    jam=`date +"%H"`
    mnt=`date +"%M"`
    home="/home/sherly"
    namafile="$home/$jam:$mnt $tgl-$bln-$thn.txt"
    bawah=${lowercase[$jam]}
    atas=${uppercase[$jam]}
    cat /var/log/syslog > /home/sherly/syslog4.txt
    cat /home/sherly/syslog4.txt | tr '[a-z]' "[$bawah-za-$bawah]" | tr '[A-Z]' "[$atas-ZA-$atas]" > "$namafile"
    ```
    > Crontab. See: [full code](../soal4.sh)
    ```sh
     0 */1 * * * ~/SoalShift_modul1_B02/soal4.sh
    ```
    **Penjelasan :**  
     Menjadikan file syslog sebagai input dengan command cat setelah itu char yang berada pada range a-z akan diubah satu persatu sesuai jam
Contoh :
apabila jam menunjukkan 2, maka a menjadi c, b menjadi d, c menjadi e, dan seterusnya. 
Range [b-za-b] adalah dari char b sampai dengan z, kemudian dilanjut dari char a sampai dengan b.
Berlaku demikian untuk huruf besar

5. Buatlah sebuah script bash untuk menyimpan record dalam syslog yang memenuhi kriteria berikut:  
	a. Tidak mengandung string “sudo”, tetapi mengandung string “cron”, serta buatlah pencarian stringnya tidak bersifat case sensitive, sehingga huruf kapital atau tidak, tidak menjadi masalah.  
    b. Jumlah field (number of field) pada baris tersebut berjumlah kurang dari 13.  
    c. Masukkan record tadi ke dalam file logs yang berada pada direktori /home/[user]/modul1.  
    d. Jalankan script tadi setiap 6 menit dari menit ke 2 hingga 30, contoh
13:02, 13:08, 13:14, dst.  
	**Jawab :**  
	> [Full code](../soal5.sh)
	```sh
	tstmp = `date '+%Y-%m-%d_%H:%M'`
	awk '{IGNORECASE=1}NF<13 !/sudo/ && /cron/{print} /var/log/syslog > ~/modul1/syslog_$tstmp.bak
	```
	> [crontab](../crontab)  
	```sh
      2-30/6 * * * * ~/SoalShift_modul1_B02/soal5.sh
	```
    **Penjelasan :**
	1. Dengan menggunakan syntax `awk` biasa.
	2. Tidak mengandung "sudo" dan mengandung "cron" : `!/sudo/ && /cron/`.
	3. Tidak bersifat _case sensitive_ : `IGNORECASE=1`.
	4. Jumlah _field_ kurang dari 13 : `NF < 13`. Maka:  
		```sh
		$ awk '{IGNORECASE=1}NF<13 !/sudo/ && /cron/{print} /var/log/syslog
		```
	5. Setelah itu _write_ hasil awk ke dalam _directory_ ~/modul1/ dan kita juga dapat memberikan _timestamp_ menggunakan _command_ `date` dikarenakan program ini nantinya akan berjalan tiap 6 menit. Maka jadilah : 
		```sh
		#!/bin/bash
		
		tstmp = `date '+%Y-%m-%d_%H:%M'`
		awk '{IGNORECASE=1}NF<13 !/sudo/ && /cron/{print} /var/log/syslog > ~/modul1/syslog_$tstmp.bak
		```
	6. Terakhir jalankan program menggunakan `crontab`.
		```sh
 		 2-30/6 * * * * ~/SoalShift_modul1_B02/soal5.sh
		``` 
		
	> See :
	> [Full code](../soal5.sh)
	> [crontab](../crontab)  
