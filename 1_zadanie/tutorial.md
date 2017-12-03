# Zadanie č. 1 (Hardering NAT routra, DHCP a SSH servera)

## Nastavenie VirtualBox-u
-	importovať image Debianu
-	vytvoriť tri klony (router, client_1, client_2)
-	nezabudnúť reinicializovať MAC adresy sieťových keriet!
-	network pre **router** nastaviť ako *NAT* a *Internal Network*
-	network pre **client_1** nastaviť ako *Internal Network*
-	network pre **client_2** nastaviť ako *Internal Network*
- `$apt-get update && apt-get upgrade`
- dôležité príkazy:
  - `$ip a`
  - `$ip link show`
  - `$ifconfig`, `$apt-get install net-tools`
  - `$ip addr del XXX.XXX.XXX.XXX/XX dev enp0s3` *// vymazanie IP adresy, ak je náhodou nastavená sekundárna IP adresa pri klientovi*
  
## 1. Smerovanie

### A) Nastavenie sieťových rozhraní

#### Konfiguračný súbor `/etc/network/interfaces`

*Router*

```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
auto enp0s3
iface enp0s3 inet dhcp

# My new network interface
allow-hotplug enp0s8
auto enp0s8
iface enp0s8 inet static
address 10.0.0.1
netmask 255.255.255.0
broadcast 10.0.0.255
network 10.0.0.0
```
**Nové rozhranie aktivujeme pomocou `$ifconfig enp0s8 up`**.

**Reštartovanie sieťových rozhraní `$service networking restart`**.

*Client_1, Client_2*

```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
auto enp0s3
iface enp0s3 inet static
netmask 255.255.255.0
broadcast 10.0.0.255
address 10.0.0.2
gateway 10.0.0.1
```

**Reštartovanie sieťových rozhraní `$service networking restart`**.	

### B) Nastavenie smerovania (iba pre router)

*Trvalé povolenie smerovania na OS Linux*

`$nano /etc/sysctl.conf`
> #net.ipv4.ip_forward=1

`$sysctl -p /etc/sysctl.conf`

*Povolenie natovania na OS Linux*

```bash
$iptables -F
$iptables --table nat --append POSTROUTING --out-interface enp0s3 -j MASQUERADE
$iptables --append FORWARD --in-interface enp0s8 -j ACCEPT
$iptables-save > /etc/iptables/rules.v4 
```

### C) DNS server
Preklad doménového mena na IP adresu pomocou DNS servera nastavíme cez DHCP server v časti **2. DHCP server**.

## 2. DHCP Server

### A) Inštalácia DHCP servera (pre router)

`$apt-get install isc-dhcp-server`

### B) Konfigurácia DHCP servera (pre router)

`$nano /etc/default/isc-dhcp-server`
> INTERFACES=”enp0s8“

```bash
$cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd_backup.conf
$echo “” > /etc/dhcp/dhcpd.conf
$nano /etc/dhcp/dhcpd.conf
```

```bash
subnet 10.0.0.0 netmask 255.255.255.0 {
    range 10.0.0.10 10.0.0.254;      // rozsah IP adries
    option routers 10.0.0.1;   	// info pre klienta o smerovači
    option subnet-mask 255.255.255.0;  // info o maske siete
    option domain-name “google.com“; // meno DNS servera
    option domain-name-servers 8.8.8.8; // adresa DNS servera
    default-lease-time 3600; // nastavenie času pre prideľovanie IP adries
    max-lease-time 7200;
  group {
         host pc1 {					// klient
             hardware ethernet A8:61:A4:99:48:28;	// MAC adr klienta
             fixed-address 10.0.0.2;  		// IP adr klienta
       }
   }                                     
}
```

### C) Manažment DHCP server

```bash
$service isc-dhcp-server status
$service isc-dhcp-server start
$service isc-dhcp-server stop
$service isc-dhcp-server restart
```

### D) Nastavenie DHCP (pre klienta)

`$nano /etc/network/interfaces`

```bash
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug eth0
auto eth0
iface eth0 inet dhcp
```

### E) Logovacie súbory DHCP server

```bash
$more /var/log/syslog
$less /var/log/syslog
```

## 3. SSH

### A) SSH Server

*Inštalácia*

`$apt-get install openssh-server`

*Spustenie/Stop/Reštart*

`$service ssh start/stop/restart/status`

*Konfigurácia*

`$nano /etc/ssh/sshd_config`

### B) SSH Klient

*Inštalácia*

`$apt-get install openssh-client`

*Konfigurácia*

`$nano /etc/ssh/ssh_config`

`$systemctl restart sshd`

*Prihlásenie*

`$ssh meno_pouzivatela@ip_adresa`

## 4. Konfiguračné súbory + hardening
  
