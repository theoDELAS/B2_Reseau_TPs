# TP3 : Routage INTER-VLAN + mise en situation
## I. Mise en place du schéma  
```
             +--+
             |R1|
             +-++
               |
               |                    +---+
               |          +---------+PC4|
+---+        +-+-+      +---+       +---+
|PC1+--------+SW1+------+SW2|
+---+        +-+-+      +-+--+
               |          |  |
               |          |  +------+--+
               |          |         |P1|
             +-+-+      +-+-+       +--+
             |PC2|      |PC3|
             +---+      +---+
```


**Tableau des réseaux utilisés**

Réseau | Adresse | VLAN | Description
--- | --- | --- | ---
`net1` | `10.3.10.0/24` | 10 | Utilisateurs
`net2` | `10.3.20.0/24` | 20 | Admins
`net3` | `10.3.30.0/24` | 30 | Visiteurs
`netP` | `10.3.40.0/24` | 40 | Imprimantes

**Tableau d'adressage**

Machine | VLAN | IP `net1` | IP `net2` | IP `net3` |  IP `netP`
--- | --- | --- | --- | --- | ---
PC1 | 10 | `10.3.10.1/24` | x | x | x
PC2 | 20 | x | `10.3.20.2/24` | x | x | x
PC3 | 20 | x | `10.3.20.3/24` | x | x | x
PC4 | 30 | x | x |  `10.3.30.4/24` | x | x
P1 | 40 | x | x | x | `10.3.40.1/24` 
R1 | x |  `10.3.10.254/24` | `10.3.20.254/24` | `10.3.30.254/24` | `10.3.40.254/24` 

**Qui peut joindre qui ?**

✅ = peuvent se joindre
❌ = ne peuvent pas se joindre

Réseaux | `net1` |  `net2` |  `net3` |  `netP`
--- | --- | --- | --- | ---
 `net1` | ✅ | ❌ | ❌ | ❌
 `net2` | ❌ | ✅ | ✅ | ✅
 `net3` | ❌ | ✅ | ✅ | ✅
 `netP` | ❌ | ✅ | ✅ | ✅

Après avoir setup this shit je peux faire mes `ping` :  
- `PC1` vers `PC2` (et tous le monde) :  
    ```
    PC1> ping 10.3.20.2
    host (10.3.10.254) not reachable
    ```  
    Le vlan 10 n'est pas autorisé entre le switch1 et le routeur donc le ping ne passe pas.  
- `PC2` vers `PC4` :  
    ```
    PC2> ping 10.3.30.4
    10.3.30.4 icmp_seq=1 timeout
    10.3.30.4 icmp_seq=2 timeout
    84 bytes from 10.3.30.4 icmp_seq=3 ttl=63 time=14.926 ms
    84 bytes from 10.3.30.4 icmp_seq=4 ttl=63 time=17.600 ms
    ```  
    `10.3.30.4 icmp_seq=1 timeout` : cette ligne apparaît car la requête ARP lors du premier `ping` se fait en meme temps que le `ping`, donc la première trame est "perdue".  

- `PC2` vers `P` :  
    ```
    PC2> ping 10.3.40.1
    10.3.40.1 icmp_seq=1 timeout
    10.3.40.1 icmp_seq=2 timeout
    84 bytes from 10.3.40.1 icmp_seq=3 ttl=63 time=18.934 ms
    84 bytes from 10.3.40.1 icmp_seq=4 ttl=63 time=12.477 ms
    ```

- `PC2` vers `PC3` :  
    ```
    PC2> ping 10.3.20.3
    84 bytes from 10.3.20.3 icmp_seq=1 ttl=64 time=0.287 ms
    84 bytes from 10.3.20.3 icmp_seq=2 ttl=64 time=0.505 ms
    84 bytes from 10.3.20.3 icmp_seq=3 ttl=64 time=0.479 ms
    ```

- `PC3` vers `PC4` :  
    ```
    PC3> ping 10.3.30.4
    84 bytes from 10.3.30.4 icmp_seq=1 ttl=63 time=14.584 ms
    84 bytes from 10.3.30.4 icmp_seq=2 ttl=63 time=15.772 ms
    84 bytes from 10.3.30.4 icmp_seq=3 ttl=63 time=12.007 ms
    ```  

- `PC3` vers `P` :  
    ```
    PC3> ping 10.3.40.1
    84 bytes from 10.3.40.1 icmp_seq=1 ttl=63 time=18.813 ms
    84 bytes from 10.3.40.1 icmp_seq=2 ttl=63 time=13.658 ms
    84 bytes from 10.3.40.1 icmp_seq=3 ttl=63 time=20.472 ms
    ```  

- `PC4` vers `P` :
    ```
    PC4> ping 10.3.40.1
    10.3.40.1 icmp_seq=1 timeout
    10.3.40.1 icmp_seq=2 timeout
    84 bytes from 10.3.40.1 icmp_seq=3 ttl=63 time=14.980 ms
    84 bytes from 10.3.40.1 icmp_seq=4 ttl=63 time=14.565 ms
    ```  
Tout le monde ping tout le monde (sauf `PC1` qui lui est isolé) en utilisant comme gateway le routeur `R1`.  

## II. Cas concret