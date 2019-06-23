# Maska

35 urządzeń zmieści się w sieci 64 czyli 2^6, więc potrzebnych jest 6 bitów na część hostową. 32 – 6 = 26 bitów części sieciowej czyli maski.
26 bitów to dziesiętnie 255.255.255.192

Każde laboratorium to oddzielna sieć według schematu:

* Lab009 – 192.168.9.x /26
* Lab13 – 192.168.13.x /26
* Lab204 – 192.168.204.x /26

I tak dalej, to daje rozróżnienie laboratorium po adresie IP.

Komputery w laboratorium oznaczone są w sposób następujący:

* PC1_9 – czyli jest to pierwszy komputer w Lab 009 itd.

Komputery maja adresy IP przypisane w następujący sposób, np.

* C14_204 – 192.168.204.14 (przedostatni oktet to nr laboratorium, ostatni to numer komputera)

W każdym laboratorium brama to ostatni adres z sieci czyli 192.168.nr_lab.62 (broadcast to 63).

Przełączniki sieciowe przyjęto, że mają po 48 portów (w PT są tylko po 24, lecz nie zmienia to konfiguracji). 

# Sieci wifi posiadają adresację:

Parter – 192.168.0.0 /24  Brama 192.168.0.254  SSIS: p0
 WPA2-PSK: p0$12345
 
Piętro 1 – 192.168.1.0 /24 Brama 192.168.1.254 SSIS: p1
WPA2-PSK: p1$12345

Piętro 2 – 192.168.2.0 /24 Brama 192.168.2.254 SSIS: p2
WPA2-PSK: p2$12345

# Konfiguracja serwera dhcp dla wifi na R0

R0> enable
R0# configure terminal
R0(config)# service dhcp (włączenie usługi)
R0(config)# ip dhcp pool wifi_p0 (utworzenie puli adresów dla piętra 0)
R0(dhcp-config)# network 192.168.0.0 255.255.255.0 (przypisanie adresów do puli)
R0(dhcp-config)# default-router 192.168.0.254 (podanie bramy dla hostów)
R0(dhcp-config)# exit
R0(config)# ip dhcp excluded-address 192.168.0.254 (wyłączenie z puli adresu swojego interfejsu, bo I tak juz zajęty)

Podobnie dla R1 I R2.

Przyznana pula adresów to 188.156.220.160 /27

Maska 27 czyli 32-27=5 bitów na hosty, co daje 2^3-32, więc:

* Adres sieci: 188.156.220.160
* Hosty: 188.156.220.161-188.156.220.190
* Broadcast: 188.156.220.191
* Maska: /27 czyli 255.255.255.224

Przyjmujemy, że adres 1.1.1.1/30 jest po stronie ISP na ISP_Router, a adres 1.1.1.2/30 po stronie Border_router do ISP. Adresy są w podsieci serwerowej, a jej brama to 188.156.220.190. Serwery są oznaczone symbolicznie jako dwa serwery o adresach:

* Serwer_1: 188.156.220.161 /27
* Serwer_2: 188.156.220.189 /27

# Połączenia pomiędzy routerami

* R0 - Border_Router: 172.16.0.0 /30
* R1 - Border_Router: 172.16.1.0 /30
* R2 - Border_Router: 172.16.2.0 /30

# Konfiguracja protokołu OSPF na Border_Router

* Border_Router(config)# ip route 0.0.0.0 0.0.0.0 1.1.1.1
(najpierw ustawienie trasy domyślnej)
* Border_Router(config)# router ospf 1
(włączenie protokołu OSPF o process id 1)
* Border_Router (config-router)# network 172.16.0.0 0.0.0.3 area 0
(konfiguracja wszystkich bezpośrednio do routera podłączonych sieci)
* Border_Router (config-router)# network 172.16.1.0 0.0.0.3 area 0
* Border_Router (config-router)# network 172.16.2.0 0.0.0.3 area 0
* Border_Router (config-router)# network 1.1.1.0 0.0.0.3 area 0
* Border_Router (config-router)# network 188.156.220.160 0.0.0.31 area 0
* Border_Router (config-router)# passive-interface Gig 3/0
(wyłączenie wysyłania powiadomień OSPF do podsieci serwerowej, bo tam nie ma już dalej routera – nie ma kto słuchać)
* Border_Router (config-router)# default-information originate
(propagacja trasy domyślenej do wszystkich pozostałych routerów)

# Konfiguracja protokołu OSPF na R0

* R0(config)# router ospf 1
* R0(config-router)# network 192.168.9.0 0.0.0.63 area 0
* R0(config-router)# network 192.168.13.0 0.0.0.63 area 0
* R0(config-router)# network 192.168.14.0 0.0.0.63 area 0
* R0(config-router)# network 192.168.17.0 0.0.0.63 area 0
* R0(config-router)# network 192.168.0.0 0.0.0.255 area 0
* R0(config-router)# network 172.16.0.0 0.0.0.3 area 0
* R0(config-router)# passive-interface Gig 5/0
* R0(config-router)# passive-interface Gig 6/0
* R0(config-router)# passive-interface Gig 7/0
* R0(config-router)# passive-interface Gig 8/0
* R0(config-router)# passive-interface Gig 9/0

# Zezwolenie dostępu z sieci wifi tylko do internet, czyli blokada ruchu do wszystkich sieci wewnętrznych 192.168.0.0/16

Dla R0

* R0(config-router)# access-list 100 deny tcp 192.168.0.0 0.0.0.255 192.168.0.0 0.0.255.255
* R0(config-router)# access-list 100 permit tcp any any

Dla R1
* R0(config-router)# access-list 101 deny tcp 192.168.1.0 0.0.0.255 192.168.0.0 0.0.255.255
* R0(config-router)# access-list 101 permit tcp any any

Dla R2
* R0(config-router)# access-list 102 deny tcp 192.168.2.0 0.0.0.255 192.168.0.0 0.0.255.255
* R0(config-router)# access-list 102 permit tcp any any

# Schemat

![alt text](https://github.com/bartek9105/sk-2019/blob/master/schemat%20zad2.JPG?raw=true)
