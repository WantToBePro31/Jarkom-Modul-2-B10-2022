# Jarkom-Modul-2-B10-2022

## Kelompok B10
| Name                      | NRP        | 
| ------------------------- | ---------- |
| Frederick Wijayadi Susilo | 5025201111 |
| Doanda Dresta Rahma       | 5025201049 |
| Zunia Aswaroh             | 5025201058 |

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

Setelah itu, kita buat script `soal2.sh` dengan isi sebagai berikut untuk membuat website utama pada folder wise
  
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
  
Setelah itu, kita buat script `soal3.sh` dengan isi sebagai berikut untuk membuat subdomain pada folder wise
  
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
- wise-1.b10.com
  
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

Setelah itu, kita buat script `soal4.sh` dengan isi sebagai berikut untuk membuat reverse domain pada folder wise

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
![image](https://user-images.githubusercontent.com/67154280/197752672-9c9da922-7cdf-403e-b8a0-4e21feea2345.png)


### 6
> Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu **operation.wise.yyy.com** dengan alias **www.operation.wise.yyy.com** yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation.

### Penyelesaian


### 7
> Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses **strix.operation.wise.yyy.com** dengan alias **www.strix.operation.wise.yyy.com** yang mengarah ke Eden.

### Penyelesaian


### 8
> Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver **www.wise.yyy.com**. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com.

### Penyelesaian


### 9
> Setelah itu, Loid juga membutuhkan agar url **www.wise.yyy.com/index.php/home** dapat menjadi menjadi **www.wise.yyy.com/home**.

### Penyelesaian


### 10
> Setelah itu, pada subdomain **www.eden.wise.yyy.com**, Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/eden.wise.yyy.com.

### Penyelesaian


### 11
> Akan tetapi, pada folder /public, Loid ingin hanya dapat melakukan directory listing saja.

### Penyelesaian


### 12
> Tidak hanya itu, Loid juga ingin menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache.

### Penyelesaian


### 13
> Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset **www.eden.wise.yyy.com/public/js** menjadi **www.eden.wise.yyy.com/js**.

### Penyelesaian


### 14
> Loid meminta agar **www.strix.operation.wise.yyy.com** hanya bisa diakses dengan port 15000 dan port 15500.

### Penyelesaian


### 15
> dengan autentikasi username Twilight dan password opStrix dan file di /var/www/strix.operation.wise.yyy

### Penyelesaian


### 16
> dan setiap kali mengakses IP Eden akan dialihkan secara otomatis ke **www.wise.yyy.com**.

### Penyelesaian


### 17
> Karena website **www.eden.wise.yyy.com** semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring “eden” akan diarahkan menuju eden.png. Bantulah Agent Twilight dan Organisasi WISE menjaga perdamaian!

### Penyelesaian


## Kendala

- 
