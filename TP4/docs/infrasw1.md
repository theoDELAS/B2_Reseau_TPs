## Conf infra-switch1

```
hostname infra-sw1

interface Ethernet0/0
    switchport trunk encapsulation dot1q
    switchport mode trunk

interface Ethernet0/1
    switchport trunk encapsulation dot1q
    switchport mode trunk

interface Ethernet1/0
    switchport access vlan 30
    switchport mode access

interface Ethernet1/1
    switchport access vlan 30
    switchport mode access

interface Vlan1
 no ip address
 shutdown
```

## VLAN
```
infra-sw1#sh vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/2, Et0/3, Et1/2, Et1/3
                                                Et2/0, Et2/1, Et2/2, Et2/3
                                                Et3/0, Et3/1, Et3/2, Et3/3
10   admins                           active
20   guests                           active
30   infra                            active    Et1/0, Et1/1
```