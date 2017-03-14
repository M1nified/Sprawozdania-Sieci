# 021 - Laboratorium - Cisco Catalyst Managed Switches

### Zadanie A: Podstawowa konfiguracja ustawień przełącznika Cisco Catalyst

Przełącznik nierutujący może posiadać swoją tożsamość w sieci IP, tzn. interfejsy IP. Wykorzystujemy tę funkcjonalność w celach administracyjnych (np. konfiguracja przez telnet, HTTP, SSH). Adresów IP nie można przypisać do portów fizycznych. W nierutujących przełącznikach powinien być skonfigurowany interfejs vlan1.
```
Switch(config)#interface vlan1 
Switch (config-if)#ip address 200.200.200.1 255.255.255.0
Switch (config-if)#no shutdown
```


**Konfigurowanie trybu pracy portu przełącznika:**

Każdy z potów może pracować w trybie access (port do DTE), trunk (port do DCE), dynamic. Od wybranego trybu zależy rodzaj ramkowania ethernet.
```
Switch(config)#interface fa 0/5
Switch(config-if)#switchport mode access 
```

Z DTE trzeba uważać, jeśli nie wszystkie przełączniki go wspierają, bo się wysypie przez sprzeczne komunikaty.
```
%DTP-5-DOMAINMISMATCH: Unable to perform trunk negotiation on port Fa0/5 because of VTP domain mismatch. 
Switch(config)#interface fa 0/5 
Switch(config-if)#switchport nonegotiate
```


Sprawdzanie informacji o interfejsach IP:
```
Switch#show ip interface brief 
Switch#show ip interface vlan 1
```


### Zadanie B: Konfigurowanie VLAN przełącznika Cisco Catalyst
```
Switch#show vlan
Switch(config)#vlan 20 % tworzenie VLAN
```

Ręczna modyfikacja puli VLAN nie jest możliwa w trybie CLIENT VTP.
```
Switch(config)#vtp mode transparent
```


**Przypisywanie do VLAN:**
```
Switch(config)#interface fa0/2 / Switch(config)#interface range fa0/15 - 17 
Switch(config-if)#no shutdown 
Switch(config-if)#switchport mode access 
Switch(config-if)#switchport access vlan 20 
```


### Zadanie C: Tworzenie VLAN trunks i tagowanych VLAN w przełącznikach Cisco Catalyst
VLANy tworzymy dzięki ramkom IEEE 802.1Q. Potrzebujemy też trunka, który nie należy do żadnego VLANu.
```
Switch(config-if)#switchport mode trunk 
```

W niektórych wersjach przełączników (np. Catalyst 3550) konieczne jest jawne wytypowanie rodzaju enkapsulacji stosowanej przez port - jako IEEE 802.1Q (zanim będzie możliwe uruchomienie trybu trunk dla portu): 
```
Switch(config-if)#switchport trunk encapsulation dot1q
```


**Przydatne polecenia:**
```
Switch(config-if)#switchport trunk allowed vlan 1-100
Switch(config-if)#switchport trunk allowed vlan remove 10 
Switch#show running-config 
Switch#show interface trunk 
Switch#show interface fa 0/1 switchport 
Switch#show interface fa 0/1 status 
Switch#show vlan
```


**Native VLAN:**
Jeden z VLANów możemy określić jako native i wtedy nie korzystamy z IEEE 802.1Q dla niego. Muszą one być poprawnie skonfigurowane w obu przełącznikach podłączonych do siebie, inaczej się wysypie.
```
Switch(config)#int fa 0/1 
Switch(config-if)#switchport trunk native vlan 10
```


### Zadanie D: VLAN i protokół VTP (Virtual LAN Trunking Protocol) 
Konfigurację VLAN możemy rozsyłać między przełącznikami w ramach tej samej domeny VTP. Przełącznik może pracować jako *server* (rozsyła konfigurację), *transparent* (przekazuje), *client* (odbiera i stosuje konfigurację). Na trasie rozsyłania muszą być tylko kompatybilne przełączniki w tej samej domenie.
```
Switch(config)#vtp domain domena
Switch(config)#vtp password haslo 
```


**Diagnostyka VTP:**
```
Switch#show vtp status
Switch#debug sw-vlan vtp events 
```


**Generalnie procedura jest taka**: podłączamy przełączniki kablem, ustawiamy serwer, ustawiamy domeny, ustawiamy klientów i odpalamy. Jeśli propagujemy VLANy, to na klientach nadmiarowe zostaną usunięte, a ich porty przeniesione najpewniej do VLAN1.
