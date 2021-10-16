# TP3 : Progressons vers le réseau d'infrastructure

## I. (mini)Architecture réseau

### 1. Adressage

| Nom du réseau | Adresse du réseau | Masque            | Nombre de clients possibles | Adresse passerelle | [Adresse broadcast]|
|---------------|-------------------|-------------------|-----------------------------|--------------------|--------------------|
| `client1`     | `10.3.1.128`      | `255.255.255.192` | 62                          | `10.3.1.190`       | `10.3.1.191`       |
| `server1`     | `10.3.1.0`        | `255.255.255.128` | 125                         | `10.3.2.126`       | `10.3.1.127`       |
| `server2`     | `10.3.1.192`      | `255.255.255.240` | 14                          | `10.3.3.206`       | `10.3.1.207`       |
 
### 2. Routeur

🖥️ **VM router.tp3**

🌞 **Vous pouvez d'ores-et-déjà créer le routeur. Pour celui-ci, vous me prouverez que :**

```
[leo@router ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4f:6e:a7 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 86239sec preferred_lft 86239sec
    inet6 fe80::a00:27ff:fe4f:6ea7/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:77:c9:3c brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.190/24 brd 10.3.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe77:c93c/64 scope link
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:13:34:e2 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.126/24 brd 10.3.1.255 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe13:34e2/64 scope link
       valid_lft forever preferred_lft forever
5: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:93:fb:d8 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.206/24 brd 10.3.1.255 scope global noprefixroute enp0s10
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe93:fbd8/64 scope link
       valid_lft forever preferred_lft forever
```
```
[leo@router ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=20.9 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=21.7 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=20.6 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 20.598/21.048/21.658/0.477 ms
```
```
[leo@router ~]$ hostname
router.tp3
```
```
[leo@router ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
```

# II. Services d'infra

## 1. Serveur DHCP

🖥️ **VM `dhcp.client1.tp3`**

🌞 **Mettre en place une machine qui fera office de serveur DHCP** dans le réseau `client1`. Elle devra :

```
[leo@dhcp ~]$ hostname
dhcp.client1.tp3
```
```
[leo@dhcp ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d6:23:7a brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.130/26 brd 10.3.1.191 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed6:237a/64 scope link 
       valid_lft forever preferred_lft forever

```
```
[leo@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.3.1.130 netmask 255.255.255.192 {
  range 10.3.1.130 10.3.1.189;
  option routers 10.3.1.190;
  option subnet-mask 255.255.255.192;
  option domain-name-servers 10.3.120;
 
}
```
```
[leo@dhcp ~]$ sudo systemctl start dhcpd
```
```
[leo@dhcp ~]$ sudo systemctl enable dhcpd
```
📁 **Fichier `dhcpd.conf`**

---

🖥️ **VM marcel.client1.tp3**

🌞 **Mettre en place un client dans le réseau `client1`**

```
[leo@marcel ~]$ hostname
marcel.client1.tp3
```
```
[leo@marcel ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:76:16:45 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.132/26 brd 10.3.1.191 scope global dynamic noprefixroute enp0s8
       valid_lft 497sec preferred_lft 497sec
    inet6 fe80::a00:27ff:fe76:1645/64 scope link
       valid_lft forever preferred_lft forever
```
```
[leo@marcel ~]$ sudo cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
NAME=enp0s8
DEVICE=enp0s8

BOOTPROTO=dhcp
ONBOOT=yes
```
```
[leo@marcel ~]$ sudo cat /etc/sysconfig/network
GATEWAY=10.3.1.190
```1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d6:23:7a brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.130/26 brd 10.3.1.191 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed6:237a/64 scope link 
       valid_lft forever preferred_lft forever
```
[leo@marcel ~]$ sudo cat /etc/resolv.conf
nameserver 1.1.1.1
```

🌞 **Depuis `marcel.client1.tp3`**

