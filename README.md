# Jarkom-Modul-3-E13-2021

Kelompok E13

|NRP           |Nama                   |
|:------------:|:---------------------:|
|05111940000090|Ihsannur Rahman Qalbi|
|05111940000003|Fairuz Hasna Rofifah|
|05111940000164|Ahmad Aunul Ma`bud|

## Soal 1

(1) Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server (1)

- Pertama, memberi **Foosha** konfigurasi sehingga dapat memberikan internet kepada **EniesLobby**, **Jipangu**, dan **Water7**
    - Network Configuration (**Foosha**)
        
        ```
        auto eth0
        iface eth0 inet dhcp
        
        auto eth1
        iface eth1 inet static
        	address 192.206.1.1
        	netmask 255.255.255.0
        
        auto eth2
        iface eth2 inet static
        	address 192.206.2.1
        	netmask 255.255.255.0
        
        auto eth3
        iface eth3 inet static
        	address 192.206.3.1
        	netmask 255.255.255.0
        ```
        
    - Agar komputer di bawahnya bisa mendapatkan internet
        
        ```
        iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.206.0.0/16
        ```
        
    - Cek nameserver: hasil → 192.168.122.1 (nameserver foosha)
        
        ```
        cat /etc/resolv.conf
        ```
        
- Konfigurasi **EniesLobby**, **Jipangu**, dan **Water7**
    - EniesLobby → DNS Server
        
        ```
        # network configuration
        iface eth0 inet static
          address 192.206.2.2
          netmask 255.255.255.0
          gateway 192.206.2.1
        ```
        
        save di `/root/.bashrc`
        
        ```
        echo nameserver 192.168.122.1 > /etc/resolv.conf
        apt-get update
        apt-get install nano
        apt-get install bind9
        ```
        
    - Jipangu → DHCP Server
        
        ```
        # network configuration
        auto eth0
        iface eth0 inet static
          address 192.206.2.4
          netmask 255.255.255.0
          gateway 192.206.2.1
        ```
        
        save  di `/root/.bashrc`
        
        ```
        echo nameserver 192.168.122.1 > /etc/resolv.conf
        apt-get update
        apt-get install nano
        apt-get install isc-dhcp-server
        dhcpd --version
        ```
        
        menentukan interface yang akan dilayani DHCP server
        
        ```
        nano /etc/default/isc-dhcp-server
        ```
        
        ubah menjadi
        
        ```
        INTERFACES="eth0"
        ```
        
    - Water7 → Proxy Server
        
        ```
        # network configuration
        auto eth0
        iface eth0 inet static
          address 192.206.2.3
          netmask 255.255.255.0
          gateway 192.206.2.1
        ```
        
        save di `/root/.bashrc`
        
        ```
        echo nameserver 192.168.122.1 > /etc/resolv.conf
        apt-get update
        apt-get install nano
        apt-get install squid
        ```
        

![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled.png)

## Soal 2

Foosha sebagai DHCP Relay

- Foosha → install DHCP relay
    
    ```
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.206.0.0/16
    apt-get update
    apt-get install nano
    apt-get install isc-dhcp-relay -y
    ```
    
- Isikan konfigutrasi seperti ini pada `/etc/default/isc-dhcp-relay`
    
    ![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%201.png)
    
    - start dhcp relay → `service isc-dhcp-relay start`

## Soal 3

Ada beberapa kriteria yang ingin dibuat oleh Luffy dan Zoro, yaitu:

- Semua client yang ada **HARUS** menggunakan konfigurasi IP dari DHCP Server.
    
    ```
    # network configuration -> loguetown, alabasta, tottoland, skypie
    auto eth0
    iface eth0 inet dhcp
    ```
    
- Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169 → `nano /etc/dhcp/dhcpd.conf` (**Jipangu**)
    
    ```
    subnet 192.206.2.0 netmask 255.255.255.0 {
        option routers 192.206.2.1;
    }
    ```
    
    ```
    subnet 192.206.1.0 netmask 255.255.255.0 {
        range 192.206.1.20 192.206.1.99;
        range 192.206.1.150 192.206.1.169;
        option routers 192.206.1.1;
        option broadcast-address 192.206.1.255;
        option domain-name-servers 192.206.2.2;
        default-lease-time 360;
        max-lease-time 7200;
    }
    ```
    
    - restart dhcp server → `service isc-dhcp-server restart` → `service isc-dhcp-server status`
    - Untuk mengecek, jalankan `ip a` pada Loguetown.
    
    ![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%202.png)
    

## Soal 4

Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50

- tambah ini di → `nano /etc/dhcp/dhcpd.conf` (Jipangu)
    
    ```
    subnet 192.206.3.0 netmask 255.255.255.0 {
        range 192.206.3.30 192.206.3.50;
        option routers 192.206.3.1;
        option broadcast-address 192.206.3.255;
        option domain-name-servers 192.206.2.2;
        default-lease-time 720;
        max-lease-time 7200;
    }
    ```
    
    - restart dhcp server → `service isc-dhcp-server restart` → `service isc-dhcp-server status`
    - Untuk mengecek, jalankan `ip a` pada TottoLand
    
    ![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%203.png)
    

## Soal 5

Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

- Tambahkan DNS Forwarder di EniesLobby
    
    `nano /etc/bind/named.conf.options` (EniesLobby)
    
    ```
    # uncomment dan tambah
    forwarders {
        192.168.122.1;
    };
    
    # comment
    // dnssec-validation auto;
    
    # tambahkan
    allow-query{any;};
    ```
    
    ![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%204.png)
    
    - restart bind9 (EniesLobby) → `service bind9 restart`

## Soal 6

Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit. *(sudah di nomor 3 dan 4)*

- Switch 1: 6 menit → 360 detik
- Switch 3: 12 menit → 720 detik
- Max: 120 menit → 7200 detik
- SS dari `cat /etc/dhcp/dhcpd.conf` (Jipangu)
    
    ![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%205.png)
    

## Soal 7

Luffy dan Zoro berencana menjadikan **Skypie** sebagai server untuk jual beli kapal yang dimilikinya dengan **alamat IP yang tetap** dengan IP [prefix IP].3.69

- cek hwadress di **Skypie → `ip a`**
    
    ![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%206.png)
    
    - hwadress **Skypie** → `c2:69:d9:d5:00:2c`
- tambah config dhcp **Jipangu →** `nano /etc/dhcp/dhcpd.conf`
    
    ```
    # tambah
    host Skypie {
        hardware ethernet c2:69:d9:d5:00:2c;
        fixed-address 192.206.3.69;
    }
    ```
    
    - restart dhcp server → `service isc-dhcp-server restart` → `service isc-dhcp-server status`
- tambah network config di **Skypie →** `nano /etc/network/interfaces`
    
    ```
    # tambah
    hwaddress ether c2:69:d9:d5:00:2c
    ```
    
    ![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%207.png)
    
- restart **Skypie,** hasilnya (`ip a`):
    
    ![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%208.png)
    

## Soal 8

**Loguetown** digunakan sebagai client **Proxy** agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi.
Pada Loguetown, proxy **harus bisa diakses** dengan nama [**jualbelikapal.yyy.com](http://jualbelikapal.yyy.com/)** dengan port yang digunakan adalah **5000**

- Buat domain mengarah ke proxy di DNS
    
    `/etc/bind/named.conf.local` di **EniesLobby**
    
    ```
    zone "jualbelikapal.e13.com" {
    	type master;
    	file "/etc/bind/jarkom/jualbelikapal.e13.com";
    };
    ```
    
- (**EniesLobby**) Buat direktori bernama jarkom, copy `db.local` dan beri nama `jualbelikapal.e13.com`
    
    ```
    mkdir /etc/bind/jarkom
    cp /etc/bind/db.local /etc/bind/jarkom/jualbelikapal.e13.com
    ```
    
- (**EniesLobby**) Tambahkan konfigurasi pada `/etc/bind/jarkom/jualbelikapal.e13.com`, seperti gambar di bawah ini
    
    ![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%209.png)
    
- Setelah domain berhasil dibuat, buat proxy. Pada **Water7**, lakukan konfigurasi pada file `/etc/squid/squid.conf`
    
    ```
    http_port 5000
    visible_hostname Water7
    
    http_access allow all
    ```
    
- Restart squid → `service squid restart`
- Pada **Loguetown**, aktifkan proxy → `export http_proxy="http://jualbelikapal.e13.com:5000"` (untuk memastikan proxy sudah aktif, cek dengan `env | grep -i proxy` — seringkali proxy tidak aktif dilakukan export menggunakan bash script), cek dengan `lynx http://its.ac.id` .
    
    ![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%2010.png)
    

