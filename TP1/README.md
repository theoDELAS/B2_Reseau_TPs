# TP1 : Back to basics

## I. Gather information :

-   Liste des cartes réseau :

    ```
    [root@localhost ~]# ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
        valid_lft forever preferred_lft forever
    2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 08:00:27:ee:5e:9a brd ff:ff:ff:ff:ff:ff
        inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
        valid_lft 85938sec preferred_lft 85938sec
        inet6 fe80::5867:f897:8e7a:7ec2/64 scope link noprefixroute
        valid_lft forever preferred_lft forever
    3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 08:00:27:d2:5d:07 brd ff:ff:ff:ff:ff:ff
        inet 192.168.142.3/24 brd 192.168.142.255 scope global dynamic noprefixroute enp0s8
        valid_lft 738sec preferred_lft 738sec
        inet6 fe80::e86f:ec6e:9867:99eb/64 scope link noprefixroute
        valid_lft forever preferred_lft forever
    ```

-   Bail DHCP hint 1 :

    ```
    [root@localhost NetworkManager]# sudo vi internal-5b37a185-e292-3945-a860-903d1a288bfa-enp0s8.lease
    # This is private data. Do not parse.
    ADDRESS=192.168.142.6
    NETMASK=255.255.255.0
    SERVER_ADDRESS=192.168.142.2
    T1=600
    T2=1050
    LIFETIME=1200
    CLIENTID=01080027ff47e1

    ```

-   Bail DHCP hint 2 :

    ```
    [root@localhost NetworkManager]# sudo nmcli -f DHCP4 con show enp0s3
    DHCP4.OPTION[1]:                        domain_name = auvence.co
    DHCP4.OPTION[2]:                        domain_name_servers = 10.33.10.20 10.33.10.2 8.8.8.8 8.8.4.4
    DHCP4.OPTION[3]:                        expiry = 1569589511
    DHCP4.OPTION[4]:                        ip_address = 10.0.2.15
    DHCP4.OPTION[5]:                        requested_broadcast_address = 1
    DHCP4.OPTION[6]:                        requested_dhcp_server_identifier = 1
    DHCP4.OPTION[7]:                        requested_domain_name = 1
    DHCP4.OPTION[8]:                        requested_domain_name_servers = 1
    DHCP4.OPTION[9]:                        requested_domain_search = 1
    DHCP4.OPTION[10]:                       requested_host_name = 1
    DHCP4.OPTION[11]:                       requested_interface_mtu = 1
    DHCP4.OPTION[12]:                       requested_ms_classless_static_routes = 1
    DHCP4.OPTION[13]:                       requested_nis_domain = 1
    DHCP4.OPTION[14]:                       requested_nis_servers = 1
    DHCP4.OPTION[15]:                       requested_ntp_servers = 1
    DHCP4.OPTION[16]:                       requested_rfc3442_classless_static_routes = 1
    DHCP4.OPTION[17]:                       requested_routers = 1
    DHCP4.OPTION[18]:                       requested_static_routes = 1
    DHCP4.OPTION[19]:                       requested_subnet_mask = 1
    DHCP4.OPTION[20]:                       requested_time_offset = 1
    DHCP4.OPTION[21]:                       requested_wpad = 1
    DHCP4.OPTION[22]:                       routers = 10.0.2.2
    DHCP4.OPTION[23]:                       subnet_mask = 255.255.255.0
    ```

-   Table de routage :

    ```
    [root@localhost ~]# ip route
    default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
    192.168.142.0/24 dev enp0s8 proto kernel scope link src 192.168.142.6 metric 101
    ```

    2ieme ligne : cette route est vers le réseau 10.0.2.15, elle est utilisée pour une connexion locale, la passerelle de cette route est à l'IP 10.0.2.0

    3ieme ligne : cette route est vers le réseau 192.168.142.6 enp0s8, elle est utilisée pour une connexion externe, la passerelle de cette route est à l'IP 192.168.142.0

-   Table ARP :
    ```
    192.168.142.1 dev enp0s8 lladdr 0a:00:27:00:00:04 DELAY
    192.168.142.2 dev enp0s8 lladdr 08:00:27:1c:ee:6d STALE
    10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 STALE
    ```
-   Liste ports en écoute :

    ```
    [root@localhost ~]# ss -tulnp
    Netid State Recv-Q Send-Q Local Address:Port Peer Address:Port
    udp UNCONN 0 0 127.0.0.1:323 0.0.0.0:* users:(("chronyd",pid=740,fd=6))
    udp UNCONN 0 0 192.168.142.6%enp0s8:68 0.0.0.0:* users:(("NetworkManager",pid=771,fd=22))
    udp UNCONN 0 0 10.0.2.15%enp0s3:68 0.0.0.0:* users:(("NetworkManager",pid=771,fd=18))
    udp UNCONN 0 0 [::1]:323 [::]:* users:(("chronyd",pid=740,fd=7))
    tcp LISTEN 0 128 0.0.0.0:22 0.0.0.0:* users:(("sshd",pid=791,fd=6))
    tcp LISTEN 0 128 [::]:22 [::]:* users:(("sshd",pid=791,fd=8))
    ```

    On voit bien que le serveur SSH écoute bien sur le port 22

