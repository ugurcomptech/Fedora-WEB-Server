# Fedora-WEB-Server

Apache, web sitelerini barındırmak ve yönetmek için güçlü ve yaygın olarak kullanılan bir web sunucu yazılımıdır. Fedora üzerinde Apache kurulumu ve yapılandırması genellikle paket yöneticisi aracılığıyla kolayca gerçekleştirilebilir. Biz Apache servisiyle bu işlemleri yapacağız.

## Kurulum

Fedora serverımıza apacheyi indirelim:

```
dnf install httpd -y
```

Servisimizi başlatalım:

```
systemctl enable --now httpd
```

Servisimizin durumunu kontrol edelim:

```
systemctl status httpd
```

Bir sorun yoksa gerekli Firewall izinlerini verelim:

```
firewall-cmd --add-service=http --perm
firewall-cmd --reload
```

Eğer bu işlemde `FirewallD is not running` hatası alırsanız aşağıdaki komutları yazınız:

```
[root@fedoraserver master]# sudo systemctl start firewalld
[root@fedoraserver master]# sudo systemctl enable firewalld
```

Herhangi bir problemle karşılaşmadıysanız taraycınızdan sunucunuzun IP adresini yazarak test page sayfasını görebilirsiniz.

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/6ac75531-72c0-4942-93dc-70eac29915aa)

Apache Dizinleri (Web Dosyaları)

Görmüş olduğumuz test page sayfasının dosya yoluna gidiyoruz

```
root@master:/home/master# nano /var/www/html/index.html
```

İçerisini açtığınızda test sayfasının HTML kodlarını göreceksiniz. HTML bilmemize gerek yok biz sadece Web dosyalarını değiştireceğiz. Googleye `basic html` yazarsanız githubda bazı html sayfaları bulabilirsiniz. Eğer kendi repolarınızda varsa bunlarıda kullanabilirsiniz. Ben githubdan bir tane buldum sizlerde bunu kullanabilirsiniz.

Serverımıza gerekli html dosyalarını indiriyoruz:

```
git clone https://github.com/designmodo/html-website-templates.git
```

Repo indirildikten sonra gerekli dizine gidiyoruz ve aşağıdaki komudu yazıyoruz:

```
[root@fedoraserver Landing Page Website for App]# cp -r * /var/www/html/
cp: overwrite '/var/www/html/index.html'? yes
```

/var/www/html/ içerisindeki index.html dosyasının üzerine yazacaktır.


Tarayıcımızda test edelim:

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/eaaf7c86-1a2b-49f5-8fa0-b78b8f5381d7)


## Virtualhost

Apache web sunucu yazılımı, birden fazla web sitesini aynı fiziksel sunucu üzerinde barındırmak için "Virtual Host" özelliğini destekler. Virtual Host, aynı IP adresi veya portu kullanarak farklı alan adlarına veya alt alan adlarına hizmet verilmesini sağlar.

İlk önce /var/www/ altında bir tane dosya oluşturacağız.

```
 mkdir -p /var/www/sirket1.com/dosyalar
```

sirket1.com'un olduğunu belirttik ve onun altına bir klasör daha açtık. Şimdi gerekli izinleri vereceğiz:

```
chown -R apache: /var/www/sirket1.com/
chown -R 755 /var/www/sirket1.com/
```

Gerekli izinleri halettikten sonra şimdi index.html dosyası oluşturacağız:

```
nano /var/www/sirket1.com/dosyalar/index.html
```

Dosyamızı oluşturduktan sonra sirket1.com.conf adında bir config dosyası oluşturacağız:

```
nano /etc/httpd/conf.d/sirket1.com.conf
```

Conf dosyamızın içerisi boş gelecektir. Ubuntu Serverımızda yaptığımız tanımların aynısını burayada yapabiliriz:


```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/sirket1.com/dosyalar

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog /var/www/sirket1.com/error.log
        CustomLog /var/www/sirket1.com/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

Config dosyamızı test etmek için `apachectl configtest` yazıyoruz. 

`systemctl restart httpd` komutuyla servisimizi yeniden başlatıyoruz. Bu kısımda SELinux ile ilgili bazı hatalar alacağız.

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/ba46f789-18fe-4a1c-aeb0-9786cf72b9a7)


İlgili hatayı gidermek için aşağıda vermiş olduğum komutları tek tek yazınız:

```
semanage fcontext -a -t httpd_sys_rw_content_t 'sirket1.com'