## Soal 9

Agar transaksi jual beli lebih aman dan pengguna website ada dua orang, proxy ****dipasang **autentikasi user proxy dengan enkripsi MD5** dengan **dua username,** yaitu luffybelikapalyyy dengan password ****luffy_yyy **dan** zorobelikapalyyy dengan password zoro_yyy

- Pada **Water7**, jalankan:
    
    `touch /etc/squid/passwd`
    
    `htpasswd -m /etc/squid/passwd luffybelikapale13`
    
    - kemudian masukkan password → `luffy_e13`
    
    `htpasswd -m /etc/squid/passwd zorobelikapale13`
    
    - kemudian masukkan password → `zoro_e13`
- Kemudian, edit file `/etc/squid/squid.conf`.
    
    ```
    http_port 5000
    visible_hostname Water7
    
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Proxy
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    http_access allow USERS
    ```
    
- Restart squid → `service squid start`.
- `lynx [http://its.ac.id](http://its.ac.id)` sebagai **Loguetown**
    
    ![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%2011.png)
    

## Soal 10

Transaksi jual beli tidak dilakukan setiap hari, oleh karena itu akses internet dibatasi hanya dapat diakses setiap hari **Senin-Kamis pukul 07.00-11.00** dan setiap hari **Selasa-Jum’at pukul 17.00-03.00** keesokan harinya **(sampai Sabtu pukul 03.00)**

