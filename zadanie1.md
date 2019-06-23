Adres 172.22.128.0/17

Lan 1 - 500 hostów
Lan 2 - 5000 hostów

Zaczynamy od większej, 5000 zmieści się w podsieci 2^13=8192, czyli potrzeba 13 bitów na część hostową. 
W adresie 17 bitów jest użytych na część sieciową więc pozostaje 32-17 = 15 na część hostową. 
Można to zapisać tak:

NNNNNNNN . NNNNNNNN . NHHHHHHH . HHHHHHHH /17
NNNNNNNN . NNNNNNNN . NSSHHHHH . HHHHHHHH /19

N - część sieciowa 
H - część hostowa
S - utworzone podsieci 15 - 13 = 2

Czyli mamy następujące podsieci:

N1 - 172.22.128.0 /19
N2 - 172.22.160.0 /19
N3 - 172.22.192.0 /19
N4 - 172.22.224.0 /19

N1 przeznaczamy na LAN2 (5000 hostów), a N2 można albo przeznaczyć na LAN1 tak jak jest albo 'okroić' tak, żeby weszło 500 hostów.
Żeby nie marnotrawić adresów okroimu N2 do około 500 maszyn. Żeby weszło 500 hostów potrzeba 2^9=512 czyli 
9 bitów na część hostową. Dzielimy N2.
172.22.160.0 /19

NNNNNNNN . NNNNNNNN . NNNHHHHH . HHHHHHHH /19
NNNNNNNN . NNNNNNNN . NNNSSSSH . HHHHHHHH / 23

"s" JEST 4 CZYLI POWSTANĄ 2^4 PODSIECI, NAM WYSTARCZY JEDNA NA LAN1.
SSSS ustawiamy na 0000 i mamy
172.22.160.0 /23

Reasumując:

# Dla LAN1
* Adres sieci : 172.22.160.0
* Maska 255.255.254.0
* Hosty : 172.22.160.1 - 172.22.161.254
* Broadcast : 172.22.161.255

# Dla LAN2

* Adres sieci : 172.22.128.0
* Maska 255.255.224.0
* Hosty : 172.22.128.1 - 172.22.159.254
* Broadcast : 172.22.159.255

Przyjmujemy adresację:

* ETH0 na PC0 - 192.168.0.1/30 (255.255.255.252)
* ETH1 na PC0 - 172.22.160.1/23 (255.255.252.0)
* ETH2 na PC0 - 172.22.128.1/19 (255.255.224.0)

* ETH0 na PC1 - 172.22.160.2 /23 (255.255.254.0)

* ETH0 na PC2 - 172.22.128.2/19 (255.255.224.0)

# Polecenia

Ustawienie adresu IP dla interfejsu

# Dla PC0

Zakładamy, że interfejs eth0 ma adres 192.168.0.2/30 i po drugiej stronie u ISP jest adres 192.168.0.1/30

nano /etc/network/interfaces  (otwarcie pliku z konfiguracją interfejsów, aby dopisać poniższe linie)	

auto eth0
iface eth0 inet static
address 192.168.0.2
netmask 255.255.255.252

auto eth1
iface eth1 inet static
address 172.22.160.1   
netmask 255.255.254.0 

auto eth2
iface eth2 inet static
address 172.22.128.1   
netmask 255.255.224.0 

service networking restart (restart usług sieciowych)

echo 1 > /proc/sys/net/ipv4/ip_forward
iptables --table nat --append POSTROUTING --out-interface eth0 -j MASQUERADE
iptables --append FORWARD --in-interface eth1 -j ACCEPT
iptables --append FORWARD --in-interface eth2 -j ACCEPT

# Wpisanie przekazywania pakietów na stałe do pliku konfiguracyjnego (żeby działało po restarcie)

nano /etc/sysctl.conf

Odkomentować linię

net.ipv4.ip_forward = 1

Koniec konfiguracji na PC0

# Dla PC1

nano /etc/network/interfaces

auto eth0
iface eth0 inet static
address 172.22.160.2
netmask 255.255.254.0
gateway 172.22.160.1

service networking restart

Koniec konfiguracji PC1

# Dla PC2

nano /etc/network/interfaces

auto eth0
iface eth0 inet static
address 172.22.128.2
netmask 255.255.224.0
gateway 172.22.128.1

service networking restart

Koniec konfiguracji PC2

# Schemat w DIA

![alt text](https://github.com/bartek9105/sk-2019/blob/master/schemat%20zad1.png?raw=true)
