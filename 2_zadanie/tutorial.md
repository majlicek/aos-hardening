#  Zadanie č. 2 (Hardering Web servera) 

## Nastavenie VirtualBox-u
-	importovať image Debianu
-	vytvoriť klony (web-server)
-	network pre **web-server** nastaviť ako *Bridged Adapter* 

## 1. Inštalácia a spustenie Apache2

*Inštalácia*

`$apt-get install apache2`

*Spustenie*

`$service apache2 start/stop/status/restart/configtest`

## 2. Konfigurácia Apache2

*Základný konfiguračný súbor*

`/etc/apache2/apache2.conf`

*Nastavenie portov a IP adries*

`/etc/apache2/ports.conf`
> ak chceme, aby server počúval na porte 80, potom `Listen 80`

> ak chceme, aby server počúval na zvolenej IP adrese, potom `Listen <IP>:80` 


*Defaultne umiestnenie webových sídel*

`/var/www`

*Vytvorenie virtuálneho webového sídla*

```bash
$mkdir /var/www/domenask
$mkdir /var/www/domenask/prvyweb
$touch /var/www/domenask/prvyweb/index.html
$chmod -R 777 /var/www
$nano /etc/apache2/sites-available/prvyweb-domena.sk.conf
```

```bash
<VirtualHost *:80>        
  ServerAdmin janko.hrasko@gmail.com
  ServerName prvyweb.domena.sk
  DocumentRoot /var/www/domenask/prvyweb

  <Directory />
     Options FollowSymLinks
     AllowOverride None
  </Directory>
  <Directory /var/www/domenask/prvyweb/>
     Options Indexes FollowSymLinks MultiViews
     AllowOverride None
     Order allow,deny
     allow from all
  </Directory> 

  ErrorLog /var/log/apache2/error.log 
  LogLevel warn
  CustomLog /var/log/apache2/prvyweb.domena.sk.access.log combined
</VirtualHost>
```

*Aktivovanie nového virtuálneho hostu*

```bash
$a2ensite prvyweb-domena.sk.conf
$systemctl reload apache2.service // ak nejaká chyba, tak $journalctl | tail
```

## 3. Konfigurácia Apache2 - SSL/HTTPS

*Modul mod_ssl*

```bash
$a2enmod ssl
$service apache2 restart
```

*Šablóna pre virtuálny web cez SSL*

`/etc/apache2/sites-available/default-ssl` – skopírovať nastavenia SSL do nášho konfiguráku nového virtuálneho webového sídla

```bash
$a2ensite prvyweb-domena.sk.conf
$service apache2 restart
```

**Potom pristupujeme k webu https://IP_ADRESA:80/**

*Certifikát (Self-signed certifikát)*

```bash
apt-get install ssl-cert
make-ssl-cert
```

## 4. Inštalácia PHP

```bash
$apt-get install php 
$php -v
```

`$nano /etc/php/../apache2/php.ini` // konfiguracia php