- Pada **Water7**, ubah `/etc/squid/acl.conf` menjadi:
    
    ```
    acl AVAILABLE_WORKING time MTWH 07:00-11:00
    acl AVAILABLE_WORKING1 time TWHF 17:00-24:00
    acl AVAILABLE_WORKING2 time WHFA 00:00-03:00
    ```
    
- edit `/etc/squid/squid.conf` menjadi:
    
    ```
    include /etc/squid/acl.conf
    
    http_port 5000
    visible_hostname Water7
    
    auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
    auth_param basic children 5
    auth_param basic realm Proxy
    auth_param basic credentialsttl 2 hours
    auth_param basic casesensitive on
    acl USERS proxy_auth REQUIRED
    http_access allow AVAILABLE_WORKING USERS
    http_access allow AVAILABLE_WORKING1 USERS
    http_access allow AVAILABLE_WORKING2 USERS
    
    http_access deny all
    ```
    
    - Keterangan: `http_access` mengambil argument dalam line yang sama sebagai logika `and` dan argument yang berbeda line sebagai `or`
    - Sehingga untuk mendapatkan kondisi yang diinginkan, kita membuat `http_access allow` sebanyak tiga line, setiap line akan dilakukan AVAILABLE_WORKING `and` USERS, untuk memberikan akses pada waktu yang diperbolehkan dan mengharuskan user untuk melakukan login
- restart squid → `service squid restart`
- untuk mengecek, gunakan sintaks dibawah ini untuk mengubah date time
    - `date -s "12 NOV 2021 16:00:00"` (access denied)
    - `date -s "12 NOV 2021 18:00:00"` (access accepted)
- coba akses `[http://its.ac.id](http://its.ac.id)` melalui **Loguetown** diluar waktu AVAILABLE_WORKING**,** maka akan muncul `Access Denied` seperti gambar dibawah ini
    
    ![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%2012.png)
    

## Soal 11