setsebool -P httpd_unified 1

ausearch -c 'httpd' --raw | audit2allow -M my-httpd

semodule -X 300 -i my-httpd.pp
```

Gerekli komutları yazdıktan sonra `systemctl restart httpd` komutuyla servisimizi yeniden başlatıyoruz 

Sunucmuzun IP adresini  tarayıcıya yazarak test ediyoruz:

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/cab90e27-533f-4426-ac30-7ced0087f8e5)


## Aynı Portta Çoklu Site Yayınlama

Şimdi bunu test etmek için ikinci bir site daha oluşturacağız.

```
 mkdir -p /var/www/sirket2.com/dosyalar
```

Gerekli izinlei verelim:

```
chown -R apache: /var/www/sirket2.com/dosyalar
chown -R 755 /var/www/sirket2.com/dosyalar
```

İndex.html dosyamızı açalım ve içine istediğimizi yazalım

```
nano /var/www/sirket2.com/dosyalar/index.html
```

Şimdi `.conf` dosyalarımızı yapılandıralım:



sirket1.conf

```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/sirket1.com/dosyalar
        ServerName   ugur.com
        ServerAlias  www.ugur.com

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog /var/www/sirket1.com/error.log
        CustomLog /var/www/sirket1.com/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

```


sirket2.conf


```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/sirket2.com/dosyalar
        ServerName   ugur1.com
        ServerAlias  www.ugur1.com

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog /var/www/sirket2.com/error.log
        CustomLog /var/www/sirket2.com/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

 Şimdi apache servisimizi `systemctl restart httpd` yazarak yeniden başlatalım.

 uan gerekli yapılandırmaları yaptık. Şimdi bunu ister DNS serverımıza tanıtabiliriz, istersek de kendi localimizdeki hosts dosyasına tanıtabiliriz. Daha kolay olması açısından Windows bilgisayarımızdaki host dosyasına tanıtalım.

`C:\Windows\System32\drivers\etc` dosya yoluna gidip `hosts` dosyasını masaüstüne yapıştırın. Dosyası açtıktan sonra aşağıdaki tanımları yapın ve kaydedin. Kaydettikten sonra masaüstündeki dosyası tekrardan `C:\Windows\System32\drivers\etc` dosya yoluna yapıştırın.


```
172.16.1.30	ugur.com
172.16.1.30	www.ugur.com

172.16.1.30	ugur1.com
172.16.1.30	www.ugur1.com
```


Bu ayarları yaptıktan sonra tarayıcınızı açıp test edebilirsiniz.

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/35fe1fb4-495a-40ae-914a-24c16aaf4523)


![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/aee7090d-dee5-47d1-8c23-b2a892962da2)



## PHP MYSQL

MYSQL server ve PHP Apache modüllerini yüklüyoruz:


```
dnf install mysql-server php php-mysqlnd -y
```

Servisimizi `systemctl restart httpd` yazarak yeniden başlatıyoruz.

### PHP

Artık serverımızda PHP kodlarıda çalışacaktır. Bunu şöyle test edebiliriz.

`nano /var/www/sirket1.com/dosyalar/info.php` yazarak info.php dosyasını oluşturup içerisine aşağıdaki komutu yazabiliriz.

```
<?php
phpinfo ();
?>
```

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/146871f5-c097-40bc-9a59-bb27d944c544)


### MYSQL


MySQL veritabanı sunucunuzu güvenli bir şekilde yapılandıralım:

```
mysql_secure_installation
```

Bu komutu yazdıktan sonra ihtiyaçlarınıza göre gelen sorulara Yes veya No Diyebilirsiniz.

DB hataları almamak için aşağıdaki komutları yazabilirsiniz:

