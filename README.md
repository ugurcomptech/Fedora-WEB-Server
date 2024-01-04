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



