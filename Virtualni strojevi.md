# Mrežni i mobilni operacijski sustavi - Seminarski rad: Skladište
Luka Ljubojević
# Uvod
Za temu 1. seminarskog rada iz kolegija Mrežni i mobilni operacijski sustavi  napraviti ću konfiguraciju web poslužitelja za vođenje stanja i lokacije artikala na skladištu.

Shema se sastoji od: 

 - Dva web poslužitelja,
 - MariaDB baze podataka,
 - Balanser opterećenja,
 - Tri klijenta (Android, Windows, GNU Linux - Ubuntu),

Grafički shema izgleda:
![](https://i.imgur.com/39rgXxc.png)


Elementi sheme su konfigurirani kao virtualni strojevi unutar alata upravljač virtualnog stroja (eng. "Virtual Machine Manager").

Seminarski rad je izrađen u sustavu GNU Linux - Mint, uz pakete (na poslužiteljskom računalu - koji pokreće virtualne strojeve): 

 - virt-manager, 
 - cloud-localds, 
 - qemu, 
 - mariadb-client-core-10.3, 
 - python3 pip, 
 - mysqlclient

# Priprema ubuntu-server slika sustava

Za potrebe seminarskog rada preuzimamo "Ubuntu Server 20.04 LTS (Focal Fossa) daily builds" sliku sustava,
* Dostupnu na: https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img

Sliku sustava kopiramo četri puta, preimenujemo u skladu s imenom virtualnog stroja i složimo u mape po slijedećoj hijerarhiji:
* PZ MMOS
    * LB
    * MySQL
    * WS1
    * WS2

Zatim kreiramo datoteku user-data pomoću sustava cloud-init koja sadrži:
* Lozinku
* Atribute da lozinka ne ističe i da se može koristiti za ssh autentikaciju

Datoteka user-data je sadržaja:
```
#cloud-config
password: lozinka1
chpasswd: {expire: False}
ssh_pwauth: True
```
Pomoću datoteke user-data stvaramo drugi disk za virtualni stroj koji će omogućiti prijavu u sustav (user-data.img)...
Naredbama:
```
lljubojevic@Omen17:~$ cloud-localds user-data.img user-data
lljubojevic@Omen17:~$ mv user-data.img "/home/lljubojevic/PZ MMOS/MySQL"
lljubojevic@Omen17:~$ cloud-localds user-data.img user-data
lljubojevic@Omen17:~$ mv user-data.img "/home/lljubojevic/PZ MMOS/LB"
lljubojevic@Omen17:~$ cloud-localds user-data.img user-data
lljubojevic@Omen17:~$ mv user-data.img "/home/lljubojevic/PZ MMOS/WS1"
lljubojevic@Omen17:~$ cloud-localds user-data.img user-data
lljubojevic@Omen17:~$ mv user-data.img "/home/lljubojevic/PZ MMOS/WS2"
```
Nakon sortiranja slika u mape izvršavamo povećanje kapaciteta (prostora za pohranu) na svakoj slici naredbama:
```
lljubojevic@Omen17:~$ cd PZ\ MMOS/
lljubojevic@Omen17:~/PZ MMOS$ ls
LB  MySQL  WS1  WS2

lljubojevic@Omen17:~/PZ MMOS$ cd LB
lljubojevic@Omen17:~/PZ MMOS/LB$ qemu-img resize lb.img 30G
Image resized.

lljubojevic@Omen17:~/PZ MMOS/LB$ cd ..
lljubojevic@Omen17:~/PZ MMOS$ cd MySQL/
lljubojevic@Omen17:~/PZ MMOS/MySQL$ qemu-img resize mysql.img 30G
Image resized.

lljubojevic@Omen17:~/PZ MMOS/MySQL$ cd ..
lljubojevic@Omen17:~/PZ MMOS$ cd WS1
lljubojevic@Omen17:~/PZ MMOS/WS1$ qemu-img resize ws1.img 30G
Image resized.

lljubojevic@Omen17:~/PZ MMOS/WS1$ cd ..
lljubojevic@Omen17:~/PZ MMOS$ cd WS2
lljubojevic@Omen17:~/PZ MMOS/WS2$ qemu-img resize ws2.img 30G
Image resized.

```
# Podešavanje virtualnih strojeva
## Virutalni stroj MySQL
Virtualni stroj podešen je prema slijedećem:
* Import existing image -> Browse Local -> mysql.img
* Operating system: Ubuntu 20.04
* Memory: 4096, CPUS: 2
* Name MySQL
* Customize configuration before install -> Add Hardware -> Storage -> Select storage -> Local -> user-data.img
![](https://i.imgur.com/ug1QA36.png)

## Virutalni stroj LB 
Virtualni stroj podešen je prema slijedećem:
* Import existing image -> Browse Local -> lb.img
* Operating system: Ubuntu 20.04
* Memory: 4096, CPUS: 2
* Name LB
* Customize configuration before install -> Add Hardware -> Storage -> Select storage -> Local -> user-data.img
![](https://i.imgur.com/fDCoY0e.png)


## Virutalni stroj WS1
Virtualni stroj podešen je prema slijedećem:
* Import existing image -> Browse Local -> ws1.img
* Operating system: Ubuntu 20.04
* Memory: 4096, CPUS: 2
* Name WS1
* Customize configuration before install -> Add Hardware -> Storage -> Select storage -> Local -> user-data.img
![](https://i.imgur.com/dGaSvhZ.png)


## Virutalni stroj WS2
Virtualni stroj podešen je prema slijedećem:
* Import existing image -> Browse Local -> ws2.img
* Operating system: Ubuntu 20.04
* Memory: 4096, CPUS: 2
* Name WS2
* Customize configuration before install -> Add Hardware -> Storage -> Select storage -> Local -> user-data.img
![](https://i.imgur.com/FPUHujU.png)

## Virtualni stroj WinCl
Virtualni stroj podešen je prema slijedećem:
* Local install media -> Browse Local -> Windows 7 Instalacijska slika.iso
* Operating system: Windows Server 2016
* Memory: 4096, CPUS: 2
* Enable storage, disk image: 40GiB
* Name WinCl
![](https://i.imgur.com/UcJSakm.png)


## Virutalni stroj UbuntuCl
Virtualni stroj podešen je prema slijedećem:
* Local install media -> Browse Local -> Ubuntu 20.04 Instalacijska slika.iso
* Operating system: Ubuntu 20.04
* Memory: 2048, CPUS: 2
* Enable storage, disk image: 20GiB
* Name UbuntuCl
![](https://i.imgur.com/qKTOpCC.png)

## Virutalni stroj Andrx86Cl
Virtualni stroj podešen je prema slijedećem:
* Local install media -> Browse Local -> Android x86 Instalacijska slika.iso, dostupna na: https://www.fosshub.com/Android-x86.html?dwl=android-x86_64-9.0-r2.iso
* Operating system: Android-x86 9.0
* Memory: 2048, CPUS: 2
* Enable storage, disk image: 10GiB
* Name Andrx86Cl
![](https://i.imgur.com/kY2JgOe.png)

# Virtualni stroj MySQL (baza podataka)
Nakon uspješne prijave u sustav iz alata virt-manager, započinje konfiguracija poslužitelja baze podataka.

Prvo provjeravamo IP adresu virtualnog stroja:
```
ubuntu@ubuntu:~$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:5a:31:f3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.77/24 brd 192.168.122.255 scope global dynamic enp1s0
       valid_lft 2003sec preferred_lft 2003sec
    inet6 fe80::5054:ff:fe5a:31f3/64 scope link 
       valid_lft forever preferred_lft forever
```

Radi jednostavnosti, spajamo se na virtualni stroj s glavnog računala putem naredbe ssh:
```
lljubojevic@Omen17:~$ ssh ubuntu@192.168.122.77
The authenticity of host '192.168.122.77 (192.168.122.77)' can't be established.
ECDSA key fingerprint is SHA256:91CCjASJznu9k2edurBZjRXqU4CFqa9piL2tFY3VY7Q.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.122.77' (ECDSA) to the list of known hosts.
ubuntu@192.168.122.77's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Jan  9 16:24:25 UTC 2022

  System load:  0.21              Processes:               137
  Usage of /:   6.0% of 28.90GB   Users logged in:         1
  Memory usage: 7%                IPv4 address for enp1s0: 192.168.122.77
  Swap usage:   0%


31 updates can be applied immediately.
14 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


Last login: Sun Jan  9 16:20:28 2022
```
Po podešavanju ssh veze, vršimo ažuriranje repozitorija i instalaciju MariaDB poslužitelja:
```
ubuntu@ubuntu:~$ sudo apt update
ubuntu@ubuntu:~$ sudo apt install mariadb-server
ubuntu@ubuntu:~$ sudo mariadb
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 50
Server version: 10.3.32-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
Naredbom:
```
ubuntu@ubuntu:~$ sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
U sekciji [mysqld] mijenjamo adresu slušanja u 0.0.0.0:
    bind-address            = 0.0.0.0
    
Adresa 0.0.0.0 predstavlja slušanje (vezivanje) na svaku dostupnu mrežu - odnosno, prihvaća zahtjeve iz svih dijelova mreže.

Nakon promjene adrese slušanja ponovno pokrećemo MariaDB servis:
```
ubuntu@ubuntu:~$ sudo systemctl restart mysql

ubuntu@ubuntu:~$ sudo systemctl status mariadb.service
● mariadb.service - MariaDB 10.3.32 database server
     Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2022-01-09 16:57:15 UTC; 18s ago
    ...
   Main PID: 3143 (mysqld)
     Status: "Taking your SQL requests now..."
      Tasks: 31 (limit: 4682)
     Memory: 63.8M
     CGroup: /system.slice/mariadb.service
             └─3143 /usr/sbin/mysqld
```
Nakon podešavanja servisa MariaDB stvaramo bazu podataka za potrebe web poslužitelja (web aplikacije).
Uz bazu podataka, potrebno je napraviti i korisnika koji će imati pravo pristupa bazi.
```
MariaDB [(none)]> CREATE DATABASE Skladiste;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> USE Skladiste;
Database changed

MariaDB [Skladiste]> CREATE USER webserver IDENTIFIED BY 'lozinka1';
Query OK, 0 rows affected (0.002 sec)

MariaDB [Skladiste]> GRANT ALL PRIVILEGES ON Skladiste TO webserver;
Query OK, 0 rows affected (0.001 sec)
```
Baza podataka je spremna za korištenje. 
Provjerimo možemo li ostvariti povezivanje na bazu s drugog računala:
```
lljubojevic@Omen17:~$ mysql -h 192.168.122.77 -u webserver -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 39
Server version: 10.3.32-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 

```
# Izrada web aplikacije
Web aplikaciju izraditi ćemo na glavnom računalu.
Prije izrade, potrebno je instalirati pakete za povezivanje na mysql bazu podataka:
```
lljubojevic@Omen17:~$ sudo apt install mariadb-client-core-10.3
lljubojevic@Omen17:~/PZ MMOS/Skladiste/skladiste$ pip install mysqlclient
```
Unutar mape projektnog zadatka "PZ MMOS" stvaramo još jednu mapu "Skladište" u kojoj ćemo stvoriti Django web aplikaciju:
```
lljubojevic@Omen17:~/PZ MMOS$ mkdir Skladiste
lljubojevic@Omen17:~/PZ MMOS$ cd Skladiste
lljubojevic@Omen17:~/PZ MMOS/Skladiste$ django-admin startproject skladiste
lljubojevic@Omen17:~/PZ MMOS/Skladiste$ ls
skladiste
```
Prije početka izrade web aplikacije, potrebno je promijeniti bazu podataka iz SQLite u MySQL.
(Prema zadanom u Django postavkama je baza SQLite).

U datoteci settings.py mijenjamo DATABASES na slijedeći način:
```
lljubojevic@Omen17:~/PZ MMOS/Skladiste$ cd skladiste/skladiste
lljubojevic@Omen17:~/PZ MMOS/Skladiste/skladiste/skladiste$ nano settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'Skladiste',
        'HOST': '192.168.122.77',
        'PORT': '3306',
        'USER': 'webserver',
        'PASSWORD': 'lozinka1',
    }
}
```
Gdje je:
* 'NAME' ime baze podataka na virtualnom stroju MySQL, 
* 'HOST' IP adresa virtualnog stroja MySQL, 
* 'PORT' zadana vrata MySQL poslužitelja, 
* 'USER' i 'PASSWORD' korisnik s pravima uporabe baze podataka

Započinjemo izradu web aplikacije kreiranjem aplikacije main:
```
lljubojevic@Omen17:~/PZ MMOS/Skladiste/skladiste$ django-admin startapp main
```
Aplikacija će prikazivati stanje robe na skladištu.

Unutar strukture aplikacije dodajemo slijedeći programski kod:

**main/urls.py:**
```
from django.urls import path
from . import views

app_name = 'main'

urlpatterns = [
    path('homepage', views.homepage, name='homepage'),
]

```
**main/views.py:**
```
from django.shortcuts import render
from main.models import Artikl

def homepage(request):
    artikls = Artikl.objects.all()
    context = {'artikls': artikls}
    return render(request, 'artikli.html', context=context)
```
**main/models.py:**
```
from django.db import models

# Create your models here.
class Artikl(models.Model):
    SifraArt=models.IntegerField(primary_key=True)
    NazivArt=models.CharField(max_length=350)
    KolNaStanju=models.IntegerField()
    Lokacija=models.CharField(max_length=400)
    Dostupan=models.BooleanField(default=True)

class Meta:
        db_table = 'Skladiste'
```
**main/admin.py:**
```
from django.contrib import admin
from django.db import models
from main.models import Artikl

# Register your models here.
admin.site.register(Artikl)
```
**main/templates/artikli.html:**
```
<ul>
    {% for a in artikls %}
        <li>
            Sifra Artikla: {{ a.SifraArt }}<br>
            Naziv Artikla: {{ a.NazivArt }}<br>
            Kolicina na stanju:{{ a.KolNaStanju }}<br>
            Lokacija:{{ a.Lokacija }} <br>
            Dostupan:{{ a.Dostupan }} 
        </li><br>
    {% endfor %}
</ul>
```
**skladiste/urls.py:**
```
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('main/', include('main.urls')),
]

```
**skladiste/settings.py:**
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'main.apps.MainConfig'
]
```
U konačnici nastaje web aplikacija ovog tipa:
![](https://i.imgur.com/AQD1i2C.png)

# Virtualni strojevi WS1 i WS2 (web poslužitelji 1 i 2)
Nakon uspješne prijave u sustave iz alata virt-manager, započinje konfiguracija oba poslužitelja iste web aplikacije.

Prvo provjeravamo IP adrese virtualnih strojeva:

WS1:
```
ubuntu@ubuntu:~$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:d7:39:a7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.133/24 brd 192.168.122.255 scope global dynamic enp1s0
       valid_lft 3524sec preferred_lft 3524sec
    inet6 fe80::5054:ff:fed7:39a7/64 scope link 
       valid_lft forever preferred_lft forever

```

WS2:
```
ubuntu@ubuntu:~$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:fe:8c:56 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.115/24 brd 192.168.122.255 scope global dynamic enp1s0
       valid_lft 3494sec preferred_lft 3494sec
    inet6 fe80::5054:ff:fefe:8c56/64 scope link 
       valid_lft forever preferred_lft forever

```
Također, radi jednostavnosti, spajati ćemo se na virtualne strojeve s glavnog računala putem naredbe ssh.

Vršimo instalaciju Python3 interpretera i okvira Django na oba web poslužitelja:
```
ubuntu@ubuntu:~$ sudo apt update
ubuntu@ubuntu:~$ sudo apt install python3 python3-pip python-is-python3 pylint
ubuntu@ubuntu:~$ sudo pip3 install Django
Collecting Django
  Downloading Django-4.0.1-py3-none-any.whl (8.0 MB)
     |████████████████████████████████| 8.0 MB 2.8 MB/s 
Collecting backports.zoneinfo; python_version < "3.9"
  Downloading backports.zoneinfo-0.2.1-cp38-cp38-manylinux1_x86_64.whl (74 kB)
     |████████████████████████████████| 74 kB 176 kB/s 
Collecting sqlparse>=0.2.2
  Downloading sqlparse-0.4.2-py3-none-any.whl (42 kB)
     |████████████████████████████████| 42 kB 747 kB/s 
Collecting asgiref<4,>=3.4.1
  Downloading asgiref-3.4.1-py3-none-any.whl (25 kB)
Installing collected packages: backports.zoneinfo, sqlparse, asgiref, Django
Successfully installed Django-4.0.1 asgiref-3.4.1 backports.zoneinfo-0.2.1 sqlparse-0.4.2

```
Postavljamo krovni direktoriji web aplikacije (mapu Skladište) na oba web poslužitelja:

Vršimo kreiranje virtualne CD slike koja sadrži web aplikaciju:
```
lljubojevic@Omen17:~/PZ MMOS/Skladiste$ mkisofs -o webapp.iso ./skladiste
I: -input-charset not specified, using utf-8 (detected in locale settings)
Total translation table size: 0
Total rockridge attributes bytes: 0
Total directory bytes: 14528
Path table size(bytes): 118
Max brk space used 22000
208 extents written (0 MB)

```
Unutar virt-manager alata dodajemo sliku CD-a u virtualne strojeve:
![](https://i.imgur.com/48dbmdh.png)

CD montiramo na sustav naredbama:
```
ubuntu@ubuntu:/dev$ cd /media
ubuntu@ubuntu:/media$ sudo mkdir cd
ubuntu@ubuntu:/media$ sudo mount /dev/sr0 /media/cd
mount: /media/cd: WARNING: device write-protected, mounted read-only.

```
Sve stavke s slike CD-a prekopiramo u korisnički direktorij web poslužitelja:
```
ubuntu@ubuntu:~$ cd /media/cd
ubuntu@ubuntu:/media/cd$ cp -r . /home/ubuntu/webapp
ubuntu@ubuntu:/media/cd$ cd /home/ubuntu/webapp
ubuntu@ubuntu:~/webapp$ ls
main  manage.py  skladiste

```
Na web poslužiteljima instaliramo MySQL kijente:
```
ubuntu@ubuntu:~/webapp$ sudo apt install mariadb-client-core-10.3
ubuntu@ubuntu:~/webapp$ sudo apt-get install python3-dev default-libmysqlclient-dev build-essential
ubuntu@ubuntu:~/webapp$ sudo pip3 install mysqlclient
Collecting mysqlclient
  Using cached mysqlclient-2.1.0.tar.gz (87 kB)
Building wheels for collected packages: mysqlclient
  Building wheel for mysqlclient (setup.py) ... done
  Created wheel for mysqlclient: filename=mysqlclient-2.1.0-cp38-cp38-linux_x86_64.whl size=109128 sha256=7d74ec4c97da034715dcdb88d94a14c6d3ec0fbbc505e94c3ace8d659a388b4a
  Stored in directory: /root/.cache/pip/wheels/61/e7/42/9d56347e42d7ce19397c0ca050c6bef56640e18be7021ac189
Successfully built mysqlclient
Installing collected packages: mysqlclient
Successfully installed mysqlclient-2.1.0

```
Dodajemo naše poslužitelje i balanser opterećenja u allowed hosts u skladiste/settings.py datoteci:
```
#WS1
ubuntu@ubuntu:~/webapp/skladiste$ sudo nano settings.py
ALLOWED_HOSTS = ['192.168.122.133', '192.168.122.132']

#WS2
ubuntu@ubuntu:~/webapp/skladiste$ sudo nano settings.py
ALLOWED_HOSTS = ['192.168.122.115','192.168.122.132']

```

Pokrećemo posluživanje web aplikacije na oba poslužitelja:
```
#WS1:
ubuntu@ubuntu:~/webapp$ ./manage.py runserver 192.168.122.133:8000

#WS2:
ubuntu@ubuntu:~/webapp$ ./manage.py runserver 192.168.122.115:8000
```

Uklanjamo CD sliku s oba poslužitelja u alatu virt-manager i testiramo poslužitelje.

WS1:
![](https://i.imgur.com/PFaHHYh.png)


WS2:
![](https://i.imgur.com/eRvCnNp.png)


# Virtualni stroj LB (Balanser opterećenja)
Nakon uspješne prijave u sustav iz alata virt-manager, započinje konfiguracija balansera opterećenja.

Prvo provjeravamo IP adresu virtualnog stroja:

```
ubuntu@ubuntu:~$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:fd:36:0d brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.132/24 brd 192.168.122.255 scope global dynamic enp1s0
       valid_lft 3503sec preferred_lft 3503sec
    inet6 fe80::5054:ff:fefd:360d/64 scope link 
       valid_lft forever preferred_lft forever

```

I na LB virtualni stroj ćemo se spojiti putem naredbe ssh, radi jednostavnosti.

```
ubuntu@ubuntu:~$ sudo apt update.
ubuntu@ubuntu:~$ sudo apt install haproxy

```

U datoteku /etc/haproxy/haproxy.cfg dodajemo konfiguraciju poslužitelja:
```
frontend skladiste
        bind :8000
        default_backend sklserveri
backend sklserveri
        server ws1 192.168.122.133:8000
        server ws2 192.168.122.115:8000

```
Ponovno pokrećemo servis HAProxy:
```
ubuntu@ubuntu:~$ sudo service haproxy restart
```
Sada možemo pristupiti web poslužiteljima putem balansera opterećenja:
![](https://i.imgur.com/2V5ONnO.png)

# Virtualni stroj Windows klijent
Za primjer arhitekture baza - poslužitelji - balanaser - klijenti koristiti ćemo više klijenata na različitim operacijskim sustavima.

Instalirali smo jedan virtualni stroj na sustav Windows 7.
Opcije instalacije su trivijalne, prema zadanim postavkama.

Web aplikacija iz klijenta na operacijskom sustavu Windows:
![](https://i.imgur.com/6Gj4f1A.png)


# Virtualni stroj GNU Linux - Ubutnu klijent
Za primjer arhitekture baza - poslužitelji - balanaser - klijenti koristiti ćemo više klijenata na različitim operacijskim sustavima.

Instalirali smo jedan virtualni stroj na sustav Ubuntu 20.04.
Opcije instalacije su trivijalne, prema zadanim postavkama.

Web aplikacija iz klijenta na operacijskom sustavu Ubuntu:
![](https://i.imgur.com/TP5e2CR.png)


# Virtualni stroj Android klijent
Za primjer arhitekture baza - poslužitelji - balanaser - klijenti koristiti ćemo više klijenata na različitim operacijskim sustavima.

Instalirali smo jedan klijent na sustav Android x86
(Android za procesore x86 - procesore u računalima).
Opcije instalacije su trivijalne, prema zadanim postavkama.

Web aplikacija iz klijenta na operacijskom sustavu Android:
![](https://i.imgur.com/WYYrEM7.png)
