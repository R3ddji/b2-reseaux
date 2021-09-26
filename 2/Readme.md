# TP2 : On va router des trucs

## I. ARP

### 1. Echange ARP

**Générer des requêtes ARP**

```
[leo@node1 ~]$ ping 10.2.1.12
PING 10.2.1.12 (10.2.1.12) 56(84) bytes of data.
64 bytes from 10.2.1.12: icmp_seq=1 ttl=64 time=0.988 ms
64 bytes from 10.2.1.12: icmp_seq=2 ttl=64 time=1.07 ms
64 bytes from 10.2.1.12: icmp_seq=3 ttl=64 time=1.11 ms
64 bytes from 10.2.1.12: icmp_seq=4 ttl=64 time=1.05 ms
^C
--- 10.2.1.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3006ms
rtt min/avg/max/mdev = 0.988/1.055/1.105/0.042 ms

```

```
[leo@node1 ~]$ [leo@node1 ~]$ ip n s
10.2.1.12 dev enp0s8 lladdr 08:00:27:15:7f:bd REACHABLE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:1d REACHABLE
```

```
[leo@node2 ~]$ [leo@node2 ~]$ ip n s
10.2.1.11 dev enp0s8 lladdr 08:00:27:bc:c6:86 REACHABLE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:1d DELAY
```

Adresse MAC de node 1 dans la table ARP de node2 : **08:00:27:bc:c6:86**

```
[leo@node2 ~]$ ip a
[...]
link/ether 08:00:27:15:7f:bd brd ff:ff:ff:ff:ff:ff
[...]

```

### 2. Analyse de trames

**Analyse de trames**

```
[leo@node1 ~]$ sudo tcpdump -i enp0s8 -c 10 -w mon_fichier.pcap
[sudo] password for leo:
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
10 packets captured
10 packets received by filter
0 packets dropped by kernel
```
```
[leo@node2 ~]$ ping 10.2.1.11
PING 10.2.1.11 (10.2.1.11) 56(84) bytes of data.
64 bytes from 10.2.1.11: icmp_seq=1 ttl=64 time=0.791 ms
64 bytes from 10.2.1.11: icmp_seq=2 ttl=64 time=1.07 ms
64 bytes from 10.2.1.11: icmp_seq=3 ttl=64 time=0.975 ms
64 bytes from 10.2.1.11: icmp_seq=4 ttl=64 time=1.07 ms
64 bytes from 10.2.1.11: icmp_seq=5 ttl=64 time=1.14 ms
64 bytes from 10.2.1.11: icmp_seq=6 ttl=64 time=1.02 ms
64 bytes from 10.2.1.11: icmp_seq=7 ttl=64 time=1.03 ms
64 bytes from 10.2.1.11: icmp_seq=8 ttl=64 time=1.02 ms
64 bytes from 10.2.1.11: icmp_seq=9 ttl=64 time=1.03 ms
64 bytes from 10.2.1.11: icmp_seq=10 ttl=64 time=0.966 ms
64 bytes from 10.2.1.11: icmp_seq=11 ttl=64 time=1.11 ms
64 bytes from 10.2.1.11: icmp_seq=12 ttl=64 time=1.08 ms
^C
--- 10.2.1.11 ping statistics ---
12 packets transmitted, 12 received, 0% packet loss, time 11021ms
rtt min/avg/max/mdev = 0.791/1.024/1.135/0.084 ms
```

![](./img/wireshark-1.png)

| ordre | type trame  | source                   | destination                |
|-------|-------------|--------------------------|----------------------------|
| 1     | Requête ARP | `PcsCompu_15:7f:bd`      | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | `PcsCompu_bc:c6:86`      | `PcsCompu_15:7f:bd`        |


## II. Routage

### 1. Mise en place du routage

**Activer le routage sur le noeud**

```
[leo@router ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
success
```

**Ajouter les routes statiques nécessaires pour que node1.net1.tp2 et marcel.net2.tp2 puissent se ping**

