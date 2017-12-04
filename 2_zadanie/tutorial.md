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

## 5. Konfiguračné súbory + hardening

### Apache2

`/etc/apache2/apache2.conf`


```bash

```

`/etc/apache2/sites-available/prvyweb-domena.sk.conf`

```text
<IfModule mod_ssl.c>
 <VirtualHost *:1000>
	ServerAdmin janko.hrasko@gmail.com
	ServerName prvyweb.domena.sk
	ServerAlias www.prvyweb.domena.sk
	DocumentRoot /var/www/domenask/prvyweb

	SSLEngine on
	SSLCertificateFile	/etc/ssl/certs/ssl-cert-snakeoil.pem
	SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
	<FilesMatch "\.(cgi|shtml|phtml|php)$">
		SSLOptions +StdEnvVars
	</FilesMatch>
	<Directory /usr/lib/cgi-bin>
		SSLOptions +StdEnvVars
	</Directory>

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
	<Directory />
		Options None
		Order allow,deny
		Allow from all
	</Directory>

	LogLevel warn
	ErrorLog /var/log/apache2/error.log
	CustomLog /var/log/apache2/prvyweb.domena.sk.access.log combined
 </VirtualHost>
</IfModule>
```

### PHP 

`/etc/php/7.0/apache2/php.ini` - neobsahuje celý súbor, iba dôležité prvky


```text
[PHP]
; Inspiration sources:
;   (1) madirish.net/199
;   (2) cyberciti.biz/tips/php-security-best-practices-tutorial.html
;   (3) owasp.org/index.php/PHP_Configuration_Cheat_Sheet

; Enable the PHP scripting language engine under Apache.
engine = On

; Obmedzenie PHP pristupovat k file systemu.
open_basedir = /secure/place

; Zakazanie nebezpecnych PHP funkcii.
disable_functions = pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,exec,passthru,shell_exec,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source,mail,imap_mail,eval,phpinfo,posix_getegid,posix_geteuid,posix_getgid,posix_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid,posix_getppid,posix_getpwnam,posix_getpwuid,posix_getrlimit,posix_getsid,posix_getuid,posix_initgroups,posix_isatty,posix_kill,posix_mkfifo,posix_mknod,posix_setegid,posix_setuid,posix_setgid,posix_setgid,posix_setsid,posix_setuid,posix_strerror,posix_ttyname,posix_uname,posix_access,posix_ctermid

; Logovanie vsetkych PHP chyb.
display_errors = Off
log_errors = On
error_log = /var/log/apache2/php_scripts_error.log

; Enable cgi.force_redirect.
cgi.force_redirect = 1

; Upload suborov
file_uploads = Off

; Zmena dopcasneho adresara pre uploadovane subory.
upload_tmp_dir = /home/tmp

; Maximum allowed size for uploaded files.
upload_max_filesize = 2M

; Maximum number of files that can be uploaded via a single request
max_file_uploads = 20

; Kontrola POST velkosti.
post_max_size = 32M

; Kontrola zdrojov (DoS control)
max_execution_time = 40
max_input_time = 40
memory_limit = 40M

; Zakaz informacii o PHP.
expose_php = Off

; Zmena znakovej sady.
default_charset = "iso-8859-1"

; Vypnutie vykonavania vzdialeneho kodu.
allow_url_fopen = Off
allow_url_include = Off

error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT

display_startup_errors = Off

log_errors_max_len = 1024

ignore_repeated_errors = Off

ignore_repeated_source = Off

report_memleaks = On

track_errors = Off

html_errors = Off

[SQL]
sql.safe_mode = On

[Session]
; Ochrana sessions.
session.save_path = "/var/lib/php/sessions"
session.use_strict_mode = 0
session.use_cookies = 1
;session.cookie_secure =
session.use_only_cookies = 1
session.auto_start = 0

; Nastavenie cookies.
session.cookie_lifetime = 1800
session.name = SESSION

session.cookie_httponly = 1
```