```
[leo@marcel ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=26.4 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=26.6 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=26.6 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 26.441/26.530/26.598/0.065 ms
```
```
[leo@marcel ~]$ ping google.com
PING google.com (216.58.213.142) 56(84) bytes of data.
64 bytes from par21s03-in-f142.1e100.net (216.58.213.142): icmp_seq=1 ttl=113 time=21.9 ms
64 bytes from par21s03-in-f142.1e100.net (216.58.213.142): icmp_seq=2 ttl=113 time=27.6 ms
64 bytes from par21s03-in-f142.1e100.net (216.58.213.142): icmp_seq=3 ttl=113 time=24.7 ms
64 bytes from par21s03-in-f142.1e100.net (216.58.213.142): icmp_seq=4 ttl=113 time=24.9 ms
^C
--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3008ms
rtt min/avg/max/mdev = 21.868/24.765/27.646/2.049 ms
```

```
[leo@marcel ~]$ traceroute google.com
traceroute to google.com (172.217.19.238), 30 hops max, 60 byte packets
 1  _gateway (10.3.1.190)  2.188 ms  2.133 ms  2.102 ms
 2  10.0.2.2 (10.0.2.2)  1.922 ms  1.628 ms  1.437 ms
```

## 2. Serveur DNS

### B. SETUP copain

🖥️ **VM dns1.server1.tp3**

🌞 **Mettre en place une machine qui fera office de serveur DNS**
```
[leo@dns1 ~]$ hostname
dns1.server1.tp3
```
```
[leo@dns1 ~]$ sudo cat /etc/resolv.conf
nameserver 1.1.1.1
```
```
[leo@dns1 ~]$ sudo cat /etc/named.conf
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
        listen-on port 53 { any; };
        listen-on-v6 port 53 { any; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { any; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-enable yes;
        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

zone "server1.tp3" IN {
        type master;
        file "/var/named/server1.tp3.forward";
        allow-update { none; };
};

zone "server2.tp3" IN {
        type master;
        file "/var/named/server2.tp3.forward";
        allow-update { none; };

};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```
```
[leo@dns1 ~]$ sudo cat /var/named/server1.tp3.forward
$TTL 86400
@   IN  SOA dns1.server1.tp3. admin.server1.tp3. (
        2021062301   ; serial
        3600         ; refresh
        1800         ; retry
        604800       ; expire
        86400 )      ; minimum TTL
;
; define nameservers
    IN  NS  dns1.server1.tp3.
;
; DNS Server IP addresses and hostnames
dns1 IN  A   10.3.1.20
;
;client records
router IN  A   10.3.1.126

```
```
[leo@dns1 ~]$ sudo cat /var/named/server2.tp3.forward
$TTL 86400
@   IN  SOA dns1.server1.tp3. admin.server1.tp3. (
        2021062301   ; serial
        3600         ; refresh
        1800         ; retry
        604800       ; expire
        86400 )      ; minimum TTL
;
; define nameservers
    IN  NS  dns1.server1.tp3.
;
; DNS Server IP addresses and hostnames
dns1 IN  A   10.3.1.20
;
;client records
router IN  A   10.3.1.206
```
🌞 **Tester le DNS depuis `marcel.client1.tp3`**
```
[leo@marcel ~]$ sudo cat /etc/resolv.conf
nameserver 10.3.1.20
```
- essayez une résolution de nom avec `dig`
  - une résolution de nom classique
    - `dig <NOM>` pour obtenir l'IP associée à un nom
    - on teste la zone forward
- prouvez que c'est bien votre serveur DNS qui répond pour chaque `dig`

> Pour rappel, avec `dig`on peut préciser directement sur la ligne de commande à quel serveur poser la question avec  le caractère `@`. Par exemple `dig google.com @8.8.8.8` permet de préciser explicitement qu'on veut demander à `8.8.8.8` à quelle IP se trouve `google.com`.

