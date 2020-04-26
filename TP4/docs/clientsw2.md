## Conf client-switch2
```
hostname client-sw2

interface Ethernet0/0
    switchport trunk encapsulation dot1q
    switchport mode trunk

interface Ethernet0/1
    switchport trunk encapsulation dot1q
    switchport mode trunk

interface Ethernet0/2
    switchport trunk encapsulation dot1q
    switchport mode trunk

interface Ethernet1/1
    switchport access vlan 10
    switchport mode access

interface Ethernet1/2
    switchport access vlan 20
    switchport mode access
```

## VLAN

```
client-sw2#sh vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/3, Et1/0, Et1/3, Et2/0
                                                Et2/1, Et2/2, Et2/3, Et3/0
                                                Et3/1, Et3/2, Et3/3
10   admins                           active    Et1/1
20   guests                           active    Et1/2
30   infra                            active
```