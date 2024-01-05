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




