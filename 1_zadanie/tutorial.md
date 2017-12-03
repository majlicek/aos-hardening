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
  - `$ip addr del XXX.XXX.XXX.XXX/XX dev enp0s3` *// vymazanie IP adresy, ak je náhodou nastavená sekundárna IP adresa pri klientovi**
  
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

**Reštartovanie sieťových rozhraní `$service networking restart`**	

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

**Reštartovanie sieťových rozhraní `$service networking restart`**	

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

## 3. SSH

## 4. Konfiguračné súbory + hardening
  