⚠️ **NOTE : A partir de maintenant, vous devrez modifier les fichiers de zone pour chaque nouvelle machine ajoutée (voir [📝**checklist**📝](#checklist).**

🌞 Configurez l'utilisation du serveur DNS sur TOUS vos noeuds

- les serveurs, on le fait à la main
- les clients, c'est fait *via* DHCP

⚠️**A partir de maintenant, vous n'utiliserez plus DU TOUT les IPs pour communiquer entre vos machines, mais uniquement leurs noms.**

## 3. Get deeper

On va affiner un peu la configuration des outils mis en place.

### A. DNS forwarder

🌞 **Affiner la configuration du DNS**

- faites en sorte que votre DNS soit désormais aussi un forwarder DNS
- c'est à dire que s'il ne connaît pas un nom, il ira poser la question à quelqu'un d'autre

> Hint : c'est la clause `recursion` dans le fichier `/etc/named.conf`. Et c'est déjà activé par défaut en fait.

🌞 Test !

- vérifier depuis `marcel.client1.tp3` que vous pouvez résoudre des noms publics comme `google.com` en utilisant votre propre serveur DNS (commande `dig`)
- pour que ça fonctionne, il faut que `dns1.server1.tp3` soit lui-même capable de résoudre des noms, avec `1.1.1.1` par exemple

### B. On revient sur la conf du DHCP

🖥️ **VM johnny.client1.tp3**

🌞 **Affiner la configuration du DHCP**

- faites en sorte que votre DHCP donne désormais l'adresse de votre serveur DNS aux clients
- créer un nouveau client `johnny.client1.tp3` qui récupère son IP, et toutes les nouvelles infos, en DHCP

---

# Entracte

**YO !**

**On se pose deux minutes pour apprécier le travail réalisé.**

A ce stade vous avez :

- un routeur qui permet aux machines d'acheminer leur trafic entre les réseaux
  - entre les LANs
  - vers internet
- un DHCP qui filent des infos à vos clients
  - une IP, une route par défaut, l'adresse d'un DNS
- un DNS
  - qui résout tous les noms localement

Vous le sentez là que ça commence à prendre forme oupa ? Tous les réseaux du monde sont fichus comme ça c:

**Allez, prend un cookie tu l'as mérité : 🍪**

![You get a cookie](./pic/you-get-a-cookie-ricky-berwick.gif "You get a cookie")

---

# III. Services métier

Ce qu'on appelle un "service métier", à l'inverse du service d'infra, c'est un truc que l'utilisateur final veut consommer. On le dit "métier" car dans une entreprise, c'est ce service qui sert le métier de l'entreprise, son coeur d'activité.

Par exemple, pour une agence de développement web, l'un des services métier, bah c'est les serveurs web qui sont déployés pour porter les sites web développés par la boîte.

> On est en 2021, le serveur web c'est le cas d'école. Y'en a partout. Mais genre + que vous imaginez, même nous les admins, on utilise de plus en plus d'interfaces web, au détriment de la ligne de commande.

## 1. Serveur Web

🖥️ **VM web1.server2.tp3**

🌞 **Setup d'une nouvelle machine, qui sera un serveur Web, une belle appli pour nos clients**

- réseau `server2`
- hello `web1.server2.tp3` !
- [📝**checklist**📝](#checklist)
- vous utiliserez le serveur web que vous voudrez, le but c'est d'avoir un serveur web fast, pas d'y passer 1000 ans :)
  - réutilisez [votre serveur Web du TP1 Linux](https://gitlab.com/it4lik/b2-linux-2021/-/tree/main/tp/1#2-cr%C3%A9ation-de-service)
  - ou montez un bête NGINX avec la page d'accueil (ça se limite à un `dnf install` puis `systemctl start nginx`)
  - ou ce que vous voulez, du moment que c'est fast
  - **dans tous les cas, n'oubliez pas d'ouvrir le port associé dans le firewall** pour que le serveur web soit joignable

> Une bête page HTML fera l'affaire. On est pas là pour faire du design. Et vous prenez pas la tête pour l'install, appelez-moi vite s'il faut, on est pas en système ni en Linux non plus. On est en réseau, on veut juste un serveur web derrière un port :)

🌞 **Test test test et re-test**

- testez que votre serveur web est accessible depuis `marcel.client1.tp3`
  - utilisez la commande `curl` pour effectuer des requêtes HTTP

---

👋👋👋 **HEY ! C'est beau là.** On a un client qui consomme un serveur web, avec toutes les infos récupérées en DHCP, DNS, blablabla + un routeur maison.  

Je sais que ça se voit pas trop avec les `curl`. Vous pourriez installer un Ubuntu graphique sur le client si vous voulez vous y croire à fond, avec un ptit Firefox, Google Chrome ou whatever.  

**Mais là on y est**, vous avez un ptit réseau, un vrai, avec tout ce qu'il faut.

## 2. Partage de fichiers

### A. L'introduction wola

Dans cette partie, on va monter un serveur NFS. C'est un serveur qui servira à partager des fichiers à travers le réseau.  

**En d'autres termes, certaines machines pourront accéder à un dossier, à travers le réseau.**

Dans notre cas, on va faire en sorte que notre serveur web `web1.server2.tp3` accède à un partage de fichier.

> Dans un cas réel, ce partage de fichiers peut héberger le site web servi par notre serveur Web par exemple.  
Ou, de manière plus récurrente, notre serveur Web peut effectuer des sauvegardes dans ce dossier. Ainsi, le serveur de partage de fichiers devient le serveur qui centralise les sauvegardes.

### B. Le setup wola

🖥️ **VM nfs1.server2.tp3**

🌞 **Setup d'une nouvelle machine, qui sera un serveur NFS**

- réseau `server2`
- son nooooom : `nfs1.server2.tp3` !
- [📝**checklist**📝](#checklist)
- je crois que vous commencez à connaître la chanson... Google "nfs server rocky linux"
  - [ce lien me semble être particulièrement simple et concis](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=nfs&f=1)
- **vous partagerez un dossier créé à cet effet : `/srv/nfs_share/`**

🌞 **Configuration du client NFS**

- effectuez de la configuration sur `web1.server2.tp3` pour qu'il accède au partage NFS
- le partage NFS devra être monté dans `/srv/nfs/`
- [sur le même site, y'a ça](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=nfs&f=2)

🌞 **TEEEEST**

- tester que vous pouvez lire et écrire dans le dossier `/srv/nfs` depuis `web1.server2.tp3`
- vous devriez voir les modifications du côté de  `nfs1.server2.tp3` dans le dossier `/srv/nfs_share/`

# IV. Un peu de théorie : TCP et UDP

Bon bah avec tous ces services, on a de la matière pour bosser sur TCP et UDP. P'tite partie technique pure avant de conclure.

🌞 **Déterminer, pour chacun de ces protocoles, s'ils sont encapsulés dans du TCP ou de l'UDP :**

- SSH
- HTTP
- DNS
- NFS

📁 **Captures réseau `tp3_ssh.pcap`, `tp3_http.pcap`, `tp3_dns.pcap` et `tp3_nfs.pcap`**

> **Prenez le temps** de réfléchir à pourquoi on a utilisé TCP ou UDP pour transmettre tel ou tel protocole. Réfléchissez à quoi servent chacun de ces protocoles, et de ce qu'on a besoin qu'ils réalisent.

🌞 **Expliquez-moi pourquoi je ne pose pas la question pour DHCP.**

🌞 **Capturez et mettez en évidence un *3-way handshake***

📁 **Capture réseau `tp3_3way.pcap`**

# V. El final

🌞 **Bah j'veux un schéma.**

- réalisé avec l'outil de votre choix
- un schéma clair qui représente
  - les réseaux
    - les adresses de réseau devront être visibles
  - toutes les machines, avec leurs noms
  - devront figurer les IPs de toutes les interfaces réseau du schéma
  - pour les serveurs : une indication de quel port est ouvert
- vous représenterez les host-only comme des switches
- dans le rendu, mettez moi ici à la fin :
  - le schéma
  - le 🗃️ tableau des réseaux 🗃️
  - le 🗃️ tableau d'adressage 🗃️
    - on appelle ça aussi un "plan d'adressage IP" :)

> J'vous le dis direct, un schéma moche avec Paint c'est -5 Points. Je vous recommande [draw.io](http://draw.io).

🌞 **Et j'veux des fichiers aussi, tous les fichiers de conf du DNS**

- 📁 Fichiers de zone
- 📁 Fichier de conf principal DNS `named.conf`
- faites ça à peu près propre dans le rendu, que j'ai plus qu'à cliquer pour arriver sur le fichier ce serait top