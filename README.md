# Jarkom-Modul-2-B10-2022

## Kelompok B10
| Name                      | NRP        | 
| ------------------------- | ---------- |
| Frederick Wijayadi Susilo | 5025201111 |
| Doanda Dresta Rahma       | 5025201049 |
| Zunia Aswaroh             | 5025201058 |

IP Prefix Kelompok: `10.8`

## Soal dan Jawaban

### Pendahuluan 
> Twilight (〈黄昏 (たそがれ) 〉, <Tasogare>) adalah seorang mata-mata yang berasal dari negara Westalis. Demi menjaga perdamaian antara Westalis dengan Ostania, Twilight dengan nama samaran Loid Forger (ロイド・フォージャー, Roido Fōjā) di bawah organisasi WISE menjalankan operasinya di negara Ostania dengan cara melakukan spionase, sabotase, penyadapan dan kemungkinan pembunuhan. Berikut adalah peta dari negara Ostania:

![image](https://user-images.githubusercontent.com/67154280/197450071-4f730306-a277-4c47-aa8e-276cdb0c0f65.png)

> WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden

### 1
> Semua node terhubung pada router Ostania, sehingga dapat mengakses internet.

### Penyelesaian
Pertama kita buat script di root dan dijalankan dengan file .bashrc dengan isi sebagai berikut:
  
- Ostania
  
  ```shell
  iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.8.0.0/16
  cat /etc/resolv.conf
  ```
  
- WISE, dll
  
  ```shell
  echo "nameserver 192.168.122.1" > /etc/resolv.conf
  cat /etc/resolv.conf
  ```
  
Setelah itu, kita lakukan test koneksi dengan `ping google.com -c 4`.
  
![image](https://user-images.githubusercontent.com/67154280/197733942-e1ca69cc-049f-4917-aae2-75003131f531.png)

### 2
> Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses **wise.yyy.com** dengan alias **www.wise.yyy.com** pada folder wise.

### Penyelesaian
Pertama kita membuat file named-1.conf.local dan wise-1.b10.com pada root node `WISE` dengan isi sebagai berikut:
  
- named-1.conf.local
  
  ```shell
  zone "wise.b10.com" {
        type master;
        file "/etc/bind/wise/wise.b10.com";
  };
  ```
- wise-1.b10.com
  
  ```shell
  ;
  ; BIND data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     wise.b10.com. root.wise.b10.com. (
                       2022100601         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
  ;
  @       IN      NS      wise.b10.com.
  @       IN      A       10.8.3.2
  www     IN      CNAME   wise.b10.com.
  ```

Setelah itu, kita buat script `soal2.sh` dengan isi sebagai berikut untuk menambahkan zone domain utama dan membuat website utama pada folder wise
  
  ```shell
  apt-get update
  apt-get install bind9 -y

  cp /root/named-1.conf.local /etc/bind/named.conf.local

  mkdir /etc/bind/wise

  cp /root/wise-1.b10.com /etc/bind/wise/wise.b10.com

  service bind9 restart
  ```
  
Lalu, kita lakukan test pada client `SSS` dan `Garden` dengan `ping wise.b10.com -c 4` dan `host -t CNAME www.wise.b10.com` dengan mengubah nameserver menjadi `10.8.3.2`.
  
![image](https://user-images.githubusercontent.com/67154280/197740918-2e10eea7-d520-4c33-bc3c-1a02e269fa1c.png)

### 3
> Setelah itu ia juga ingin membuat subdomain **eden.wise.yyy.com** dengan alias **www.eden.wise.yyy.com** yang diatur DNS-nya di WISE dan mengarah ke Eden.

### Penyelesaian
Pertama kita membuat file wise-2.b10.com pada root node `WISE` dengan isi sebagai berikut: 

  ```shell
  ;
  ; BIND data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     wise.b10.com. root.wise.b10.com. (
                       2022100601         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
  ;
  @       IN      NS      wise.b10.com.
  @       IN      A       10.8.3.2
  www     IN      CNAME   wise.b10.com.
  eden    IN      A       10.8.2.3
  www.eden IN     CNAME   eden.wise.b10.com.
  ```
  
Setelah itu, kita buat script `soal3.sh` dengan isi sebagai berikut untuk membuat subdomain eden.wise.b10.com pada folder wise
  
  ```shell
  cp /root/wise-2.b10.com /etc/bind/wise/wise.b10.com

  service bind9 restart
  ```
  
Lalu, kita lakukan test pada client `SSS` dan `Garden` dengan `ping eden.wise.b10.com -c 4` dan `host -t CNAME www.eden.wise.b10.com`.
  
![image](https://user-images.githubusercontent.com/67154280/197744058-98596e5c-daa4-4b2f-af52-0f997865cd86.png)


### 4
> Buat juga reverse domain untuk domain utama.

### Penyelesaian
Pertama kita membuat file named-2.conf.local dan 3.8.10.in-addr.arpa-1 pada root node `WISE` dengan isi sebagai berikut:
  
- named-2.conf.local
  
  ```shell
  zone "wise.b10.com" {
          type master;
          file "/etc/bind/wise/wise.b10.com";
  };

  zone "3.8.10.in-addr.arpa" {
      type master;
      file "/etc/bind/wise/3.8.10.in-addr.arpa";
  };
  ```
- 3.8.10.in-addr.arpa-1
  
  ```shell
  ;
  ; BIND data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     wise.b10.com. root.wise.b10.com. (
                          2022100601      ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
  ;
  3.8.10.in-addr.arpa. IN NS      wise.b10.com.
  2                    IN PTR     wise.b10.com.
  ```

Setelah itu, kita buat script `soal4.sh` dengan isi sebagai berikut untuk menambahkan zone reverse domain dan membuat reverse domain (in-addr.arpa) pada folder wise

  ```shell
  cp /root/named-2.conf.local /etc/bind/named.conf.local
  cp /root/3.8.10.in-addr.arpa-1 /etc/bind/wise/3.8.10.in-addr.arpa

  service bind9 restart
  ```
  
Lalu, kita lakukan test pada client `SSS` dan `Garden` dengan `host -t PTR 10.8.3.2`.
  
![image](https://user-images.githubusercontent.com/67154280/197747909-d9f11e86-0fcd-49dc-b8df-8254df1d2958.png)


### 5
> Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama.

### Penyelesaian
Pertama kita membuat file named-3.conf.local pada root node `WISE` dengan isi sebagai berikut:
  
  ```shell
  zone "wise.b10.com" {
          type master;
          notify yes;
          also-notify { 10.8.2.2; };
          allow-transfer { 10.8.2.2; };
          file "/etc/bind/wise/wise.b10.com";
  };

  zone "3.8.10.in-addr.arpa" {
      type master;
      file "/etc/bind/wise/3.8.10.in-addr.arpa";
  };
  ```

Kita juga membuat script `soal5.sh` untuk menambahkan konfigurasi ketika beralih ke Berlint dengan isian sebagai berikut:
  
  ```shell
  cp /root/named-3.conf.local /etc/bind/named.conf.local

  service bind9 restart
  ```
  
Lalu, kita membuat file named-1.conf.local pada root node `Berlint` dengan isi sebagai berikut:
  
  ```shell
  zone "wise.b10.com" {
      type slave;
      masters { 10.8.3.2; };
      file "/var/lib/bind/wise.b10.com";
  };
  ```

Kita juga membuat script `soal5.sh` untuk membuat Berlint menjadi DNS Slave dengan isian sebagai berikut:
  
  ```shell
  apt-get update
  apt-get install bind9 -y

  cp /root/named-1.conf.local /etc/bind/named.conf.local

  service bind9 restart
  ```
  
Untuk pengecekan, kita membuat file script test-stop.sh pada root node `WISE` dengan command `service bind9 stop`. Setelah itu, kita test slave pada client `SSS` dan `Garden` dengan command `ping wise.b10.com -c 4`.

![image](https://user-images.githubusercontent.com/67154280/197752672-9c9da922-7cdf-403e-b8a0-4e21feea2345.png)

### 6
> Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu **operation.wise.yyy.com** dengan alias **www.operation.wise.yyy.com** yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation.

### Penyelesaian
Pertama kita membuat file named-1.conf.options dan wise-3.b10.com pada root node `WISE` dengan isi sebagai berikut:
  
- named-1.conf.options
  
  ```shell
  options {
          directory "/var/cache/bind";

          // If there is a firewall between you and nameservers you want
          // to talk to, you may need to fix the firewall to allow multiple
          // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

          // If your ISP provided one or more IP addresses for stable
          // nameservers, you probably want to use them as forwarders.
          // Uncomment the following block, and insert the addresses replacing
          // the all-0's placeholder.

          // forwarders {
          //      0.0.0.0;
          // };

          //=====================================================================$
          // If BIND logs error messages about the root key being expired,
          // you will need to update your keys.  See https://www.isc.org/bind-keys
          //=====================================================================$
          // dnssec-validation auto;
          allow-query{any;};

          auth-nxdomain no;    # conform to RFC1035
          listen-on-v6 { any; };
  };
  ```
  
- wise-3.b10.com
  
  ```shell
  ;
  ; BIND data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     wise.b10.com. root.wise.b10.com. (
                       2022100601         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
  ;
  @       IN      NS      wise.b10.com.
  @       IN      A       10.8.3.2
  www     IN      CNAME   wise.b10.com.
  eden    IN      A       10.8.2.3
  www.eden IN     CNAME   eden.wise.b10.com.
  ns1     IN      A       10.8.2.3
  operation IN    NS      ns1
  ```

Kita juga membuat script `soal6.sh` untuk menambahkan delegasi subdomain berupa operation serta meng-allow semua query dengan isian sebagai berikut:
  
  ```shell
  cp /root/named-1.conf.options /etc/bind/named.conf.options
  cp /root/wise-3.b10.com /etc/bind/wise/wise.b10.com

  service bind9 restart
  ```

Lalu, kita membuat file named-1.conf.options, named-2.conf.local, dan operation-1.wise.b10.com pada root node `Berlint` dengan isi sebagai berikut:
  
- named-1.conf.options
  
  ```shell
  options {
          directory "/var/cache/bind";

          // If there is a firewall between you and nameservers you want
          // to talk to, you may need to fix the firewall to allow multiple
          // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

          // If your ISP provided one or more IP addresses for stable
          // nameservers, you probably want to use them as forwarders.
          // Uncomment the following block, and insert the addresses replacing
          // the all-0's placeholder.

          // forwarders {
          //      0.0.0.0;
          // };

          //=====================================================================$
          // If BIND logs error messages about the root key being expired,
          // you will need to update your keys.  See https://www.isc.org/bind-keys
          //=====================================================================$
          // dnssec-validation auto;
          allow-query{any;};

          auth-nxdomain no;    # conform to RFC1035
          listen-on-v6 { any; };
  };
  ```
  
- named-2.conf.local
  
  ```shell
  zone "wise.b10.com" {
      type slave;
      masters { 10.8.3.2; };
      file "/var/lib/bind/wise.b10.com";
  };

  zone "operation.wise.b10.com" {
     type master;
     file "/etc/bind/operation/operation.wise.b10.com";
  };
  ```
  
- operation-1.wise.b10.com
  
  ```shell
  ;
  ; BIND data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     operation.wise.b10.com. root.operation.wise.b10.com. (
                          2022100601      ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
  ;
  @       IN      NS      operation.wise.b10.com.
  @       IN      A       10.8.2.3
  www     IN      CNAME   operation.wise.b10.com.
  ```
  
Kita juga membuat script `soal6.sh` sebagai pembuatan alias dari delegasi subdomain dengan isian sebagai berikut:
  
  ```shell
  cp /root/named-1.conf.options /etc/bind/named.conf.options
  cp /root/named-2.conf.local /etc/bind/named.conf.local

  mkdir /etc/bind/operation
  cp /root/operation-1.wise.b10.com /etc/bind/operation/operation.wise.b10.com

  service bind9 restart
  ```
  
Lalu, kita lakukan test pada client `SSS` dan `Garden` dengan `ping operation.wise.b10.com -c 4` dan `ping www.operation.wise.b10.com -c 4`.
  
![image](https://user-images.githubusercontent.com/67154280/197771909-fa2179ed-7c16-4780-b6a5-11634ef9ee6c.png)

### 7
> Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses **strix.operation.wise.yyy.com** dengan alias **www.strix.operation.wise.yyy.com** yang mengarah ke Eden.

### Penyelesaian
Pertama kita membuat file operation-2.wise.b10.com pada root node `Berlint` dengan isi sebagai berikut:
  
  ```shell
  ;
  ; BIND data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     operation.wise.b10.com. root.operation.wise.b10.com. (
                          2022100601      ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
  ;
  @       IN      NS      operation.wise.b10.com.
  @       IN      A       10.8.2.3
  www     IN      CNAME   operation.wise.b10.com.
  strix   IN      A       10.8.2.3
  www.strix IN    CNAME   strix.operation.wise.b10.com.
  ```
  
Kita juga membuat script `soal7.sh` untuk menambahkan subdomain agar informasinya spesifik mengenai Operation Strix dengan isian sebagai berikut:
  
  ```shell
  cp /root/operation-2.wise.b10.com /etc/bind/operation/operation.wise.b10.com

  service bind9 restart
  ```
  
Lalu, kita lakukan test pada client `SSS` dan `Garden` dengan `ping strix.operation.wise.b10.com -c 4` dan `ping www.strix.operation.wise.b10.com -c 4`.
  
![image](https://user-images.githubusercontent.com/67154280/197773578-3d916eca-7b5c-4b8c-b0f4-fe3015aa5588.png)

### 8
> Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver **www.wise.yyy.com**. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com.

### Penyelesaian
***Pada node SSS dan Garden***

Lakukan `apt-get update` dan juga penginstalan lynx dengan cara 

```
apt-get update
apt-get install lynx -y
```

***Pada node Eden***

Lakukan instalasi Apache, php, openssl serta git,unzip nya untuk melakukan download ke website https dengan cara

```
apt-get install apache2 -y
apt-get install php -y
apt-get install libapache2-mod-php7.0 -y
apt-get install ca-certificates openssl -y
apt-get install unzip -y
apt-get install git -y
```

Untuk mendownload file, lakukan `git clone https://github.com/WantToBePro31/modul2source-jarkom.git` (repository tambahan khusus file zip yang dibutuhkan) dan unzip file nya dengan command `unzip -o /root/modul2source-jarkom/\*.zip -d /root/modul2source-jarkom`.

![image](https://github.com/zunia25/praktikum-modul-2-/blob/main/ls_dokumen.png)

Copy file `default-wise-1.conf` ke `wise.b10.com.conf` dengan perintah `cp /root/default-wise-1.conf /etc/apache2/sites-available/wise.b10.com.conf`. 
  
Kemudian aktifkan website dengan `a2ensite wise.b10.com`, lalu buat direktori `var/www/wise.b10.com` dengan perintah `mkdir /var/www/wise.b10.com` dan copy file dengan perintah `cp /root/wise.b10.com/index.php /var/www/wise.b10.com`.
  
Restart apache dengan `service apache2 restart`.

Lakukan tes pada node SSS dan Garden dengan `lynx http://www.wise.b10.com` atau `lynx http://www.wise.b10.com/index.php` maka akan mendapatkan tampilan seperti berikut 

![image](https://user-images.githubusercontent.com/67154280/198006471-78b62a5d-9d74-4bfe-9980-223e57522fac.png)

### 9
> Setelah itu, Loid juga membutuhkan agar url **www.wise.yyy.com/index.php/home** dapat menjadi menjadi **www.wise.yyy.com/home**.

### Penyelesaian
Pada node Eden, copy semua file di folder wise yang sudah diunzip dengan command `cp -r /root/modul2source-jarkom/wise/. /var/www/wise.b10.com`, selanjutnya konfigurasi file `wise.b10.com.conf` dengan `cp /root/default-wise-2.conf /etc/apache2/sites-available/wise.b10.com.conf` dengan isi `default-wise-2.conf` sebagai berikut.

```
<VirtualHost *:80>
        ServerName wise.b10.com
        ServerAlias www.wise.b10.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/wise.b10.com

        Alias "/home" "/var/www/wise.b10.com/index.php/home"

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Lakukan restart apache2 dengan `service apache2 restart`
  
Terakhir lakukan testing pada node Garden dan SSS dengan command `lynx www.wise.b10.com/home`

![image](https://user-images.githubusercontent.com/67154280/198043861-607baa70-0fcc-4a7a-bfcb-c201b913ffe2.png)

### 10
> Setelah itu, pada subdomain **www.eden.wise.yyy.com**, Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/eden.wise.yyy.com.

### Penyelesaian
***Pada node Eden***

Lakukan konfigurasi pada `default-wise-3.conf` dengan 

```
<VirtualHost *:80>
        ServerName eden.wise.b10.com
        ServerAlias www.eden.wise.b10.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/eden.wise.b10.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Kemudian copy file dengan perintah `cp /root/default-wise-3.conf /etc/apache2/sites-available/eden.wise.b10.com.conf`.

Lalu, aktifkan virtualhost dengan a2ensite, membuat direktori untuk documentroot di `/var/www/eden.wise.b10.com` dan jangan lupa untuk melakukan copy content ke documentroot dengan cara

```
a2ensite eden.wise.b10.com

mkdir /var/www/eden.wise.b10.com
cp -r /root/modul2source-jarkom/eden.wise/. /var/www/eden.wise.b10.com
```

Restart apache `service apache2 restart`.

Lakukan testing pada node Garden dan SSS dengan command `lynx http://www.eden.wise.b10.com`.

![image](https://user-images.githubusercontent.com/67154280/198212646-415c6aa5-46ce-4714-af97-c01170d5429f.png)

### 11
> Akan tetapi, pada folder /public, Loid ingin hanya dapat melakukan directory listing saja.

### Penyelesaian
***Pada node Eden***

Konfigurasi file `default-wise-4.conf` dengan 

```
<VirtualHost *:80>
        ServerName eden.wise.b10.com
        ServerAlias www.eden.wise.b10.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/eden.wise.b10.com

        <Directory /var/www/eden.wise.b10.com>
                Options +Indexes
        </Directory>

        <Directory /var/www/eden.wise.b10.com/error>
                Options -Indexes
        </Directory>

        <Directory /var/www/eden.wise.b10.com/public>
                Options +Indexes
        </Directory>
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

dan jangan lupa copy file dengan command `cp /root/default-wise-4.conf /etc/apache2/sites-available/eden.wise.b10.com.conf`.

Kemudian restart apache `service apache2 restart`.

Lalu, lakukan testing dengan command `lynx http://www.eden.wise.b10.com/public` maka akan mendapat tampilan seperti berikut.

![image](https://user-images.githubusercontent.com/67154280/198213720-5ffed1dc-75a7-4844-b932-8a68bb3c1039.png)

### 12
> Tidak hanya itu, Loid juga ingin menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache.

### Penyelesaian
***Pada node Eden***

Konfigurasi file `default-wise-5.conf` dengan menambhkan `ErrorDocument 404 /error/404.html` sehingga terdapat pesan error yang dibuat sendiri seperti berikut

```
<VirtualHost *:80>
        ServerName eden.wise.b10.com
        ServerAlias www.eden.wise.b10.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/eden.wise.b10.com

        <Directory /var/www/eden.wise.b10.com>
                Options +Indexes
        </Directory>

        <Directory /var/www/eden.wise.b10.com/public>
                Options +Indexes
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        ErrorDocument 404 /error/404.html
  </VirtualHost>
```
  
Copy file dengan command `cp /root/default-wise-5.conf /etc/apache2/sites-available/eden.wise.b10.com.conf`.

Restart apache `service apache2 restart`.

Lakukan testing pada node Garden dan SSS ketika mengakses url invalid seperti `lynx http://www.eden.wise.b10.com/wrong`, maka akan mendapatkan tampilan berikut.

![image](https://user-images.githubusercontent.com/67154280/198214501-1e667b6f-82f7-49a6-a3d1-84661a7ba05a.png)

### 13
> Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset **www.eden.wise.yyy.com/public/js** menjadi **www.eden.wise.yyy.com/js**.

### Penyelesaian
***Pada node Eden***

Lakukan konfigurasi pada file `default-wise-6.conf` dengan menambahkan `Alias "/js" "/var/www/eden.wise.b10.com/public/js"` sebagi berikut.

```
<VirtualHost *:80>
        ServerName eden.wise.b10.com
        ServerAlias www.eden.wise.b10.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/eden.wise.b10.com

        <Directory /var/www/eden.wise.b10.com>
                Options +Indexes
        </Directory>

        <Directory /var/www/eden.wise.b10.com/public>
                Options +Indexes
        </Directory>

        Alias "/js" "/var/www/eden.wise.b10.com/public/js"

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        ErrorDocument 404 /error/404.html
</VirtualHost>
```

Copy file dengan command `cp /root/default-wise-6.conf /etc/apache2/sites-available/eden.wise.b10.com.conf`. 

Restart apache `service apache2 restrat`.

Lakukan testing pada node Garden dan SSS ketika mengakses `lynx http://www.eden.wise.b10.com/js`, maka akan mendapatkan tampilan seperti berikut.

![image](https://user-images.githubusercontent.com/67154280/198215123-6afd18cf-3966-434d-a7c8-e2d6cae54404.png)

### 14
> Loid meminta agar **www.strix.operation.wise.yyy.com** hanya bisa diakses dengan port 15000 dan port 15500.

### Penyelesaian
***Pada node Eden***

Membuat konfigurasi Web Server di `default-wise-1-15000.conf` dan `default-wise-1-15500.conf` sebaagi berikut.

- default-wise-1-15000.conf

  ```
  <VirtualHost *:15000>
          ServerName strix.operation.wise.b10.com
          ServerAlias www.strix.operation.wise.b10.com

          ServerAdmin webmaster@localhost
          DocumentRoot /var/www/strix.operation.wise.b10.com

          ErrorLog ${APACHE_LOG_DIR}/error.log
          CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```

- default-wise-1-15500.conf

  ```
  <VirtualHost *:15500>
          ServerName strix.operation.wise.b10.com
          ServerAlias www.strix.operation.wise.b10.com

          ServerAdmin webmaster@localhost
          DocumentRoot /var/www/strix.operation.wise.b10.com

          ErrorLog ${APACHE_LOG_DIR}/error.log
          CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```

Kemudian copy file dengan command `cp /root/default-wise-1-15000.conf /etc/apache2/sites-available/strix.operation.wise.b10.com-15000.conf` dan `cp /root/default-wise-1-15500.conf /etc/apache2/sites-available/strix.operation.wise.b10.com-15000.conf`

Tambahkan port yang akan di listen pada `ports-1.conf` sebagi berikut.

```
Listen 80
Listen 15000
Listen 15500

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>
```

Kemudian copi file `cp /root/ports-1.conf /etc/apache2/ports.conf`.

Kemudian aktifkan a2ensite dengan `a2ensite strix.operation.wise.b10.com-15000` dan `a2ensite strix.operation.wise.b10.com-15500`.

Lakukan pembuatan direktori baru dengan `mkdir /var/www/strix.operation.wise.b10.com`.

Copy file - file lampiran github ke folder yang telah dibuat `cp -r /root/modul2source-jarkom/strix.operation.wise/. /var/www/strix.operation.wise.b10.com`.

Restart apache `service apache2 restart`.

Pada node Garden dan SSS, kita dapat melakukan testing dengan menggunakan lynx pada port 15000 atau 15500 yaitu `lynx http://www.strix.operation.wise.b10.com:15000` atau `lynx http://www.strix.operation.wise.b10.com:15500`.

![image](https://user-images.githubusercontent.com/67154280/198248486-8379c5f3-f550-4c33-ab4d-af1c576844fe.png)

![image](https://user-images.githubusercontent.com/67154280/198248537-d1952e86-3310-4870-9255-ae334215c3ea.png)

### 15
> dengan autentikasi username Twilight dan password opStrix dan file di /var/www/strix.operation.wise.yyy

### Penyelesaian
***Pada node Eden*** 

Tambahkan code baru berikut pada file `default-wise-2-15000.conf` dan `default-wise-2-15500.conf` sebagai berikut.

- default-wise-2-15000.conf

  ```
  <VirtualHost *:15000>
          ServerName strix.operation.wise.b10.com
          ServerAlias www.strix.operation.wise.b10.com

          ServerAdmin webmaster@localhost
          DocumentRoot /var/www/strix.operation.wise.b10.com

          ErrorLog ${APACHE_LOG_DIR}/error.log
          CustomLog ${APACHE_LOG_DIR}/access.log combined

          <Directory "var/www/strix.operation.wise.b10.com">
                  AuthType Basic
                  AuthName "Restricted Content"
                  AuthUserFile /etc/apache2/.htpasswd
                  Require valid-user
          </Directory>
  </VirtualHost>
  ```

- default-wise-2-15500.conf

  ```
  <VirtualHost *:15500>
          ServerName strix.operation.wise.b10.com
          ServerAlias www.strix.operation.wise.b10.com

          ServerAdmin webmaster@localhost
          DocumentRoot /var/www/strix.operation.wise.b10.com

          ErrorLog ${APACHE_LOG_DIR}/error.log
          CustomLog ${APACHE_LOG_DIR}/access.log combined

          <Directory "var/www/strix.operation.wise.b10.com">
                  AuthType Basic
                  AuthName "Restricted Content"
                  AuthUserFile /etc/apache2/.htpasswd
                  Require valid-user
          </Directory>
  </VirtualHost>
  ```

Copy file dengan command `cp /root/default-wise-2-15000.conf /etc/apache2/sites-available/strix.operation.wise.b10.com-15000` dan `cp /root/default-wise-2-15500.conf /etc/apache2/sites-available/strix.operation.wise.b10.com-15500`.

Kemudian buat autentikasi baru dengan command berikut sehingga memunculkan file .htpasswd pada dengan command `htpasswd -b -c /etc/apache2/.htpasswd Twilight opStrix`.

Restart apache `service apache2 restart`.

Ketika web server `strix.operation.wise.b10.com` diakses, akan diminta authentikasi username dan password.

Input username `Twilight` dan password `opStrix`.
  
![image](https://user-images.githubusercontent.com/67154280/198258402-66399df7-2668-4af9-91b5-b2b6ad76f54f.png)

Kemudian akan menampilakn hasil berikut.

![image](https://user-images.githubusercontent.com/67154280/198260798-a6185c38-0f1e-4b9b-8a74-ce1808345a04.png)

![image](https://user-images.githubusercontent.com/67154280/198262421-80f026e6-3068-4aa7-b785-e33b6690e441.png)

### 16
> dan setiap kali mengakses IP Eden akan dialihkan secara otomatis ke **www.wise.yyy.com**.

### Penyelesaian
***Pada node Eden***

Konfigurasi pada file `default-1.conf` sebagai berikut.
 
```
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/wise.b10.com

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Copy file tersebut dengan command `cp /root/default-1.conf /etc/apache2/sites-available/000-default.conf`.

Konfigurasi pada file `wise-1.htaccess` sebagai berikut.

```
a2enmod rewrite
RewriteEngine On
RewriteBase /
RewriteCond %{HTTP_HOST} ^10\.8\.2\.3$
RewriteRule ^(.*)$ http://www.wise.b10.com/$1 [L,R=301]
```

Inti dari konfigurasi tersebut adalah kita melakukan cek apakah pengaksesan berupa ip ke arah Eden jika hal tersebut terpenuhi maka kita membuat rule untuk melakukan direct ke `www.wise.b10.com`.

Kemudian copy file dengan command `cp /root/wise-1.htaccess /var/www/wise.b10.com/.htaccess`.

Restart apache `service apache2 restart`.

Testing pada node Garden dan SSS dengan command `lynx 10.8.2.3`. 
 
![image](https://user-images.githubusercontent.com/67154280/198265190-14a2900f-43f7-4ba4-a9df-72259a6de614.png)

### 17
> Karena website **www.eden.wise.yyy.com** semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring “eden” akan diarahkan menuju eden.png. Bantulah Agent Twilight dan Organisasi WISE menjaga perdamaian!

### Penyelesaian
***Pada node Eden***

Konfigurasi pada file `wise-2.htaccess` sebagai berikut.

```
a2enmod rewrite
ewriteEngine On
RewriteBase /
RewriteCond %{REQUEST_URI} !\beden.png\b
RewriteRule eden http://eden.wise.b10.com/public/images/eden.png$1 [L,R=301]
```

Inti dari konfigurasi tersebut adalah kita melakukan cek apakah request mengandung substring `eden`, jika ya maka akan diarahkan ke `http://eden.wise.b10.com/public/images/eden.png`.

Copy file dengan command `cp /root/wise-2.htaccess /var/www/eden.wise.b10.com/.htaccess`.

Konfigurasi pada file `default-wise-7.conf` sebagai berikut.

```
<VirtualHost *:80>
        ServerName eden.wise.b10.com
        ServerAlias www.eden.wise.b10.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/eden.wise.b10.com

        <Directory /var/www/eden.wise.b10.com>
                Options +Indexes
                AllowOverride All
        </Directory>

        <Directory /var/www/eden.wise.b10.com/public>
                Options +Indexes
        </Directory>

        Alias "/js" "/var/www/eden.wise.b10.com/public/js"

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        ErrorDocument 404 /error/404.html
</VirtualHost>
```

Copy file dengan command `cp /root/default-wise-7.conf /etc/apache2/sites-available/eden.wise.b10.com.conf`.

Restart apache `service apache2 restart`.

Testing pada node Garden dan SSS dengan command `lynx www.eden.wise.b10.com/public/images/abcedendef`, maka akan muncul tampilan berikut.

![image](https://user-images.githubusercontent.com/67154280/198266977-6d9fb7a0-c5fc-4e16-9165-6f7a8044cb26.png)

## Kendala

- Terjadi error pada nomor 17 mengenai pengaksesan substring
