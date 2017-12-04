# Zadanie č. 1 (Hardening NAT routra, DHCP a SSH servera)

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
  
### Router

`/etc/network/interfaces`

```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

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

`/etc/sysctl.conf`

```bash
# ***********************************************************
# Inspiration sources:
#  (1) linoxide.com/how-tos/linux-server-protection
#  (2) wiki.archlinux.org/index.php/sysctl
# ***********************************************************

# Enables IP spoofing attacks protection.
net.ipv4.conf.all.rp_filter = 1

# Enables TCP SYN cookie protection.
# Prevents SYN flood DDoS attacks by testing the validity of SYN packets.
net.ipv4.tcp_syncookies = 1

# Enables packet forwarding.
net.ipv4.ip_forward = 1
#net.ipv6.conf.all.forwarding = 1

# Accept ICMP redirects (we are a router)
net.ipv4.conf.all.accept_redirects = 1
net.ipv6.conf.all.accept_redirects = 1

# Enables to send ICMP redirects (we are a router)
net.ipv4.conf.all.send_redirects = 1
net.ipv4.conf.default.send_redirects = 1
# net.ipv6.conf.all.send_redirects = 1
net.ipv4.conf.all.secure_redirects = 1

# Accepts IP source route packets (we are a router)
net.ipv4.conf.all.accept_source_route = 1
#net.ipv6.conf.all.accept_source_route = 1

# Enables logging spoofed packets.
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Disables the magic system request key.
kernel.sysrq = 0

# Ignoring broadcast request. Prevents Smurf attacks.
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Enables bad error message protection.
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Turns off the tcp_sack.
net.ipv4.tcp_sack = 0

# Turns off the tcp_timestamps.
net.ipv4.tcp_timestamps = 0
```

### DHCP Server

`/etc/default/isc-dhcp-server`

```bash
# Defaults for isc-dhcp-server (sourced by /etc/init.d/isc-dhcp-server)

# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
#DHCPDv4_CONF=/etc/dhcp/dhcpd.conf
#DHCPDv6_CONF=/etc/dhcp/dhcpd6.conf

# Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
#DHCPDv4_PID=/var/run/dhcpd.pid
#DHCPDv6_PID=/var/run/dhcpd6.pid

# Additional options to start dhcpd with.
#	Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
#OPTIONS=""

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#	Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACESv4="enp0s8"
INTERFACESv6=""
```

`/etc/dhcp/dhcpd.conf`

```bash
subnet 10.0.0.0 netmask 255.255.255.0 {
  range 10.0.0.10 10.0.0.254;
  option routers 10.0.0.1;
  option subnet-mask 255.255.255.0;
  option domain-name "google.com";
  option domain-name-servers 8.8.8.8;
  default-lease-time 3600;
  max-lease-time 7200;
  group {
	host pc1 {
	 hardware ethernet 08:00:27:FC:AC:6F; 
	 fixed-address 10.0.0.3;
	}
  }
}
```

### SSH

`/etc/ssh/sshd_config`

```bash
Port 8500 
Banner /etc/ssh/sshd-banner
#AddressFamily any

# Logging
LogLevel VERBOSE
#SyslogFacility AUTH

# Authentication:
ChallengeResponseAuthentication no
LoginGraceTime 1m
MaxAuthTries 4
MaxSessions 5
StrictModes yes
PasswordAuthentication no 
PermitEmptyPasswords no
PermitRootLogin no 
PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
AuthorizedKeysFile	.ssh/authorized_keys .ssh/authorized_keys2
#AuthorizedPrincipalsFile none
#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

HostbasedAuthentication no
IgnoreUserKnownHosts yes
IgnoreRhosts yes

# Kerberos options
KerberosAuthentication no
KerberosOrLocalPasswd yes
KerberosTicketCleanup yes
KerberosGetAFSToken no

# GSSAPI options
GSSAPIAuthentication no
GSSAPICleanupCredentials yes
GSSAPIStrictAcceptorCheck yes
GSSAPIKeyExchange no

UsePAM yes

AllowAgentForwarding yes
AllowTcpForwarding no
GatewayPorts no
X11Forwarding no
X11DisplayOffset 10
X11UseLocalhost yes
PermitTTY yes
PrintMotd no
PrintLastLog yes
TCPKeepAlive no
UseLogin no
UsePrivilegeSeparation sandbox
PermitUserEnvironment no
Compression yes
ClientAliveInterval 600
ClientAliveCountMax 3
UseDNS no
PidFile /var/run/sshd.pid
MaxStartups 10:30:100
PermitTunnel no

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem	sftp	/usr/lib/openssh/sftp-server
```