```
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

`mysql_secure_installation` yazdıktan sonra şifre belirleyebilirsiniz. Eğer şifre belirleyemediyseniz aşağıdaki komutu yazabilirsiniz.

```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '1234-Aaa!';
```

Aşağıdaki komutu yazarak `mysql` girişi yapabilirsiniz.

```
mysql -u root -p
```

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/92c9c0dd-ff7b-42ef-babc-cd06f2bec5f4)

Bu şekilde şifrenizi girerek giriş yapabilirsiniz.

```
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0,001 sec)
MariaDB [(none)]>
```

Aşağıdaki komutları yazarak tabloları görüntüleyebilirsiniz.

```
MariaDB [(none)]> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [mysql]> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| column_stats              |
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| global_priv               |
| gtid_slave_pos            |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| index_stats               |
| innodb_index_stats        |
| innodb_table_stats        |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| roles_mapping             |
| servers                   |
| slow_log                  |
| table_stats               |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| transaction_registry      |
| user                      |
+---------------------------+
31 rows in set (0,001 sec)

MariaDB [mysql]>
```

Tabloların içeriğini görüntülemek için aşağıdaki sorguyu yazabilirsiniz.

```
MariaDB [mysql]> SELECT user,host FROM user;
+-------------+-----------+
| User        | Host      |
+-------------+-----------+
| mariadb.sys | localhost |
| mysql       | localhost |
| root        | localhost |
+-------------+-----------+
3 rows in set (0,001 sec)

MariaDB [mysql]>
```


## PhpMyAdmin

PhpMyAdmini indirelim.


```
dnf install mysql-server php php-mysqlnd -y
```

Gerekli kurulumlar yapıldıktan sonra bir `MYSQL`e bağlanıp `PhpMyAdmin` için kullanıcı oluşturalım.


```
MariaDB [(none)]> CREATE USER 'phpmyadmin'@'localhost' IDENTIFIED BY '1234-Aaa!';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO 'phpmyadmin'@'localhost';
```




Bunları yazdıktan sonra tarayıcımıza girip test edelim.

**Not:** Test etmeden önce bir kaç bilgi vereceğim. `http://alanadi.com/phpmyadmin/` olarak yazarsanız yüksek ihtimalle `Forbidden – You don’t have permission to access / on this server` hatası alacaksınız. Bunun sebebi PhpMyAdmin de gerekli yetkilerimizin olmaması. Log dosyalarına bakarakta bunları görebiliriz.

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/039d5299-edb7-4848-afbe-8fcc6de6e42b)


Bu hatayı gidermek için aşağıdaki komutu yazabilirsiniz.

```
sudo chmod -R 755 /usr/share/phpMyAdmin
```

Oluşan hata yüksek ihtimalle bu komutla düzelmeyecektir. Bu hatayı düzeltmek için PhpMyAdminin .conf dosyasına gideceğiz. `nano /etc/httpd/conf.d/phpMyAdmin.conf` yazarak .conf dosyasına gidelim ve aşağıdaki komutları yazalım.

```
<Directory /usr/share/phpMyAdmin/>
   AddDefaultCharset UTF-8
   Options Indexes FollowSymLinks MultiViews
   DirectoryIndex index.php
   AllowOverride all
   Require all granted
   Require local
</Directory>
```

Bu işlemi yaptıktan sonra `systemctl restart httpd` yazarak servisimizi restart ediyoruz.


Tarayıcımızı açıp test edelim.


![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/447590da-fc25-4021-a184-8c8b28df533a)


![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/0c8ac2ac-4590-4396-aeed-37a78a212e89)


PhpMyAdmin başarılı bir şekilde kurulmuştur.


## Wordpress Kurulumu

Aşağıdaki komutu yazarak `Wordpressi` indirelim

```
wget https://wordpress.org/latest.tar.gz
```

`latest.tar.gz` dosyamızın geldiğini görüyoruz. `tar xzfv latest.tar.gz` yazarak arşiv dosyamızı ayıklıyoruz. `Wordpress` adında bir dizin geldiğini göreceksiniz. Bu dizin içerisine girip şu komutu yazmanız gerekecektir. `cp -r * ..` Dizin içerisindeki dosyaları, klasörleri bir üst dizine koplayacaktır. Bu işlem yapıldıktan sonra aşağıdaki komutları tek tek yazın.

```
cd ..
rm -rf wordpress/
rm -rf latest.tar.gz
```

Karışıklık çıkarmaması adına bunları siliyoruz. Şimdi tarayıcımızı açıp test edelim:


