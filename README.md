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
nano /etc/apache2/sites-enabled/sirket1.com.conf
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

`systemctl restart httpd` komutuyla servisimizi yeniden başlatıyoruz. Bu kısımda Seleniux ile ilgili bazı hatalar alacağız.

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







