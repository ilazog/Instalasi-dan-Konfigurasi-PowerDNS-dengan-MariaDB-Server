Instalasi dan Konfigurasi PowerDNS dengan MariaDB Server
===============

Repositori untuk melakukan instalasi dan konfigurasi PowerDNS dengan menggunakan database server MariaDB

## Task
Instalasi dan Konfigurasi
* PowerDNS
* Database Server MariaDB 10.1.44
* Glue Record Domain

Ketentuan pengerjaan:
* Menggunakan VPS dengan OS centos7
* Menggunakan domain utama (domain.tld)
* Menggunakan DNS Server Pdns
* Menggunakan Database Server MariaDB 10.1.44

## Tentang PowerDNS
#### 1. Pengertian PowerDNS
PowerDNS adalah DNS Server yang bisa berjalan di banyak turunan Linux/Unix. aplikasi ini juga bisa dikonfigurasi dengan berbagai macam backend termasuk BIND, Database Relational atau Load balancer. Kemampuan lainnya adalah aplikasi ini juga bisa diatur untuk menjadi DNS recursor yang berjalan sebagai service atau proses yang berbeda (pdns-recursor).

#### 2. Tiga Produk Open Source dalam PowerDNS
##### A. PowerDNS Authoritative Server
PowerDNS Authoritative Server adalah satu-satunya solusi yang memungkinkan layanan DNS otoritatif dari semua database utama, tetapi tidak terbatas pada MySQL, PostgreSQL, SQLite3, Oracle, Sybase, Microsoft SQL Server, LDAP dan file teks biasa.

Record DNS juga dapat ditulis menggunakan berbagai bahasa (scripting) seperti Lua, Java, Perl, Python, Ruby, C dan C ++. Skrip semacam itu dapat digunakan untuk dynamic redirection,  (spam) filtering atau real time intervention.

Selain itu, PowerDNS Authoritative Server adalah implementasi DNSSEC terkemuka, menampung sebagian besar semua domain DNSSEC di seluruh dunia. Authoritative Server memiliki setidaknya 30% dari semua nama domain di Eropa, dan sekitar 90% dari semua domain DNSSEC di Eropa.

##### B. DNSdist
DNSdist adalah fitur untuk loadbalancer yang berguna untuk DNS, DoS dan penyalahgunaan lainnya. Tujuannya adalah untuk mengarahkan lalu lintas ke server terbaik, memberikan kinerja terbaik untuk pengguna yang sah sambil mengubah atau memblokir lalu lintas yang mencurigakan. Dnsdist digunakan untuk melindungi dan mengoptimalkan lalu lintas DNS dari ratusan juta pelanggan internet.

##### C. Recursor
PowerDNS Recursor adalah name server dengan penyelesaian tinggi, berkinerja tinggi yang mendukung resolusi DNS yang setidaknya lebih dari seratus juta pelanggan. Memanfaatkan banyak prosesor dan mendukung kemampuan scripting yang sama dari Server Resmi, Recursor memberikan kinerja terbaik sambil mempertahankan fleksibilitas yang dibutuhkan untuk penyebaran DNS modern.

## Instalasi dan Konfigurasi Pdns dan MariaDB 10.1.44
#### Step 1: Instalasi Database Server MariaDB 10.1.44

Untuk melakukan instalasi MariaDB 10.1.44 yang pertama adalah melakukan remote ke IP VPS dengan menggunakan SSH

> ```$ ssh root@ipaddress```

Setelah berhasil login ke VPS, lakukan pembaharuan paket/repository dari system operasi Centos7 dengan perintah sebagai berikut:

> ```# yum -y update```

Setelah melakukan update system selanjutnya lakukan install epel-release

> ```# yum install epel-release -y```

Setelah melakukan install epel-release selanjutnya lakukan instalasi database server mariaDB 10.1.44 dan langkah pertama ialah menambahkan repository untuk mariaDB.

> ```# vi /etc/yum.repos.d/mariadb.repo```

Masukan perintah berikut:

```
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

Setelah menambahkan repo mariaDB, lakukan instalasi mariaDB dengan perintah berikut:

> ```# yum install mariadb-server```

Selanjutnya lakukan enable direktori dan file database mariaDB, berikut perintahnya:

