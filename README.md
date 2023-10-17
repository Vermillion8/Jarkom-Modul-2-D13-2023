# Jarkom-Modul-2-D13-2023

<table>
<tbody>
  <thead>
    <tr>
      <th>Name</th>
      <th>NRP</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Thalent Athalla Razzaq</td>
      <td>5025211101</td>
    </tr>
    <tr>
      <td> Jawahirul Wildan </td>
      <td> 5025211150 </td>
  </tbody>
</table>

## Soal
1. Yudhistira akan digunakan sebagai DNS Master, Werkudara sebagai DNS Slave, Arjuna merupakan Load Balancer yang terdiri dari beberapa Web Server yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Buatlah topologi dengan pembagian sebagai berikut. Folder topologi dapat diakses pada drive berikut
2. Buatlah website utama pada node arjuna dengan akses ke **arjuna.yyy.com** dengan alias **www.arjuna.yyy.com** dengan yyy merupakan kode kelompok.
3. Dengan cara yang sama seperti soal nomor 2, buatlah website utama dengan akses ke **abimanyu.yyy.com** dan alias **www.abimanyu.yyy.com**.
4. Kemudian, karena terdapat beberapa web yang harus di-deploy, buatlah subdomain **parikesit.abimanyu.yyy.com** yang diatur DNS-nya di Yudhistira dan mengarah ke Abimanyu.
5. Buat juga reverse domain untuk domain utama. (_Abimanyu saja yang direverse_)
6. Agar dapat tetap dihubungi ketika DNS Server Yudhistira bermasalah, buat juga Werkudara sebagai DNS Slave untuk domain utama.
7. Seperti yang kita tahu karena banyak sekali informasi yang harus diterima, buatlah subdomain khusus untuk perang yaitu **baratayuda.abimanyu.yyy.com** dengan alias **www.baratayuda.abimanyu.yyy.com** yang didelegasikan dari Yudhistira ke Werkudara dengan IP menuju ke Abimanyu dalam folder Baratayuda.
8. Untuk informasi yang lebih spesifik mengenai Ranjapan Baratayuda, buatlah subdomain melalui Werkudara dengan akses **rjp.baratayuda.abimanyu.yyy.com** dengan alias **www.rjp.baratayuda.abimanyu.yyy.com** yang mengarah ke Abimanyu.
9. Arjuna merupakan suatu Load Balancer Nginx dengan tiga worker (yang juga menggunakan nginx sebagai webserver) yaitu Prabakusuma, Abimanyu, dan Wisanggeni. Lakukan deployment pada masing-masing worker.
10. Kemudian gunakan algoritma **Round Robin** untuk Load Balancer pada **Arjuna**. Gunakan _server_name_ pada soal nomor 1. Untuk melakukan pengecekan akses alamat web tersebut kemudian pastikan worker yang digunakan untuk menangani permintaan akan berganti ganti secara acak. Untuk webserver di masing-masing worker wajib berjalan di port 8001-8003. Contoh
    - _Prabakusuma:8001_
    - _Abimanyu:8002_
    - _Wisanggeni:8003_
11. Selain menggunakan Nginx, lakukan konfigurasi Apache Web Server pada worker Abimanyu dengan web server **www.abimanyu.yyy.com**. Pertama dibutuhkan web server dengan DocumentRoot pada /var/www/abimanyu.yyy
12. Setelah itu ubahlah agar url **www.abimanyu.yyy.com/index.php/home** menjadi **www.abimanyu.yyy.com/home**.
13. Selain itu, pada subdomain **www.parikesit.abimanyu.yyy.com**, DocumentRoot disimpan pada /var/www/parikesit.abimanyu.yyy
14. Pada subdomain tersebut folder /public hanya dapat melakukan _directory listing_ sedangkan pada folder /secret tidak dapat diakses (_403 Forbidden_).
15. Buatlah kustomisasi halaman error pada folder /error untuk mengganti error kode pada Apache. Error kode yang perlu diganti adalah 404 Not Found dan 403 Forbidden.
16. Buatlah suatu konfigurasi virtual host agar file asset **www.parikesit.abimanyu.yyy.com/public/js** menjadi **www.parikesit.abimanyu.yyy.com/js**
17. Agar aman, buatlah konfigurasi agar **www.rjp.baratayuda.abimanyu.yyy.com** hanya dapat diakses melalui port 14000 dan 14400.
18. Untuk mengaksesnya buatlah autentikasi username berupa “Wayang” dan password “baratayudayyy” dengan yyy merupakan kode kelompok. Letakkan DocumentRoot pada /var/www/rjp.baratayuda.abimanyu.yyy.
19. Buatlah agar setiap kali mengakses IP dari Abimanyu akan secara otomatis dialihkan ke **www.abimanyu.yyy.com (alias)**
20. Karena website **www.parikesit.abimanyu.yyy.com** semakin banyak pengunjung dan banyak gambar gambar random, maka ubahlah request gambar yang memiliki substring “abimanyu” akan diarahkan menuju abimanyu.png.