-   Liste des DNS :
    Pour effectuer une requête DNS, j'ai utilisé la commande `dig`. D'abord du l'installer avec `yum install bind-utils`

    ```
    reddit.map.fastly.net.  29      IN      A       151.101.193.140
    reddit.map.fastly.net.  29      IN      A       151.101.65.140
    reddit.map.fastly.net.  29      IN      A       151.101.1.140
    reddit.map.fastly.net.  29      IN      A       151.101.129.140
    ```

-   État actuel du firewall :
    ```
    [root@localhost ~]# [root@localhost ~]# firewall-cmd --list-all
    public (active)
    target: default
    icmp-block-inversion: no
    interfaces: enp0s3 enp0s8
    sources:
    services: cockpit dhcpv6-client ssh
    ports: 22/tcp
    protocols:
    masquerade: no
    forward-ports:
    source-ports:
    icmp-blocks:
    rich rules:
    ```

## II. Edit configuration

### 1. Configuration cartes réseau

-   Modification IP enp0s8 en 192.168.142.10

    ```
    [root@localhost network-scripts]# [root@localhost network-scripts]# cat ifcfg-enp0s8
    TYPE=Ethernet
    ...
    DEVICE=enp0s8
    ONBOOT=yes
    IPADDR=192.168.142.10
    ...
    ```

-   Ajouter une deuxieme carte réseau depuis Vbox
    Ajout de enp0s9 avec l'IP statique 192.168.1.10
    Vérification des changements avec un `ip a` :

    ```
    [root@localhost network-scripts]# ip a
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
        valid_lft forever preferred_lft forever
    2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 08:00:27:92:89:27 brd ff:ff:ff:ff:ff:ff
        inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
        valid_lft 84698sec preferred_lft 84698sec
        inet6 fe80::5867:f897:8e7a:7ec2/64 scope link noprefixroute
        valid_lft forever preferred_lft forever
    3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 08:00:27:ff:47:e1 brd ff:ff:ff:ff:ff:ff
        inet 192.168.142.10/24 brd 192.168.142.255 scope global noprefixroute enp0s8
        valid_lft forever preferred_lft forever
        inet6 fe80::f93:a705:36d2:ec66/64 scope link noprefixroute
        valid_lft forever preferred_lft forever
    4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 08:00:27:b3:6a:a6 brd ff:ff:ff:ff:ff:ff
        inet 192.168.1.10/24 brd 192.168.1.255 scope global noprefixroute enp0s9
        valid_lft forever preferred_lft forever
        inet6 fe80::4cec:b247:63e6:15b0/64 scope link noprefixroute
        valid_lft forever preferred_lft forever
    ```

-   `ip route` :

    ```
    [root@localhost network-scripts]# [root@localhost network-scripts]# ip route
    default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
    192.168.1.0/24 dev enp0s9 proto kernel scope link src 192.168.1.10 metric 102
    192.168.142.0/24 dev enp0s8 proto kernel scope link src 192.168.142.10 metric 101
    ```

### 2. Serveur SSH

-   Modifier config systeme pour que le serveur ssh tourne sur le port 2222 ET fermer l'ancien port 22

    Le port SSH tourne bien sur le port 2222

    ```
    [root@localhost ~]# ss -tulnp
    Netid   State     Recv-Q    Send-Q           Local Address:Port    Peer  Address:Port
        ...

       users:(("chronyd",pid=742,fd=7))
    tcp     LISTEN    0         128                    0.0.0.0:2222            0.0.0.0:*       users:(("sshd",pid=1680,fd=5))
    tcp     LISTEN    0         128                       [::]:2222               [::]:*       users:(("sshd",pid=1680,fd=7))
    ```

    Le port 22 à bien était supprimé

    ```
    [root@localhost ~]# firewall-cmd --list-ports
    2222/tcp
    ```

# III. Routage simple

**VM1 +---+ 192.168.1.0 +---+ router +---+ 192.168.2.0 +---+ VM2**

-   Création de 2 VMs nommées respectivement `VM1` et `VM2`.  
    VM1 : `192.168.1.1` net1  
    VM2 : `192.168.2.2` net2  
    Création d'une autre VM qui servira de routeur.  
    Router net1 : `192.168.1.2`  
    Router net2 : `192.168.2.3`

-   J'ai renommé chaque VM dans leur fichier `/etc/hostname`

-   Dans leur fichier `/etc/hosts` j'ai ensuite assigné leur IP à leur nom  
    Exemple VM1 :

    ```
    [theo@VM1 ~]# cat /etc/hosts
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    192.168.2.2 VM2
    ```