> ```# systemctl enable mariadb```

Setelah direktori dan file database mariaDB dienable, jalankan service mariaDB dengan perintah berikut:

> ```# systemctl start mariadb```

Apabila paket database server telah selesai diinstall pastikan service mariaDB berjalan dengan status Running.

```
# systemctl status mariadb
● mariadb.service - MariaDB 10.1.44 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/mariadb.service.d
           └─migrated-from-my.cnf-settings.conf
   Active: active (running) since Sat 2020-03-07 20:09:38 WIB; 22h ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
  Process: 29925 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
  Process: 29885 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`/usr/bin/galera_recovery`; [ $? -eq 0 ]   && systemctl set-environment _WSREP_START_POSITION=$VAR || exit 1 (code=exited, status=0/SUCCESS)
  Process: 29883 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
 Main PID: 29897 (mysqld)
   Status: "Taking your SQL requests now..."
   CGroup: /system.slice/mariadb.service
           └─29897 /usr/sbin/mysqld
```

#### Step 2: Secure mariaDB server and Configure Database

Setelah melakukan instalasi mariaDB Server selanjutnya kita harus mengamankan databese server dengan cara menambahkan password login saat mengakses mariaDB server.

> ```# mysql_secure_installation```

Nantinya kita akan melakukan perubahan password untuk root database server, pilih `Y` dan masukan Password baru yang kuat.

```
Set root password? [Y/n] Y
New password: 
Re-enter new password: 
```

Apabila ada yang lain silakan klik `Y`

```
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```

Setelah itu kita coba untuk melakukan login dengan password baru yang telah dibuat dengan perintah berikut:

```
# mysql -u root -p
Input Password
```

Apabila telah login selanjutnya buat database, user dan password untuk service PowerDNS.

```
MariaDB [(none)]> Create database testpdns;
MariaDB [(none)]> grant all privileges on testdns.* to pdns@localhost identified by 'pdnspassword';
MariaDB [(none)]> flush privileges; 
```

Setelah itu pilih database testpdns;

```
MariaDB [(none)]>use testpdns;
MariaDB [testpdns]>
```

Buat table baru untuk menyimpan record pdns pada database testpdns.

```
MariaDB [testpdns]> CREATE TABLE domains (
  id                    INT AUTO_INCREMENT,
  name                  VARCHAR(255) NOT NULL,
  master                VARCHAR(128) DEFAULT NULL,
  last_check            INT DEFAULT NULL,
  type                  VARCHAR(6) NOT NULL,
  notified_serial       INT UNSIGNED DEFAULT NULL,
  account               VARCHAR(40) CHARACTER SET 'utf8' DEFAULT NULL,
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'latin1';
```

```
MariaDB [testpdns]> CREATE UNIQUE INDEX name_index ON domains(name);
```

```
MariaDB [testpdns]>CREATE TABLE records (
  id                    BIGINT AUTO_INCREMENT,
  domain_id             INT DEFAULT NULL,
  name                  VARCHAR(255) DEFAULT NULL,
  type                  VARCHAR(10) DEFAULT NULL,
  content               VARCHAR(64000) DEFAULT NULL,
  ttl                   INT DEFAULT NULL,
  prio                  INT DEFAULT NULL,
  disabled              TINYINT(1) DEFAULT 0,
  ordername             VARCHAR(255) BINARY DEFAULT NULL,
  auth                  TINYINT(1) DEFAULT 1,
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'latin1';
```

```
MariaDB [testpdns]> CREATE INDEX nametype_index ON records(name,type);
```

```
MariaDB [testpdns]> CREATE INDEX domain_id ON records(domain_id);
```

```
MariaDB [testpdns]> CREATE INDEX ordername ON records (ordername);
```

```
MariaDB [testpdns]> CREATE TABLE supermasters (
  ip                    VARCHAR(64) NOT NULL,
  nameserver            VARCHAR(255) NOT NULL,
  account               VARCHAR(40) CHARACTER SET 'utf8' NOT NULL,
  PRIMARY KEY (ip, nameserver)
) Engine=InnoDB CHARACTER SET 'latin1';
```