![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/e976884b-181a-4f6f-9243-cb63a5add766)


Şimdi Veritabanında `Wordpress` için bir tane Database ve kullanıcı oluşturuyoruz.

```
MariaDB [(none)]> CREATE DATABASE wpdb;
MariaDB [(none)]> CREATE USER 'wpdbuser'@'localhost' IDENTIFIED BY '1234-Aaa??';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON wpdb. * TO 'wpdbuser'@'localhost';
```

Kullanıcımızı ve Databaseyi oluşturduktan sonra şimdi giriş yapalım:

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/e8e9eb16-65b7-4df7-9ee5-4999bf5f4226)


`Submit` butonuna bastıktan sonra karşınıza aşağıdaki gibi bir ekran gelebilir. Eğer gelmediyse devam edin, geldiyse aşağıdaki adımları takip ederek bu sorunu düzeltelim.

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/cd290d14-18f7-4ab1-9bd6-94bc27ad7ecf)



**Not:** Bu sorunun oluşmasının sebebi `wp-config.php` dosyasının manual olarak oluşamamasındandır. Bu sorunu gidermek için aşağıdaki adımları takip ediniz:

- Ekranda çıkan `php` kodlarını kopyalayın
- Terminale/konsola gelip `nano wp-config.php` yazın.
- Kopyalamış olduğunuz `php` kodunu buraya yapıştırın ve kaydedip çıkın.

Butona tıkladıktan sonra eğer işlemleri doğru yaptıysanız aşağıdaki gibi bir ekran gelecektir.

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/6410a996-0388-4f33-b521-5ad15c3348e0)


Bu kısıma gerekli bilgileri girdikten sonra `Install WordPress` butonuna tıklayın. İşlem tammadır

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/d2a52d0b-3e30-4ce5-93c6-cdca49ca5ab0)


Gerekli bilgileri doldurduktan sonra giriş yapın.

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/391105aa-5f7b-4147-9a56-94cfc97f7cc5)


WordPress kurulumu başarıyla tamamlanmıştır.


## SSL Kurulumu

SSL (Secure Sockets Layer), internet üzerindeki veri iletimini şifreleyen bir güvenlik protokolüdür. Bu, kullanıcıların web siteleri aracılığıyla bilgi gönderirken, üçüncü şahısların bu bilgilere erişmesini zorlaştırarak güvenli bir iletişim sağlar.

Şimdi serverımıza SSL kuracağız. Aşağıdaki komutu yazarak gerekli indirmeleri yapalım.

```
sudo dnf install certbot python3-certbot-apache
```

İlgili kurulumları yapmak için `cerbot --apache` yazıyoruz. Bunu yazınca aşağıdaki gibi bir hata alabilirsiniz. Bu hatayı almanızın sebebi ilgili alan adınızn herhangi bir DNS bölgesi olmamasından kaynaklıdır. Biz kendi localimizde denediğimiz için böyle bir hata vermesi gayet normaldir.

```
[root@fedora masterweb]# certbot --apache
Saving debug log to /var/log/letsencrypt/letsencrypt.log

Which names would you like to activate HTTPS for?
We recommend selecting either all domains, or all domains in a VirtualHost/server block.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: ugur.com
2: www.ugur.com
3: ugur1.com
4: www.ugur1.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 4
Requesting a certificate for www.ugur1.com

Certbot failed to authenticate some domains (authenticator: apache). The Certificate Authority reported these problems:
  Domain: www.ugur1.com
  Type:   dns
  Detail: DNS problem: NXDOMAIN looking up A for www.ugur1.com - check that a DNS record exists for this domain; DNS problem: NXDOMAIN looking up AAAA for www.ugur1.com - check that a DNS record exists for this domain

Hint: The Certificate Authority failed to verify the temporary Apache configuration changes made by Certbot. Ensure that the listed domains point to this Apache server and that it is accessible from the internet.

Some challenges have failed.
Ask for help or search for solutions at https://community.letsencrypt.org. See the logfile /var/log/letsencrypt/letsencrypt.log or re-run Certbot with -v for more details.
[root@fedora masterweb]#
```

### Public Sunucu da SSL Kurulumu