```
[leo@marcel ~]$ sudo nano /etc/sysconfig/network-scripts/route-enp0s9
10.2.2.12 via 10.2.2.254 dev enp0s9
```

```
sudo nano /etc/sysconfig/network-scripts/route-enp0s8
10.2.2.12 via 10.2.1.254 dev enp0s8
```

```
[leo@node1 ~]$ ping 10.2.2.12
PING 10.2.2.12 (10.2.2.12) 56(84) bytes of data.
64 bytes from 10.2.2.12: icmp_seq=1 ttl=63 time=1.38 ms
64 bytes from 10.2.2.12: icmp_seq=2 ttl=63 time=1.76 ms
64 bytes from 10.2.2.12: icmp_seq=3 ttl=63 time=1.81 ms
^C
--- 10.2.2.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 1.384/1.649/1.806/0.188 ms
```
```
[leo@marcel ~]$ ping 10.2.1.11
PING 10.2.1.11 (10.2.1.11) 56(84) bytes of data.
64 bytes from 10.2.1.11: icmp_seq=1 ttl=63 time=1.15 ms
64 bytes from 10.2.1.11: icmp_seq=2 ttl=63 time=1.93 ms
64 bytes from 10.2.1.11: icmp_seq=3 ttl=63 time=2.00 ms
^C
--- 10.2.1.11 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 1.150/1.692/2.001/0.386 ms
```
**Analyse des échanges ARP**

*On vide les tables ARP*
```
[leo@node1 ~]$ sudo ip neigh flush all

[leo@marcel ~]$ sudo ip neigh flush all

[leo@router ~]$ sudo ip neigh flush all
```

*ping de node1.net1.tp2 vers marcel.net2.tp2*

```
[leo@node1 ~]$ ping 10.2.2.12
PING 10.2.2.12 (10.2.2.12) 56(84) bytes of data.
64 bytes from 10.2.2.12: icmp_seq=1 ttl=63 time=2.25 ms
64 bytes from 10.2.2.12: icmp_seq=2 ttl=63 time=1.92 ms
64 bytes from 10.2.2.12: icmp_seq=3 ttl=63 time=1.84 ms
^C
--- 10.2.2.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 1.842/2.006/2.254/0.185 ms
```
*Table ARP des trois noeuds.*

```
[leo@router ~]$ [leo@router ~]$ ip n s
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:02 DELAY
10.2.1.11 dev enp0s8 lladdr 08:00:27:fe:55:b8 STALE
10.2.2.12 dev enp0s9 lladdr 08:00:27:28:db:d4 STALE
```
```
[leo@node1 ~]$ ip n s
10.2.1.254 dev enp0s8 lladdr 08:00:27:67:a0:39 STALE
10.2.1.1 dev enp0s8 lladdr 0a:00:27:00:00:02 DELAY
```
```
[leo@marcel ~]$ ip n s
10.2.2.1 dev enp0s9 lladdr 0a:00:27:00:00:15 DELAY
10.2.2.254 dev enp0s9 lladdr 08:00:27:7b:c0:b9 STALE
```

On peut voir que le routeur à récupérer les deux ip des machines qui veulent communiquer entre elle, alors que les deux machines eux n'on que la passerelle dans leurs tables arp.

*On vide les table ARP*
```
[leo@node1 ~]$ sudo ip neigh flush all

[leo@marcel ~]$ sudo ip neigh flush all

[leo@router ~]$ sudo ip neigh flush all
```

*Ping node1 vers marcel*
```
ping 10.2.2.12
PING 10.2.2.12 (10.2.2.12) 56(84) bytes of data.
64 bytes from 10.2.2.12: icmp_seq=1 ttl=63 time=1.28 ms
64 bytes from 10.2.2.12: icmp_seq=2 ttl=63 time=1.83 ms
64 bytes from 10.2.2.12: icmp_seq=3 ttl=63 time=1.80 ms
64 bytes from 10.2.2.12: icmp_seq=4 ttl=63 time=1.97 ms
^C
--- 10.2.2.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 1.280/1.718/1.968/0.266 ms
```