```
MariaDB [testpdns]> CREATE TABLE comments (
  id                    INT AUTO_INCREMENT,
  domain_id             INT NOT NULL,
  name                  VARCHAR(255) NOT NULL,
  type                  VARCHAR(10) NOT NULL,
  modified_at           INT NOT NULL,
  account               VARCHAR(40) CHARACTER SET 'utf8' DEFAULT NULL,
  comment               TEXT CHARACTER SET 'utf8' NOT NULL,
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'latin1';
```

```
MariaDB [testpdns]> CREATE INDEX comments_name_type_idx ON comments (name, type);
```

```
MariaDB [testpdns]> CREATE INDEX comments_order_idx ON comments (domain_id, modified_at);
```

```
MariaDB [testpdns]> CREATE TABLE domainmetadata (
  id                    INT AUTO_INCREMENT,
  domain_id             INT NOT NULL,
  kind                  VARCHAR(32),
  content               TEXT,
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'latin1';
```

```
MariaDB [testpdns]> CREATE INDEX domainmetadata_idx ON domainmetadata (domain_id, kind);
```

```
MariaDB [testpdns]> CREATE TABLE cryptokeys (
  id                    INT AUTO_INCREMENT,
  domain_id             INT NOT NULL,
  flags                 INT NOT NULL,
  active                BOOL,
  published             BOOL DEFAULT 1,
  content               TEXT,
  PRIMARY KEY(id)
) Engine=InnoDB CHARACTER SET 'latin1';
```

```
MariaDB [testpdns]>CREATE INDEX domainidindex ON cryptokeys(domain_id);
```

```
MariaDB [testpdns]> CREATE TABLE tsigkeys (
  id                    INT AUTO_INCREMENT,
  name                  VARCHAR(255),
  algorithm             VARCHAR(50),
  secret                VARCHAR(255),
  PRIMARY KEY (id)
) Engine=InnoDB CHARACTER SET 'latin1';
```

```
MariaDB [testpdns]> CREATE UNIQUE INDEX namealgoindex ON tsigkeys(name, algorithm);
```

Tambahkan perintah berikut untuk membuat kunci untuk setiap table diatas.

```
MariaDB [testpdns]> ALTER TABLE records ADD CONSTRAINT `records_domain_id_ibfk` FOREIGN KEY (`domain_id`) REFERENCES `domains` (`id`) ON DELETE CASCADE ON UPDATE CASCADE;
MariaDB [testpdns]> ALTER TABLE comments ADD CONSTRAINT `comments_domain_id_ibfk` FOREIGN KEY (`domain_id`) REFERENCES `domains` (`id`) ON DELETE CASCADE ON UPDATE CASCADE;
MariaDB [testpdns]> ALTER TABLE domainmetadata ADD CONSTRAINT `domainmetadata_domain_id_ibfk` FOREIGN KEY (`domain_id`) REFERENCES `domains` (`id`) ON DELETE CASCADE ON UPDATE CASCADE;
MariaDB [testpdns]> ALTER TABLE cryptokeys ADD CONSTRAINT `cryptokeys_domain_id_ibfk` FOREIGN KEY (`domain_id`) REFERENCES `domains` (`id`) ON DELETE CASCADE ON UPDATE CASCADE;
```

Lihat hasil dari penambahan table dengan perintah berikut:

```
MariaDB [testpdns]> show tables;
+--------------------+
| Tables_in_testpdns |
+--------------------+
| comments           |
| cryptokeys         |
| domainmetadata     |
| domains            |
| records            |
| supermasters       |
| tsigkeys           |
+--------------------+
7 rows in set (0.00 sec)
```

#### Step 3: Instalasi dan konfigurasi PowerDNS

Setelah membuat database dan table untuk service PowerDNS selanjutnya lakukan instalasi PowerDNS.

> ```# yum -y install pdns pdns-backend-mysql bind-utils```

Lakukan konfigurasi file `pdns.conf`

```
# cd /etc/pdns/
pdns#  vi pdns.conf
```

Rubah dan tambahakan perintah berikut.

```
#i################################
# launch        Which backends to launch and order to query them in
#
# launch=bind (kasih tanda pagar untuk nonaktifkan)
launch=gmysql
gmysql-host=localhost
gmysql-user=pdns (user database)
gmysql-password=y4m4h475 (Password database)
gmysql-dbname=testpdns (nama database)
```

