# Fedora-WEB-Server

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



## Aynı Portta Çoklu Site Yayınlama

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











