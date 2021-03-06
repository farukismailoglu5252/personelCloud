# KİŞİSEL BULUT DEPOLAMA
![owncloud](https://github.com/farukismailoglu5252/personelCloud/blob/master/ownloud.png "owncloud server bilesen diyagrami")

## Projenin Amacı
Projeyi yapma amacım kişisel verilerimin google drive, yandex disk ve benzeri bulut depolama alanlarında bulundurmak yerine kendi sunucum üzerinde bulundurmak. Böylece kişisel verilerimin güvenliğini arttırmakdır.


## Proje Hakkinda 
Projeyi yaparken de maliyet ve tasarruf acısından daha iyi olan raspberry pi 3 ve acik kaynakli 'owncloud' yazilimini kullandim. Depolama olarak ise 1 TB harici harddisk kullanarak 1 TB depolama alanina sahip oldum. Sizde ihtiyacınıza göre depolama alanları kullanarak düşük maliyetlerle kişisel bulut depolama alanlanınıza sahip olabilirsiniz. Ayrıca owncloud ücretli ve ücretsiz eklentilerle ek ozellikler ekleyebilirsiniz. Birden fazla kullanıcı oluşturarak kullanmalarına izin verdiklerinizde kişisel bulut depolama alanınıza kullanabilirler. 

Owncloud için yapılmış client uygulamaları sayesinde neredeyse bütün cihazlardan veri sekronize edebilir ve istediğiniz cihazdan erişebilirsiniz.

Ayrıca owncloud'un  Türkçe dil desteği olduğunuda bildirmek isterim.

Projemde Raspberry pi 3b+ kullandım ama araştırmalarım sonucunda  raspberry pi 2 ninde desteklediğini öğrendim lakin denemedim.
Bu projeyi yapmak  için Raspberry alacak iseniz üzerinde dahili wifi modeli bulunan raspberry pi 3b+ modelinin alınmasını öneririm.

Projeyi sanal makine üzerinde Raspbean işletim sisteminde ip ayarlamaları sebebiyle çalıştıramadım.



## Gerekli Donanım Listesi
* Raspbian Stretch yüklü raspberry pi 3/2
* Micro SD Card(En az 8 GB)
* Power Supply
* Harddisk (isteğe bağlı)

## KURULUM AŞAMALARI

root kullanici yetkisi almak icin
```
sudo su
```
Rasperry pi 3 güncelleştirmelerinin yapılması
```
apt update  && apt  upgrade
```
Güvenlik için rasperry pi 3 şifresinin değiştirilmesi

## Gerekli paketlerin kurulumu
Owncloud 10 yazılımı için gerekli paketlerin kurulması
```
apt install apache2 -y
```
```
apt install -y apache2 mariadb-server libapache2-mod-php7.0 \
    php7.0-gd php7.0-json php7.0-mysql php7.0-curl \
    php7.0-intl php7.0-mcrypt php-imagick \
    php7.0-zip php7.0-xml php7.0-mbstring
```

## Owncloud 10 yazılımının indirilmesi ve kurulum aşamaları

/tmp konumuna Owncloud  yazılımının indirilmesi
```
cd /tmp

wget https://download.owncloud.org/community/owncloud-10.0.10.tar.bz2
```
İndirilen Owncloud yazılımının arşivden cıkarılması
```
tar -xvf owncloud-10.0.10.tar.bz2

chown -R www-data:www-data owncloud
```
Oluşturulan Owncloud klasörünün taşınması
```
mv owncloud /var/www/html/
```

Ana dizin geri dönüş
```
cd
```

Apache sunucusunun baslatılması ve önyüklenebilir hale getirilmesi
```
systemctl start apache2

systemctl enable apache2
```


## Apache Web Server Yapılandırma
Aşağıda ki komut ile yeni bir yapılandırma dosyası oluşturma
```
sudo nano /etc/apache2/sites-available/owncloud.conf
```
Oluşturulan yeni yapılandırma dosyasına aşağıdaki satırları yapıştırarak kaydedin
```
Alias /owncloud "/var/www/html/owncloud/"

<Directory /var/www/html/owncloud/>
 Options +FollowSymlinks
 AllowOverride All

<IfModule mod_dav.c>
 Dav off
 </IfModule>

SetEnv HOME /var/www/html/owncloud
SetEnv HTTP_HOME /var/www/html/owncloud

</Directory>
```
```
ln -s /etc/apache2/sites-available/owncloud.conf /etc/apache2/sites-enabled/owncloud.conf
```
```
a2enmod headers
systemctl restart apache2
a2enmod env
a2enmod dir
a2enmod mime
```
```
mysql -u root -p
```
```
MariaDB [(none)]> create database owncloud;
 Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> create user owncloud@localhost identified by '12345';
 Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all privileges on owncloud.* to owncloud@localhost identified by '12345';
 Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
 Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit;
 Bye
```
## Harici Harddisk kullanamak istemeyenler

Depolama olarak harddisk kullanmak istemeyenler aşağıdaki 'Harici harddisk takilmasi 'kismini atlaayarak devam edebilir.
Eger bunu yaparsaniz depolama alaniniz yaklasik olarak Raspberry pi 3 deki bos hafisaniz kadar olacaktir. Bunu düşünerek hafıza kartı seçilmelidir.

## Harici  Harddisk takılması
Harici Harddisk için gerekli ntfs paketinin kurulumu
```
sudo apt-get install ntfs-3g -y
```
Verilerin depolanacağı alanın oluşturulması
```
sudo mkdir /media/ownclouddrive
```

```
sudo groupadd www-data
sudo usermod -a -G www-data www-data
```
İzinler ile alakalı kısım
```
sudo chown -R www-data:www-data /media/ownclouddrive
sudo chmod -R 775 /media/ownclouddrive
```
Harddiskin farklı usb portlarından takılabildiğinde de tanınabilmesi için gerekli kısım
```
id -g www-data
```
```
id -u www-data
```
```
ls -l /dev/disk/by-uuid
```
```
sudo nano /etc/fstab

```
Yukarıdaki komutlardan elde edilen bilgiler aşağıdaki komut satırındaki yerlere yazılarak tamamlanır.
```
UUID=F6941E59941E1D25 /media/ownclouddrive auto nofail,uid=33,gid=33,umask=0027,dmask=0027,noatime 0 0

```
Raspberry'nin yeniden başlatılması
```
sudo reboot

```
## Ip adresinin bulunması
Owncloud kurulumu icin ip adresimizin bulunmasi gerekmektedir.
Terminal ekranına `ifconfig` komutu yazılır.
Daha sonra resimde görüldüğü gibi işaretlenmiş alanda ip adresi yazmaktadır.

![owncloud](https://github.com/farukismailoglu5252/personelCloud/blob/master/ip.png "ip adresinin bulunması")


## Owncloud Ayarlarının Tamamlanması

Rasperry pi üzerinden veya aynı ağa bağlı bir cihazın internet tarayıcısını açarak  `ip_adresiniz/owncloud ` adresi gidilmeli,  ardından açılan owncloud arayüzünde gerekli alanlar doldurulmalı

![owncloud](https://github.com/farukismailoglu5252/personelCloud/blob/master/login.png "kullanıcı Adı ve Parola")
- Yönetici kullanıcı adı ve parola 

- Veritabanı için gerekli alanları veritabanı oluştururken girdiğimiz değerleri girin

![owncloud](https://github.com/farukismailoglu5252/personelCloud/blob/master/settings.png "database config")

- Data folder alanına `/media/ownclouddrive`
(Harddisk kullanmayacaksaniz Data Folder alanındaki kısmı degistirmeyin.)
* Username: owncloud
* Password: 12345
* Database: owncloud
* Server: localhost

`Finish setup` butonuna  tıklayarak ayarlamaları bitiriyoruz



## Owncloud'a bağlanma

OwnCloud kurulumu tamamlandıktan sonra cıkan ekrandan farklı platformlardan nasıl bağlanabileceği ve uygulama mağazalarında bulunan uygulamalara ulaşabilirsiniz.
![owncloud](https://github.com/farukismailoglu5252/personelCloud/blob/master/setup.png "client apps")

Ayrica aşağıda verdiğim link ile owncloud istemcilerinin indirme sayfasına gidebilirsiniz.

[Owncloud Client Download](https://owncloud.org/download/#install-clients)

## Owncloud Mobile Indirme QR Kod

![owncloud](https://github.com/farukismailoglu5252/personelCloud/blob/master/mobileDownload.png "mobile QR Kode")

## Windows Uygulaması Uzerinden baglanma 
İndirme sayfasındaki uygulama kurulduktan sonra gelen ekranda `Server Address ` bölümüne aşagıdaki resimde belirttiğm gibi owncloud adresinizi yazarak ownclouda bağlanıyoruz.

![owncloud](https://github.com/farukismailoglu5252/personelCloud/blob/master/winclient.png "windows client")

Desteklenen diğer platformlarda da aynı şekilde giriş yaparak verilerinizi sekronize edebilir ve verilerinize herhangi bir cihazdan ulaşabilirsiniz.