Save dan Close kemudian enable serta aktifkan service PowerDNS.

```
# systemctl enable pdns
# systemctl start pdns
```

Pastikan service PowerDNS berjalan

```
# systemctl status pdns
● pdns.service - PowerDNS Authoritative Server
   Loaded: loaded (/usr/lib/systemd/system/pdns.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2020-03-15 13:08:11 WIB; 1h 21min ago
     Docs: man:pdns_server(1)
           man:pdns_control(1)
           https://doc.powerdns.com
 Main PID: 2796 (pdns_server)
   CGroup: /system.slice/pdns.service
           └─2796 /usr/sbin/pdns_server --guardian=no --daemon=no --disable-s...
Mar 15 13:08:11 pdns.padiakse.my.id pdns_server[2796]: UDP server bound to 0....
Mar 15 13:08:11 pdns.padiakse.my.id pdns_server[2796]: UDPv6 server bound to ...
Mar 15 13:08:11 pdns.padiakse.my.id pdns_server[2796]: TCP server bound to 0....
Mar 15 13:08:11 pdns.padiakse.my.id pdns_server[2796]: TCPv6 server bound to ...
Mar 15 13:08:11 pdns.padiakse.my.id pdns_server[2796]: PowerDNS Authoritative...
Mar 15 13:08:11 pdns.padiakse.my.id pdns_server[2796]: Using 64-bits mode. Bu...
Mar 15 13:08:11 pdns.padiakse.my.id pdns_server[2796]: PowerDNS comes with AB...
Mar 15 13:08:11 pdns.padiakse.my.id pdns_server[2796]: Creating backend conne...
Mar 15 13:08:11 pdns.padiakse.my.id pdns_server[2796]: About to create 3 back...
Mar 15 13:08:11 pdns.padiakse.my.id pdns_server[2796]: Done launching threads...
Hint: Some lines were ellipsized, use -l to show in full.
```

Setelah aktif pastikan service Pdns telah terhubung dengan database yang dibuat tadi dengan cara berikut.

Matikan service PowerDNS

```
# systemctl stop pdns
```

Jalankan perintah berikut:

```
# /usr/sbin/pdns_server --daemon=no --guardian=no --loglevel=9
Mar 15 14:30:36 Reading random entropy from '/dev/urandom'
Mar 15 14:30:36 Loading '/usr/lib64/pdns/libgmysqlbackend.so'
Mar 15 14:30:36 [gmysqlbackend] This is the gmysql backend version 4.1.11 reporting
Mar 15 14:30:36 This is a standalone pdns
Mar 15 14:30:36 Listening on controlsocket in '/var/run/pdns.controlsocket'
Mar 15 14:30:36 UDP server bound to 0.0.0.0:53
Mar 15 14:30:36 UDPv6 server bound to [::]:53
Mar 15 14:30:36 TCP server bound to 0.0.0.0:53
Mar 15 14:30:36 TCPv6 server bound to [::]:53
Mar 15 14:30:36 PowerDNS Authoritative Server 4.1.11 (C) 2001-2018 PowerDNS.COM BV
Mar 15 14:30:36 Using 64-bits mode. Built using gcc 4.8.5 20150623 (Red Hat 4.8.5-36).
Mar 15 14:30:36 PowerDNS comes with ABSOLUTELY NO WARRANTY. This is free software, and you are welcome to redistribute it according to the terms of the GPL version 2.
Mar 15 14:30:36 Set effective group id to 993
Mar 15 14:30:36 Set effective user id to 996
Mar 15 14:30:36 Creating backend connection for TCP
Mar 15 14:30:36 gmysql Connection successful. Connected to database 'testpdns' on 'localhost'.
Mar 15 14:30:36 About to create 3 backend threads for UDP
Mar 15 14:30:36 gmysql Connection successful. Connected to database 'testpdns' on 'localhost'.
Mar 15 14:30:36 gmysql Connection successful. Connected to database 'testpdns' on 'localhost'.
Mar 15 14:30:36 gmysql Connection successful. Connected to database 'testpdns' on 'localhost'.
Mar 15 14:30:36 Done launching threads, ready to distribute questions
```

Apabila service Pdns telah terhubung dengan database selanjutnya kita close dan jalankan service PowerDNS kembali.