PS:

- **yyy** pada url adalah **kode kelompok anda**
- File requirement dapat diakses melalui drive berikut.

## Topologi

![alt text](./images/topologi.png)

## Setup DNS Master
> _named.conf.local_ pada **Yudhistira**

```vim
zone "arjuna.d13.com" {
	type master;
	notify yes;
	also-notify { 10.28.3.2; }; //Masukkan IP Werkudara
	allow-transfer { 10.28.3.2; }; //Masukkan IP Werkudara
	file "/etc/bind/jarkom/arjuna.d13.com";
};
zone "abimanyu.d13.com" {
	type master;
	notify yes;
	also-notify { 10.28.3.2; }; //Masukkan IP Werkudara
	allow-transfer { 10.28.3.2; }; //Masukkan IP Werkudara
	file "/etc/bind/jarkom/abimanyu.d13.com";
};
zone "3.28.10.in-addr.arpa" {
    type master;
    file "/etc/bind/jarkom/3.28.10.in-addr.arpa";
};
```

> _Arjuna Zone_ pada **Yudhistira**

```vim
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     arjuna.d13.com. root.arjuna.d13.com. (
                     2023101101         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      arjuna.d13.com.
@       IN      A       10.28.3.3	;IP ArjunaLoadBalancer
www	IN	CNAME	arjuna.d13.com.
@       IN      AAAA    ::1 
```

2. Buatlah website utama pada node arjuna dengan akses ke arjuna.yyy.com dengan alias www.arjuna.yyy.com dengan yyy merupakan kode kelompok.

    `arjuna.d13.com mempunyai IP Address 10.28.3.3 yang mengarah ke node arjuna.`

> _Abimanyu Zone_ pada **Yudhistira**

```vim
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     abimanyu.d13.com. root.abimanyu.d13.com. (
                     2023101101         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       	IN      NS      abimanyu.d13.com.
@       	IN      A       10.28.3.4	;IP AbimanyuWebServer
www		IN	CNAME	abimanyu.d13.com.
parikesit	IN	A	10.28.3.4	;IP AbimanyuWebServer
ns1		IN	A	10.28.3.4	;IP AbimanyuWebServer
baratayuda	IN	NS	ns1
@       	IN      AAAA    ::1
```

3. Dengan cara yang sama seperti soal nomor 2, buatlah website utama dengan akses ke abimanyu.yyy.com dan alias www.abimanyu.yyy.com.
  
    `abimanyu.d13.com mempunyai IP Address 10.28.3.4 yang mengarah ke node abimanyu`

4. Kemudian, karena terdapat beberapa web yang harus di-deploy, buatlah subdomain **parikesit.abimanyu.yyy.com** yang diatur DNS-nya di Yudhistira dan mengarah ke Abimanyu.

    `parikesit.abimanyu.d13.com merupakan subdomain dari abimanyu.d13.com dengan IP Address 10.28.3.4 yang mengarah ke node abimanyu`

> _Reverse Zone_ pada **Yudhistira**

```vim
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     abimanyu.d13.com. root.abimanyu.d13.com. (
                     2023101101         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
3.28.10.in-addr.arpa.	IN	NS	abimanyu.d13.com.
4			IN	PTR	abimanyu.d13.com. ;Byte ke-4 YudhistiraDNSMaster
```

5. Buat juga reverse domain untuk domain utama. (_Abimanyu saja yang direverse_)
   
   `Karena abimanyu.d13.com memiliki IP 10.28.3.4, maka reverse nya adalah 3.28.10.in-addr.arpa`

> _named.conf.options_ pada **Yudhistira**

```vim
options {
  directory "/var/cache/bind";
  //dnssec-validation auto;
  allow-query{any;};
  auth-nxdomain no;    # conform to RFC1035
  listen-on-v6 { any; };
};
```

## Setup DNS Slave
> _named.conf.local_ pada **Werkudara**

```vim
zone "arjuna.d13.com" {
	type slave;
	masters { 10.28.2.2; }; // Masukan IP	Yudhistira
	file "/var/lib/bind/arjuna.d13.com";
};

zone "abimanyu.d13.com" {
	type slave;
	masters { 10.28.2.2; }; // Masukan IP	Yudhistira
	file "/var/lib/bind/abimanyu.d13.com";
};

zone "baratayuda.abimanyu.d13.com" {
	type master;
	file "/etc/bind/delegasi/baratayuda.abimanyu.d13.com";
};
```

> _named.conf.options_ pada **Werkudara**

```vim
options {
        directory "/var/cache/bind";

	//dnssec-validation auto;
	allow-query{any;};
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```