İşlemler tamamen aynı herhangi bir değişiklik yok. Terminale `certbot --apache` yazıyoruz.

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/faeafe10-c5c0-4ca6-be4b-8c5e2032c4b8)

Bu kısımda SSL sertifikasını kurmak istediğiniz alan adını belirtiyorsunuz. Ben "altimasiciyorum.com.tr" alan adımı seçiyorum. 

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/6b56ddc0-bc6b-4cf1-92da-60de4893ad11)



Bana burada belirtmiş olduğum sitede zaten bir SSL olduğunu ve sertifikayı kaldırıp yeniden yükleyebileceğimizi yada sertifikayı yenileyip değiştirebileceğimizi söylüyor. Ben sertifikayı tekrar yükleyeceğim. 


Kurulum otomatik tamamlandı ve SSL sertifikamız başarılı bir şekilde kuruldu. SSL sertifkanızın .conf dosya yoluna gitmek için `/etc/httpd/conf.d/alanadi.com-le-ssl.com` yazabilirsiniz.

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/dd3dbdae-f216-44c0-acaa-27dd7f5093c0)




![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/145ec110-d628-4a4d-913a-0848703322d3)



**Not:** Kurmuş olduğumuz SSL let's encrypt sertifikasıdır. Her 3 ayda bir yenilenmesi gerekir.

### OPENSSL

Biz de openssl ile kuracağız. Bu tür sertifikalara `Self-signed` denir. Self-signed (kendi imzalı) bir SSL sertifikası, bir güvenlik sertifikası sağlayıcısı yerine web sitesinin kendisi tarafından oluşturulan ve imzalanan bir sertifikadır. Bu tür sertifikalar genellikle test ortamları veya kişisel kullanım için kullanılır

Aşağıdaki komutu yazarak işlemi başlatalım:

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/pki/tls/private/server.key -out /etc/pki/tls/certs/server.crt
```

Aşağıdaki bilgileri doldurunuz 

```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:
State or Province Name (full name) []:
Locality Name (eg, city) [Default City]:
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:
Email Address []:
```


.conf dosyamıza  `/etc/httpd/conf.d/sirket1.com.conf`  yazarak gidelim ve aşağıdaki configleri yapalım. Virtual kısmına `<VirtualHost *:443>` yazıyoruz.

```
        SSLEngine on
        SSLCertificateFile /etc/pki/tls/certs/server.crt
        SSLCertificateKeyFile /etc/pki/tls/private/server.key
```

Bunları yazdıktan sonra kaydedip çıkıyoruz. `systemctl restart httpd` yazarak servisimizi yeniden başlatıyoruz.


Web sayfamızın Sertifikasını incelediğimizde bizim verdiğimiz bilgilerle kurulduğu görülmüştür.

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/f0fd0535-7d94-4087-b4e3-b2cc183e7672)


Hala güvenli değil ibaresi alıyor olabilirsiniz bunun sebebi kurmuş olduğumuz sertifikanın `Self-signed` olmasındandır.

### HTTPS Redirect

Bunun için ` nano /etnano /etc/httpd/conf.d/sirket1.com-80.conf` yazıyoruz ve dosyamız oluşuyor. Bunun içerisine ` nano /etnano /etc/httpd/conf.d/sirket1.com.conf` dosyamızın içerisindeki verileri alıp yapıştırıyoruz ve aşağıdaki ayarları yapalım.

```
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.
        #ServerName www.example.com


        ServerName   ugur.com
        ServerAlias www.ugur.com
        Redirect permanent / https://www.ugur.com

        #SSLEngine on
        #SSLCertificateFile /etc/pki/tls/certs/server.crt
        #SSLCertificateKeyFile /etc/pki/tls/private/server.key

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog /var/www/sirket1.com/error.log
        CustomLog /var/www/sirket1.com/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

Şimdi web sitemize giriş yapalım. 

![image](https://github.com/ugurcomptech/Fedora-WEB-Server/assets/133202238/92198316-bebe-400a-9d89-f2a545922005)

Bu şekilde giriş yaptığınızda tekrardan `https` kısmına yönlendirdiğini görebilirsiniz.

SSL kurulumu başarılı bir şekilde yapılmıştır.