#### Step 4: Add Glue Record and Add Record

Untuk menambahkan glue record pada domain.tld, silakan menghubungi pihak registrar domain tersebut dan dalam case ini kami menggunakan domain dari registrar Domain Cloud.

Cara lihat registrar domain

```
$ whois domain.tld
Sponsoring Registrar PANDI ID:garuda
Sponsoring Registrar Organization:Domain Cloud
Sponsoring Registrar City:Jakarta Selatan
Sponsoring Registrar State/Province:Jakarta
Sponsoring Registrar Postal Code:12870
Sponsoring Registrar Country:ID
Sponsoring Registrar Phone:02129682828
Sponsoring Registrar Contact Email:registrar@isi.co.id
```

Masuk pada portal domain dan pilih bagian name server masukan nama name server dan Ip Address kemudian save changes.


<img src="https://manan.s3-id-jkt-1.kilatstorage.id/gambar/1.png" width="500">


Selanjutnya tambahkan record DNS untuk domain tersebut pada table domains dan records

``` 
MariaDB [testpdns]> INSERT INTO domains (name, type) values ('padiakse.my.id', 'NATIVE');
MariaDB [testpdns]> INSERT INTO records (domain_id, name, content, type,ttl,prio) VALUES (1,'padiakse.my.id','padiakse.my.id root.padiakse.my.id 1 10380 3600 604800 3600','SOA',86400,NULL);
MariaDB [testpdns]> INSERT INTO records (domain_id, name, content, type,ttl,prio) VALUES (1,'padiakse.my.id','ns1.padiakse.my.id','NS',86400,NULL);
MariaDB [testpdns]> INSERT INTO records (domain_id, name, content, type,ttl,prio) VALUES (1,'padiakse.my.id','ns2.padiakse.my.id','NS',86400,NULL);
MariaDB [testpdns]> INSERT INTO records (domain_id, name, content, type,ttl,prio) VALUES (1,'ns1.padiakse.my.id','103.23.20.70','A',3600,NULL);
MariaDB [testpdns]> INSERT INTO records (domain_id, name, content, type,ttl,prio) VALUES (1,'ns2.padiakse.my.id','103.23.20.70','A',3600,NULL);
MariaDB [testpdns]> INSERT INTO records (domain_id, name, content, type,ttl,prio) VALUES (1,'padiakse.my.id','103.23.20.70','A',3600,NULL);
```

Berikut isi dari table records dan domain.

```
MariaDB [testpdns]> select *from domains;
+----+----------------+--------+------------+--------+-----------------+---------+
| id | name           | master | last_check | type   | notified_serial | account |
+----+----------------+--------+------------+--------+-----------------+---------+
|  1 | padiakse.my.id | NULL   |       NULL | NATIVE |            NULL | NULL    |
+----+----------------+--------+------------+--------+-----------------+---------+
1 row in set (0.00 sec)
```


```
MariaDB [testpdns]> select *from records;
+----+-----------+---------------------+------+-------------------------------------------------------------+-------+------+----------+-----------+------+
| id | domain_id | name                | type | content                                                     | ttl   | prio | disabled | ordername | auth |
+----+-----------+---------------------+------+-------------------------------------------------------------+-------+------+----------+-----------+------+
|  1 |         1 | padiakse.my.id      | SOA  | padiakse.my.id root.padiakse.my.id 1 10380 3600 604800 3600 | 86400 | NULL |        0 | NULL      |    1 |
|  2 |         1 | padiakse.my.id      | NS   | ns1.padiakse.my.id                                          | 86400 | NULL |        0 | NULL      |    1 |
|  3 |         1 | padiakse.my.id      | NS   | ns2.padiakse.my.id                                          | 86400 | NULL |        0 | NULL      |    1 |
|  4 |         1 | ns1.padiakse.my.id  | A    | 103.23.20.70                                                |  3600 | NULL |        0 | NULL      |    1 |
|  5 |         1 | ns2.padiakse.my.id  | A    | 103.23.20.70                                                |  3600 | NULL |        0 | NULL      |    1 |
|  6 |         1 | padiakse.my.id      | A    | 103.23.20.70                                                |  3600 | NULL |        0 | NULL      |    1 |
+----+-----------+---------------------+------+-------------------------------------------------------------+-------+------+----------+-----------+------+
6 rows in set (0.00 sec)
```