Agar transaksi bisa lebih fokus berjalan, maka dilakukan redirect website agar mudah mengingat website transaksi jual beli kapal. Setiap **mengakses [google.com](http://google.com/), akan diredirect menuju [super.franky.yyy.com](http://super.franky.yyy.com/)** dengan website yang sama pada soal shift modul 2. Web server [super.franky.yyy.com](http://super.franky.yyy.com/) berada pada node **Skypie**

- pada Skypie:
    - install apache2
    
        `apt-get install apache2 -y`
    
    - buat direktori **/var/www/super.franky.e13.com** dan isi dengan file hasil unzip
    
        ```jsx
        wget [https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip](https://raw.githubusercontent.com/FeinardSlim/Praktikum-Modul-2-Jarkom/main/super.franky.zip)
        unzip super.franky.zip```
        ```
    
    - buat file **/etc/apache2/sites-available/super.franky.e13.com.conf**, dengan copy dari file **000-default.conf**. Lalu tambahkan dengan:
    
        ```jsx
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.e13.com
        ServerName super.franky.e13.com
        ServerAlias www.super.franky.e13.com
        ```
    
    - aktifkan konfigurasi
    
        `a2ensite super.franky.b05.com.conf`
    
    - restart apache2
- pada EniesLobby:
    - menambahkan file **/etc/bind/kaizoku/super.franky.b05.com** , dengan isi:
    - pada file **/etc/bind/named.conf.local** , menambahkan:
    
        ```jsx
        zone "super.franky.e13.com" {
                type master;
                file "/etc/bind/kaizoku/super.franky.e13.com";
                allow-transfer { 192.206.3.69; };
        };
        ```
    
- pada Skypie:
    - tambahkan ServerName, pada file **/etc/apache2/apache2.conf ,** menandakan kika nameserver nya adalah IP dari Skypie.
    
        ```jsx
        ServerName 192.206.3.69
        ```
    
- pada Water7:
    - pastikan isi dari **/etc/nano resolv.conf** adalah nameserver IP dari EniesLobby (DNS Server).
    - pada **/etc/squid/squid.conf**
    
        ```jsx
        acl BLACKLIST dstdomain google.com
        deny_info http://super.franky.e13.com/ BLACKLIST
        http_reply_access deny BLACKLIST
        ```
    
- pada Loguetown:
    - jalankan `lynx [google.com](http://google.com)` akan teredirect ke [super.franky.e13.com](http://super.franky.e13.com) .

![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%2013.png)

## Soal 12

Saatnya berlayar! Luffy dan Zoro akhirnya memutuskan untuk berlayar untuk **mencari harta karun di [super.franky.yyy.com](http://super.franky.yyy.com/)**. Tugas pencarian dibagi menjadi dua misi, Luffy bertugas untuk **mendapatkan gambar (.png, .jpg)**, sedangkan Zoro **mendapatkan sisanya**. Karena Luffy orangnya sangat teliti untuk mencari harta karun, ketika ia berhasil mendapatkan gambar, ia mendapatkan gambar dan melihatnya dengan kecepatan **10 kbps**

- pada Water7:
    - pada file **/etc/squid/acl-bandwidth.conf** , tambahkan:
    
        ```jsx
        acl download url_regex -i \.jpg$ \.png$
        
        auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
        
        acl luffy proxy_auth luffybelikapalb05
        acl zoro proxy_auth zorobelikapalb05
        
        delay_pools 2
        delay_class 1 1
        delay_parameters 1 1250/1250
        delay_access 1 allow luffy download
        delay_access 1 deny zoro
        delay_access 1 deny all
        ```
    
    - include kan **acl-bandwidth.conf** pada file **squid**
    
        ```jsx
        include /etc/squid/acl-bandwidth.conf
        ```
    
    - uji coba untuk melakukan download ketika menjadi luffy, kecepatan akan dibatasi untuk format .png dan .jpg .

![Screenshot (120).png](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Screenshot_(120).png)

## Soal 13

Sedangkan, Zoro yang sangat bersemangat untuk mencari harta karun, sehingga kecepatan kapal Zoro tidak dibatasi ketika sudah mendapatkan harta yang diinginkannya

- pada Water7:
    - tambahkan isi file **/etc/squid/acl-bandwidth.conf** dengan:
    
        ```jsx
        delay_class 2 1
        delay_parameters 2 -1/-1
        delay_access 2 allow zoro
        delay_access 2 deny luffy
        delay_access 2 deny all
        ```
    
    - uji coba untuk melakukan download ketika menjadi zoro, maka kecepatan tidak dibatasi.

![Untitled](Jarkom-Modul-3-E13-2021%20da0913b782464fd0b9ec3dd4b9d144c3/Untitled%2014.png)

## Kendala

1. Mengalami kendala pada nomor 10, [its.ac.id](http://its.ac.id) tidak bisa diakses meski pada waktu yang diperbolehkan. Akhirnya diperbaiki syntax acl-nya, variable acl nya dipisah di http_access.
2. Mengalami kendala pada nomor 11, pada semua gambar semua kecepatannya dibatasi. Akhirnya dapat yang berupa .jpg dan .png saja dengan memperbaiki acl-squid.conf

## Pembagian Tugas

|Nama|Soal|
|:------------:|:---------------------:|
|Fairuz Hasna Rofifah|1-5|
|Ahmad Aunul Ma`bud|6-10|
|Ihsannur Rahman Qalbi|10-13|