*Lancement de tcpdump sur chaque machine.*

```
[leo@router ~]$ sudo tcpdump -i enp0s8 -c 10 -w mon_fichier.pcap not port 22
[sudo] password for leo:
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
10 packets captured
11 packets received by filter
0 packets dropped by kernel
```
```
[leo@node1 ~]$ sudo tcpdump -i enp0s8 -c 10 -w mon_fichier.pcap not port 22
[sudo] password for leo:
dropped privs to tcpdump
tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
10 packets captured
11 packets received by filter
0 packets dropped by kernel
```
```
[leo@marcel ~]$ sudo tcpdump -i enp0s9 -c 10 -w mon_fichier.pcap not port 22
[sudo] password for leo:
dropped privs to tcpdump
tcpdump: listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
10 packets captured
11 packets received by filter
0 packets dropped by kernel
```

![](./img/trame2-1.PNG)
![](./img/trame2-2.PNG)
![](./img/trame2-3.PNG)

| ordre | type trame  | IP source  | MAC source                   | IP destination | MAC destination    |
|-------|-------------|----------- |---------------------------   |----------------|--------------------|
| 1     | Requête ARP | 10.2.2.12  | `marcel` `08:00:27:28:db:d4` | 10.2.2.254     | `08:00:27:7b:c0:b9`|
| 2     | Réponse ARP | 10.2.2.254 | `routeur` `08:00:27:7b:c0:b9`| 10.2.2.12      | `08:00:27:7b:c0:b9`|
| 3     | Requête ARP | 10.2.2.254 | `node1` `08:00:27:7b:c0:b9`  | 10.2.2.11      | `08:00:27:7b:c0:b9`|

### 3. Accès internet

```
$sudo ip route add 10.2.2.254 via 10.2.2.12 dev enp0s8
```
```
$sudo ip route add 10.2.2.254/24 via 10.2.1.11 dev enp0s9
```

```
[leo@node1 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=13.7 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=14.10 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=15.3 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 13.684/14.666/15.316/0.706 ms
```
```
[leo@marcel ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=13.7 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=14.10 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=15.3 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 13.684/14.666/15.316/0.706 ms
```

**Analyse de trames**

```
[leo@node1 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=13.7 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=14.10 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=15.3 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 13.684/14.666/15.316/0.706 ms
```
```
[leo@node1 ~]$ sudo tcpdump -i enp0s8 -c 10 -w mon_fichier.pcap not port 22
```

| ordre | type trame | IP source                   | MAC source                  | IP destination | MAC destination   |
|-------|------------|-----------------------------|-----------------------------|----------------|-------------------|
| 1     | ping       | `node1` `10.2.1.11`         | `node1` `08:00:27:fe:55:b8` | `8.8.8.8`      | 08:00:27:67:a0:39 |
| 2     | pong       | `8.8.8.8`                   |         `08:00:27:67:a0:39` | `10.2.1.11`    | 08:00:27:fe:55:b8 |

## III. DHCP

### 1. Mise en place du serveur DHCP

```
[leo@node1 ~]$ sudo dnf install dhcp-server
```
```
[leo@node1 ~]$ sudo nano /etc/dhcp/dhcpd.conf
```
```
option domain-name  "serveurtp1";
option domain-name-servers  serveurtp1;
default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.2.1.0 netmask 255.255.255.0 {
  range 10.2.1.10 10.2.1.200;
  option routers 10.2.1.254;
  option subnet-mask 255.255.255.0;
  option domain-name-servers 1.1.1.1;

}
```
```
[leo@node1 ~]$ systemctl enable --now dhcpd
```

```
[leo@node1 ~]$firewall-cmd --add-service=dhcp
```