Setelah lakukan perubahan nameserver dan penambahan record pada domain, maka akan menyebabkan domain mengalami proses propagasi yang mana akan membutuhkan waktu maksimal hingga 2x24 jam untuk dapat resolv sepenuhnya keseluruh ISP dan cepat atau lambatnya proses propagasi tergantung ISP yang anda gunakan.

Untuk pengecekan dapat dilakukan dengan cara berikut:

* Cek Name Server

```
$ whois padiakse.my.id |grep Server
Name Server:ns1.padiakse.my.id
Name Server:ns2.padiakse.my.id
```

* Cek Name Server telah resolve

```
$ dig @ns1.padiakse.my.id padiakse.my.id +short
103.23.20.70
$ dig @ns2.padiakse.my.id padiakse.my.id +short
103.23.20.70
```

* Cek domain telah resolve 

```
$ dig padiakse.my.id +short
103.23.20.70
```

#### Step 4: Pointing domain at PowerDNS

Langkah pertama tentukan alamat IP tujuan poiting domain, apabila sudah ditentukan selanjutnya melakukan penambahan pada table records yang ada pada database testpdns.

```
MariaDB [testpdns]> INSERT INTO records (domain_id, name, content, type,ttl,prio) VALUES (1,'wp1.padiakse.my.id','117.53.44.143','A',3600,NULL);
MariaDB [testpdns]> INSERT INTO records (domain_id, name, content, type,ttl,prio) VALUES (1,'pdns.padiakse.my.id','103.23.22.251','A',3600,NULL);
```

Apabila telah ditambahkan berikut isi table records pada database testpdns.

```
MariaDB [testpdns]> select *from records;
+----+-----------+---------------------+------+-------------------------------------------------------------+-------+------+----------+-----------+------+
| id | domain_id | name                | type | content                                                     | ttl   | prio | disabled | ordername | auth |
+----+-----------+---------------------+------+-------------------------------------------------------------+-------+------+----------+-----------+------+
|  1 |         1 | padiakse.my.id      | SOA  | padiakse.my.id root.padiakse.my.id 1 10380 3600 604800 3600 | 86400 | NULL |        0 | NULL      |    1 |
|  2 |         1 | padiakse.my.id      | NS   | ns1.padiakse.my.id                                          | 86400 | NULL |        0 | NULL      |    1 |
|  3 |         1 | padiakse.my.id      | NS   | ns2.padiakse.my.id                                          | 86400 | NULL |        0 | NULL      |    1 |
|  4 |         1 | ns1.padiakse.my.id  | A    | 103.23.20.70                                                |  3600 | NULL |        0 | NULL      |    1 |
|  5 |         1 | ns2.padiakse.my.id  | A    | 103.23.20.70                                                |  3600 | NULL |        0 | NULL      |    1 |
|  6 |         1 | padiakse.my.id      | A    | 103.23.20.70                                                |  3600 | NULL |        0 | NULL      |    1 |
|  7 |         1 | wp1.padiakse.my.id  | A    | 117.53.44.143                                               |  3600 | NULL |        0 | NULL      |    1 |
|  8 |         1 | pdns.padiakse.my.id | A    | 103.23.22.251                                               |  3600 | NULL |        0 | NULL      |    1 |
+----+-----------+---------------------+------+-------------------------------------------------------------+-------+------+----------+-----------+------+
8 rows in set (0.00 sec)
```

Tunggu hingga resolve dan berikut hasil pengecekannya.

```
$ dig wp1.padiakse.my.id +short
117.53.44.143
```

```
$ dig pdns.padiakse.my.id  +short
103.23.22.251
```

Apabila diakses dengan menggunakan web browser berikut tampilan `wp1.padiakse.my.id`

<img src="https://manan.s3-id-jkt-1.kilatstorage.id/gambar/cake1.png" width="500">

Apabila diakses dengan menggunakan web browser berikut tampilan `pdns.padiakse.my.id`

<img src="https://manan.s3-id-jkt-1.kilatstorage.id/gambar/cake2.png" width="500">


*_Sekian dan Terima Kasih_*

