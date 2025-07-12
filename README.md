# Cara-Menginstal-Moodle-LMS-di-VPS-CloudKilat

Halo, **_Kawan Belajar!_** 

Ingin membangun platform _e-learning_ yang andal dan mudah dikelola? **Moodle**, sebagai salah satu _Learning Management System (LMS) open-source_ terpopuler, adalah pilihan tepat untuk kebutuhan pendidikan online Anda. Dengan **VPS** dari CloudKilat, Anda dapat menginstal _Moodle_ dengan cepat dan menikmati performa hosting berbasis teknologi komputasi awan yang unggul. 

Dalam artikel ini, kami akan memandu Anda langkah demi langkah untuk _menginstal Moodle di VPS Kilat VM_, lengkap dengan spesifikasi minimum yang dibutuhkan dan alasan mengapa layanan ini ideal untuk _e-learning_. _**Yuk, simak!**_

<br>

### 1. Prasyarat

1.  VPS atau server dengan Ubuntu 24.04 (Kilat VM).
2.  Pengguna non-root dengan hak sudo.
3.  Domain yang sudah mengarah ke IP server.
4.  Moodle 4.0.4
### 2. Instalasi Dependencies (LAMP Stack)

Lakukan akses layanan VPS (Kilat VM 2.0) dan untuk detailnya dapat dilihat pada tautan berikut: [**[Klik di sini](https://kb.cloudkilat.id/akses-kilat-vm)**]. setelah berhasil masuk silakan dilakukan update sistem dan instalasi LAMP dengan perintah:

```
sudo apt update  
sudo apt install apache2 mariadb-server php-cli php-intl php-xmlrpc php-soap php-mysql php-zip php-gd php-tidy php-mbstring php-curl php-xml php-pear php-bcmath libapache2-mod-php
```
![install_LampStack.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752301652_install_LampStack.png)
Gambar 1. Install Dependencies

Kemudian, aktifkan dan periksa status layanan Apache dan MariaDB dengan perintah yang sesuai.

```
sudo systemctl is-enabled apache2
sudo systemctl status apache2
sudo systemctl is-enabled mariadb
sudo systemctl status mariadb
```
![status_apache.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752301702_status_apache.png)
Gambar 2. Status Apache2

![StatusDB.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752301720_StatusDB.png)
Gambar 3. Status MariaDB

Terakhir pastikan versi PHP dan Module PHP yang terinstall

```
php -v && php -m
```

![phpvmod.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752301906_phpvmod.png)
Gambar 4. Versi dan Module PHP

### 3. Konfigurasi MariaDB

Setelah dependensi terinstal, ubah mesin penyimpanan default MariaDB menjadi **InnoDB** yang dibutuhkan oleh Moodle, dengan mengedit konfigurasi server MariaDB, lalu amankan instalasi server menggunakan utilitas *mariadb_secure_installation*.

Lakukan editing pada file `/etc/mysql/mariadb.conf.d/50-server.cnf` dan tambahkan:

```
vi /etc/mysql/mariadb.conf.d/50-server.cnf
```

Tambahkan

```
[mysqld]
innodb_file_format = Barracuda  
default_storage_engine = innodb  
innodb_large_prefix = 1  
innodb_file_per_table = 1  
```

![AddConfigInnoDB.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752301937_AddConfigInnoDB.png)
Gambar 5. Penambahan konfigurasi MariaDB

Setelah itu, lakukan restart dan jalankan proses pengamanan.

```
sudo systemctl restart mariadb  
sudo mariadb_secure_installation
```

> Dalam prosesnya, Anda akan diminta untuk menjawab beberapa pertanyaan berikut:
>    * Untuk instalasi default server MariaDB tanpa kata sandi root, tekan ENTER saat diminta memasukkan kata sandi.
>    * Karena autentikasi lokal untuk pengguna root MariaDB sudah diamankan secara default, ketik 'n' saat ditanya apakah ingin mengubah metode autentikasi ke unix_socket.
>    * Ketik 'Y' untuk membuat kata sandi root MariaDB yang baru. Kemudian, masukkan kata sandi yang kuat untuk pengguna root MariaDB dan ulangi kembali.
>    * Saat ditanya apakah ingin menonaktifkan autentikasi jarak jauh untuk pengguna root MariaDB, ketik 'Y' untuk menyetujui.
>    * Instalasi default MariaDB menyertakan database 'test' dan mengizinkan pengguna anonim mengaksesnya. Ketik 'Y' pada kedua opsi untuk menghapus database 'test' dan menghapus akses anonim.
>    * Terakhir, ketik 'Y' untuk mengonfirmasi proses reload hak akses tabel.

![SecureDb.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302146_SecureDb.png)
Gambar 6. Pengamanan user root mariaDB

### 4. Buat Database Moodle

Sekarang setelah Anda mengonfigurasi server MariaDB, mari buat database dan pengguna baru melalui klien mariadb. Masuk ke server MariaDB dengan perintah klien mariadb di bawah ini, lalu masukkan kata sandi root MariaDB Anda saat diminta.

Berikut perintah login ke server MySQL

```
sudo mariadb -u root -p
```

Kemudian, jalankan perintah berikut untuk membuat database, pengguna, menetapkan kata sandi, serta memberikan hak akses (privilege) penuh kepada pengguna yang telah dibuat:

```
CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
GRANT ALL ON moodle.* TO 'moodle'@'localhost' IDENTIFIED BY "MoodlePassw0rd";
FLUSH PRIVILEGES;
EXIT;
```

> **Catatan:** Sesuaikan nama Database, User Database dan Password user database dengan aman.

![CreateDBMoodle.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302175_CreateDBMoodle.png)
Gambar 7. Pembuatan database

### 5. Konfigurasi PHP

Setelah itu lakukan penyesuaian awal pada file konfigurasi PHP pada file `/etc/php/8.3/apache2/php.ini`:

```
sudo vi /etc/php/8.3/apache2/php.ini
```

Sesuaikan dengan alokasi memory yang akan Anda terapkan

```
memory_limit = 256M  
upload_max_filesize = 60M  
max_execution_time = 300  
date.timezone = <Zona_Waktu_Anda>  
max_input_vars = 5000
```

![phpini.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302319_phpini.png)
Gambar 8. Tunning awal PHP

Setelah konfigurasi disesuaikan dan disimpan lakukan restart service apache dengan perintah:

```
sudo systemctl restart apache2
```

### 6. Download & Setup Moodle

Sekarang LAMP Stack sudah terpasang dan dikonfigurasi. Selanjutnya, kita akan unduh source code Moodle dan atur direktori instalasinya.

Masuk ke direktori _/var/www_, lalu gunakan perintah _wget_ untuk mengunduh source code Moodle. Pastikan kamu cek halaman resmi Moodle untuk link versi terbarunya. Dalam contoh ini, kita akan pakai versi stabil **Moodle 4.0.4**.

```
cd /var/www  
wget https://download.moodle.org/download.php/direct/stable404/moodle-latest-404.tgz  
```

![Downlodmoodle.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302350_Downlodmoodle.png)
Gambar 9. download source code moodle


Setelah Moodle berhasil diunduh, ekstrak file-nya dengan perintah tar di bawah ini. Source code Moodle nantinya akan tersedia di direktori /var/www/moodle.

```
tar xvf moodle-latest-404.tgz  
```

![extrakmoodle.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302368_extrakmoodle.png)
Gambar 10. ektrak source code moodle

Terakhir, jalankan perintah berikut untuk membuat direktori data baru _/var/www/moodledata_, mengubah kepemilikan direktori Moodle ke user _www-data_, dan pastikan kedua direktori (Moodle dan moodledata) bisa ditulis oleh user www-data.

```
sudo mkdir -p /var/www/moodledata  
sudo chown -R www-data:www-data /var/www/moodle /var/www/moodledata  
sudo chmod u+rwx /var/www/moodle /var/www/moodledata
```

![addmoodledtchown.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302426_addmoodledtchown.png)
Gambar 11. menambah direktori moodledata

### 7. Konfigurasi Virtual Host Apache

Setelah Moodle berhasil diunduh, sekarang saatnya membuat file _virtual host Apache baru_ agar Moodle bisa dijalankan. Pastikan kamu sudah menyiapkan domain yang mengarah ke alamat IP VPS kamu.

Pertama, aktifkan modul rewrite dengan perintah a2enmod berikut:

```
sudo a2enmod rewrite
```

Lalu buat file virtual host Apache baru di /etc/apache2/sites-available/moodle.conf menggunakan editor vi:


```
vi /etc/apache2/sites-available/moodle.conf
```

![EditVirtualhost.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302481_EditVirtualhost.png)
Gambar 12. tambahkan virtualhost


Masukkan konfigurasi berikut dan jangan lupa ganti nama domain sesuai dengan nama domain yang kamu miliki:

```
<VirtualHost *:80>
 DocumentRoot /var/www/moodle/
 ServerName moodle.howtoforge.local
 ServerAdmin admin@example.com
 
 <Directory /var/www/moodle/>
 Options +FollowSymlinks
 AllowOverride All
 Require all granted
 </Directory>

 ErrorLog /var/log/apache2/moodle_error.log
 CustomLog /var/log/apache2/moodle_access.log combined
</VirtualHost>
```

Simpan file dan keluar dari editor jika sudah selesai.

Setelah itu, jalankan perintah berikut untuk mengaktifkan file _moodle.conf_ dan periksa apakah konfigurasi Apache kamu sudah benar:

```
sudo a2ensite moodle.conf
sudo apache2ctl configtest
```

Jika tidak ada kesalahan, kamu akan melihat output **"Syntax OK"**.

Terakhir, restart Apache untuk menerapkan semua perubahan:

```
sudo systemctl restart apache2
```

![Restartapc.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302515_Restartapc.png)
Gambar 13. restart webserver

### 8. Aktifkan HTTPS dengan Certbot & Let's Encrypt

Terakhir, kamu bisa membuat sertifikat SSL/TLS untuk mengamankan Moodle dengan HTTPS. Pada bagian ini, kita akan menerapkan HTTPS untuk Moodle menggunakan Certbot dan Let's Encrypt. Jika kamu menginstal Moodle secara lokal (tidak online), bagian ini bisa dilewati.

Kalau kamu ingin perlindungan lebih profesional dan terpercaya untuk situs produksi, kamu juga bisa menggunakan sertifikat SSL berbayar dari CloudKilat, seperti Sectigo SSL atau AlphaSSL. Kedua opsi ini menawarkan validitas lebih tinggi, dukungan teknis, dan tampilan "secure" yang lebih terpercaya di mata pengguna.

Namun, untuk keperluan standar dan gratis, cukup ikuti langkah-langkah berikut menggunakan Certbot:

Pertama, instal paket _certbot_ dan _python3-certbot-apache_ dengan perintah berikut:

```
sudo apt install certbot python3-certbot-apache -y
```

Setelah instalasi selesai, jalankan perintah certbot di bawah ini untuk membuat _sertifikat SSL/TLS_ untuk _Moodle_. Jangan lupa ganti nama domain dan alamat email sesuai dengan punyamu:

```
sudo certbot --apache --agree-tos --redirect --hsts --staple-ocsp --email kamu@example.com -d moodle.domainkamu.com
```

Setelah proses selesai, _sertifikat SSL_ kamu akan tersimpan di direktori:

```
/etc/letsencrypt/live/domainkamu.com
```

Dan instalasi Moodle kamu akan otomatis diamankan dengan HTTPS.

![sslmod.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302724_sslmod.png)
Gambar 13. Secure SSL


### 9. Installing Moodle

Buka domain Moodle kamu, misalnya https://moodle.domainkamu.com/, dan kamu akan melihat tampilan wizard instalasi Moodle.

Berikut langkah-langkah yang perlu kamu ikuti:

a. Pilih bahasa default yang ingin digunakan, lalu klik Next.

![Instalasiweb.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302740_Instalasiweb.png)
Gambar 14. pilih bahasa

b. Masukkan lokasi direktori data Moodle, yaitu: _/var/www/moodledata_

![Instalasiweb1.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302935_Instalasiweb1.png)
Gambar 15. tentukan roor direktori

c. Pilih MariaDB sebagai driver database.

![Instalasiweb2.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302757_Instalasiweb2.png)
Gambar 16. sesuaikan database server

d. Masukkan detail database yang sudah kamu buat sebelumnya (nama database, username, dan password).

![Instalasiweb3.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302779_Instalasiweb3.png)
Gambar 17. penyesuaian database

e. Klik Continue untuk menyetujui pemberitahuan hak cipta (copyright notice).

![Instalasiweb4.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752302957_Instalasiweb4.png)
Gambar 18. setujui hak cipta

f. Pada bagian server checks, pastikan semua kebutuhan sistem sudah terpenuhi.

![Instalasiweb5.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752303017_Instalasiweb5.png)
Gambar 19. penyesuaian kebutuhan

g. Setelah itu, proses instalasi Moodle akan berjalan secara otomatis.

![Instalasiweb6.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752303042_Instalasiweb6.png)
Gambar 20. proses instalasi

h. Jika sudah selesai, kamu akan diminta untuk membuat akun admin baru. Masukkan username, email, dan password untuk akun admin Moodle kamu.

![Instalasiweb7.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752303063_Instalasiweb7.png)
Gambar 21. credential admin 

i. Setelah semua selesai, kamu akan diarahkan ke dashboard Moodle, dan siap untuk digunakan!

![SiapDiEksekusi.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752303167_SiapDiEksekusi.png)
Gambar 22. dashbord moodle

### 10. Troubleshooting Moodle

Pada saat bagian f, kami menemukan kendala error sesuai Gambar 23.

![Errortypemysql.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752303197_Errortypemysql.png)
Gambar 23. error type database

Dengan error itu kami melakukan penyesuaian pada file moodle/config.php seperti Gambar 24

```
vi moodle/config.php
```

lakukan penyesuaian pada bagian berikut:

```
$CFG->dbtype    = 'mariadb';
```

![resolveerror.png](https://s3-id-jkt-1.kilatstorage.id/knowledgebase/2025/07/1752303278_resolveerror.png)
Gambar 24. penyesuaian type database

### Kesimpulan

Selamat! Kamu telah berhasil menyelesaikan instalasi Moodle di server Ubuntu 24.04.

Kamu telah menginstal **Moodle 4.0.4** di Ubuntu dengan menggunakan LAMP Stack (Linux, Apache, MariaDB, dan PHP), serta mengamankan Moodle menggunakan HTTPS melalui _Certbot dan Let's Encrypt_.

Sekarang Moodle kamu sudah siap digunakan untuk keperluan belajar, mengajar, atau mengelola sistem e-learning! 🎉
