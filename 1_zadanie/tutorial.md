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

### Nastavenie sieťových rozhraní

### Nastavenie smerovania (iba pre router)

Preklad doménového mena na IP adresu pomocou DNS servera nastavíme cez DHCP server v časti **2. DHCP server**.

### DNS server

## 2. DHCP Server

## 3. SSH

## 4. Konfiguračné súbory + hardening
  
