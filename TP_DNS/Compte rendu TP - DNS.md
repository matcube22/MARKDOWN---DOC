# Configuration des machines

## Configuration de G34
```
# On nettoye les configurations résiduelles
ip addr flush eth0

# Ajout des addresses IP
ip addr add 10.34.121.1/24 dev eth0

# Ajout d'une route par défaut
ip route add default via 10.34.121.254
```

## Configuration de S34
```
# On nettoye les configurations résiduelles
ip addr flush eth0
ip addr flush eth1
ip addr flush eth2

# Ajout des addresses IP
ip addr add 10.254.34.1/16 dev eth0
ip addr add 10.34.211.1/24 dev eth1
ip addr add 10.34.121.254/24 dev eth2

# Ajout d'une route par défaut
ip route add default via 10.254.0.254

# On active ip forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward
```

## Configuration de D34
```
# On nettoye les configurations résiduelles
ip addr flush eth0

# Ajout des addresses IP
ip addr add 10.34.221.1/24 dev eth0

# Ajout d'une route par défaut
ip route add default via 10.34.221.254
```

## Test de connectivité
On test les remises directes, puis opposées et enfin **148.60.235.253**
```
# Remises directes
Ping D34 -> S34 : OK
Ping G34 -> S34 : OK
Ping S34 -> 10.254.0.254

# Remises opposées
Ping G34 -> 10.34.221.254 : OK
Ping D34 -> 10.34.121.254 : OK
... etc

# Ping STP
Ping G34 -> STP : OK
Ping S34 -> STP : OK
Ping D34 -> STP : OK
```

# Configuration de base du serveur DNS de S34
## Visualisation de named.conf
```
root@S34:~# cat /etc/bind/named.conf
// This is the primary configuration file for the BIND DNS server named.
//
// Please read /usr/share/doc/bind9/README.Debian.gz for information on the
// structure of BIND configuration files in Debian, *BEFORE* you customize
// this configuration file.
//
// If you are just adding zones, please do that in /etc/bind/named.conf.local

include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```

**named.conf.options** : Ce fichier contient des paramètres généraux du serveur, par exemple le port d'écoute du serveur
**named.conf.local** : Ce fichier contient la configuration locale du serveur DNS, on y déclare les zones associées au domaine. Il reste au niveau du LAN
**named.conf.default-zones** : Ce fichier permet de déclarer les zones que nous gérons, et dont on peut apporter une réponse, s'il y a des requêtes externes.

## Visualisation de /etc/bind/named.conf.options
```
root@S34:~# cat /etc/bind/named.conf.options
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      0.0.0.0;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        listen-on-v6 { any; };
};
root@S34:~#
```

## Visualisation /etc/bind/named.conf.local
```
root@S34:~# cat /etc/bind/named.conf.local
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

root@S34:~#
```

## Visualisation de /etc/bind/named.conf.default-zones
```
root@S34:~# cat /etc/bind/named.conf.local
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

root@S34:~# cat /etc/bind/named.conf.default-zones
// prime the server with knowledge of the root servers
zone "." {
        type hint;
        file "/usr/share/dns/root.hints";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
        type master;
        file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
        type master;
        file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
        type master;
        file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
        type master;
        file "/etc/bind/db.255";
};


root@S34:~#
```

## Configuration du serveur DNS S34 en mode "forward"
On édite le fichier **named.conf.options**
```
root@S34:~# cat /etc/bind/named.conf.options
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
              0.0.0.0;
        };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation no;

        listen-on-v6 { any; };
        listen-on { any; };
        allow-query { any; };
        allow-recursion { any;};
};
root@S34:~#
```

Nous avons ajouter ou modifier ces paramètres dans le fichier.
```
forwarders {
     0.0.0.0;
   };

dnssec-validation no;
forward only;
listen-on-v6 { any; };
listen-on { any; };
allow-query { any; };
allow-recursion { any;};
```

**Démarrer le service**:
```
systemctl restart bind9
```

On peut vérifier que le processus est bien lancer:
```
systemctl status bind9
```
sortie de commandes:
```
root@S34:~# systemctl status bind9
● named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-10-25 14:38:07 CEST; 2min 35s ago
       Docs: man:named(8)
   Main PID: 113681 (named)
      Tasks: 14 (limit: 9411)
     Memory: 30.9M
        CPU: 133ms
     CGroup: /system.slice/named.service
             └─113681 /usr/sbin/named -f -u bind

root@S34:~#
```

Voir les logs **/var/log/syslog**
```
less /var/log/syslog
# OU la commande
tail -f /var/log/syslog
```

Sortie de commande 'extrait':
```
Oct 25 08:22:33 s34 named[833]: using default UDP/IPv4 port range: [32768, 60999]
Oct 25 08:22:33 s34 named[833]: using default UDP/IPv6 port range: [32768, 60999]
Oct 25 08:22:33 s34 named[833]: listening on IPv4 interface lo, 127.0.0.1#53
Oct 25 08:22:33 s34 named[833]: listening on IPv4 interface eth2, 10.34.2.1#53
Oct 25 08:22:33 s34 named[833]: listening on IPv4 interface eth0, 10.254.34.1#53
Oct 25 08:22:33 s34 named[833]: listening on IPv4 interface eth1, 10.34.1.1#53
Oct 25 08:22:33 s34 named[833]: IPv6 socket API is incomplete; explicitly binding to each IPv6 address separately
```

Nous faison **dig google.fr** pour vérifier le bon fonctionnement
```
root@S34:~# dig google.fr

; <<>> DiG 9.16.27-Debian <<>> google.fr
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22732
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 1e030f62ee5cee46010000006357da4057cac6d80ea8ad26 (good)
;; QUESTION SECTION:
;google.fr.                     IN      A

;; ANSWER SECTION:
google.fr.              300     IN      A       216.58.214.67

;; Query time: 75 msec
;; SERVER: ::1#53(::1)
;; WHEN: Tue Oct 25 14:44:48 CEST 2022
;; MSG SIZE  rcvd: 82

root@S34:~#
```

On voit bien dans le champs **QUERY** la requête DNS, puis dans le champs **ANSWER** notre réponse à la question.

On regarde la liste des processus:
```
root@S34:~# ps -ef | grep named
root       88468    3220  0 14:34 pts/1    00:00:04 pluma ./named.conf.options
bind      113681       1  0 14:38 ?        00:00:00 /usr/sbin/named -f -u bind
root      184386    2414  0 14:47 pts/0    00:00:00 grep named
root@S34:~#
```

**grep** nous permet de filtrer uniquement le processus qui a pour nom *named*.

## Amélioration de la configuration
C'est le fichier **root.hints** qui se trouve dans le répertoire */usr/share/dns/*

Ce fichier contient toutes les adresses @IP des serveurs racines gérant les TLD.

Il est important d'avoir cette liste vide, car à chaque fois il n'a pas la réponse, il va d'abord contacter les serveurs RACINES, qui vont lui répondre en mode itératif, il l'envoie une adresse de serveur DNS susceptible d'y répondre. Si les serveurs racines sont indisponibles, alors il s'arrêtera au niveau du TLD, et répondra au client qu'il n'y a pas de réponse.

## Pour nettoyer le fichier root.hints

D'abord nous faisons un backup du fichier:
```
cp /usr/share/dns/root.hints /usr/share/dns/root.hints.backup
```

Puis nous vidons sont contenus:
```
echo  > /usr/share/dns/root.hints
```

On vérifie après redémarrage:
```
root@S34:~# systemctl status bind9
● named.service - BIND Domain Name Server
     Loaded: loaded (/lib/systemd/system/named.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2022-10-25 14:54:47 CEST; 1min 43s ago
       Docs: man:named(8)
   Main PID: 240887 (named)
      Tasks: 10 (limit: 9411)
     Memory: 31.4M
        CPU: 35ms
     CGroup: /system.slice/named.service
             └─240887 /usr/sbin/named -f -u bind

oct. 25 14:54:47 S34.tp341.iut named[240887]: command channel listening on 127.0.0.1#953
oct. 25 14:54:47 S34.tp341.iut named[240887]: configuring command channel from '/etc/bind/rndc.key'
oct. 25 14:54:47 S34.tp341.iut named[240887]: command channel listening on ::1#953
oct. 25 14:54:47 S34.tp341.iut named[240887]: managed-keys-zone: loaded serial 193
oct. 25 14:54:47 S34.tp341.iut named[240887]: zone 0.in-addr.arpa/IN: loaded serial 1
oct. 25 14:54:47 S34.tp341.iut named[240887]: zone 127.in-addr.arpa/IN: loaded serial 1
oct. 25 14:54:47 S34.tp341.iut named[240887]: zone localhost/IN: loaded serial 2
oct. 25 14:54:47 S34.tp341.iut named[240887]: zone 255.in-addr.arpa/IN: loaded serial 1
oct. 25 14:54:47 S34.tp341.iut named[240887]: all zones loaded
oct. 25 14:54:47 S34.tp341.iut named[240887]: running
root@S34:~#
```

Au niveau des clients dans le fichier **/etc/resolv.conf**, ajoute **nameserver 10.32.121.254**
# Création d'une zone de recherche directe
On crée le répertoire de zones:
```
mkdir /etc/bind/meszones
```

On copie **db.local** en **db.local.poste28.etudiant.iut**
```
cp db.local /etc/bind/db.local.poste28.etudiant.iut
```

On édite le fichier de zones:
```
root@S34:/etc/bind/meszones# cat db.poste34.etudiant.iut
;
; BIND data file for local loopback interface
;
$ORIGIN poste34.etudiant.iut.
$TTL    604800
@       IN      SOA      poste34.etudiant.iut. root.poste34.etudiant.iut. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      poste34.etudiant.iut.
@       IN      A       10.34.121.254
@       IN      A       10.34.221.254
@       IN      A       10.254.34.1
```

On vérifie que le fichier de zones est correct:
```
named-checkzone poste34.etudiant.iut. ./db.poste34.etudiant.iut
```

La commande nous retourne **OK**

Nous devons déclarer la nouvelles zones dans le fichier **named.conf.default-zones** 
```
root@S34:/etc/bind/meszones# cat /etc/bind/named.conf.default-zones
// prime the server with knowledge of the root servers
zone "." {
        type hint;
        file "/usr/share/dns/root.hints";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
        type master;
        file "/etc/bind/db.local";
};
zone "poste34.etudiant.iut"{
    type master;
    file "/etc/bind/meszones/db.poste34.etudiant.iut";
};

zone "127.in-addr.arpa" {
        type master;
        file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
        type master;
        file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
        type master;
        file "/etc/bind/db.255";
};
```

On ajoute ces lignes:
```
zone "poste34.etudiant.iut"{
    type master;
    file "/etc/bind/meszones/db.poste34.etudiant.iut";
};
```

Puis on vérifie que le fichier de configuration est **OK** 
```
named-checkconf /etc/bind/named.conf.default-zones
```

Il nous retourne c'est OK, puis nous redémarrons le service.
```
systemctl restart bind9
```

Nous testons faisons un test **dig poste34.etudiant.iut** :
```
root@D34:~# dig poste34.etudiant.iut

; <<>> DiG 9.16.27-Debian <<>> poste34.etudiant.iut
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40621
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 6172f7f2babc6a96010000006357e33de0542e0f0dd35b5c (good)
;; QUESTION SECTION:
;poste34.etudiant.iut.          IN      A

;; ANSWER SECTION:
poste34.etudiant.iut.   604800  IN      A       10.34.121.254
poste34.etudiant.iut.   604800  IN      A       10.34.221.254
poste34.etudiant.iut.   604800  IN      A       10.254.34.1

;; Query time: 0 msec
;; SERVER: 10.34.221.254#53(10.34.221.254)
;; WHEN: Tue Oct 25 13:08:42 CEST 2022
;; MSG SIZE  rcvd: 125

```

Dans la section **ANSWER** nous avons bien nos trois adresses déclarées

### Wireshark du test

**Le contenu de la requête:**
```
Frame 2: 103 bytes on wire (824 bits), 103 bytes captured (824 bits) on interface eth0, id 0
Ethernet II, Src: HewlettP_10:ea:0b (d4:85:64:10:ea:0b), Dst: D-LinkIn_32:fa:ed (10:62:eb:32:fa:ed)
Internet Protocol Version 4, Src: 10.34.121.1, Dst: 10.34.121.254
User Datagram Protocol, Src Port: 45709, Dst Port: 53
Domain Name System (query)
    Transaction ID: 0xc848
    Flags: 0x0120 Standard query
    Questions: 1
    Answer RRs: 0
    Authority RRs: 0
    Additional RRs: 1
    Queries
        poste34.etudiant.iut: type A, class IN
            Name: poste34.etudiant.iut
            [Name Length: 20]
            [Label Count: 3]
            Type: A (Host Address) (1)
            Class: IN (0x0001)
    Additional records
        <Root>: type OPT
    [Response In: 3]
```

Dans le champ **DNS -> Queries** nous voyons bien la formulation de la question.

**Le contenu de la réponse:**
```
Frame 3: 167 bytes on wire (1336 bits), 167 bytes captured (1336 bits) on interface eth0, id 0
Ethernet II, Src: D-LinkIn_32:fa:ed (10:62:eb:32:fa:ed), Dst: HewlettP_10:ea:0b (d4:85:64:10:ea:0b)
Internet Protocol Version 4, Src: 10.34.121.254, Dst: 10.34.121.1
User Datagram Protocol, Src Port: 53, Dst Port: 45709
Domain Name System (response)
    Transaction ID: 0xc848
    Flags: 0x8580 Standard query response, No error
    Questions: 1
    Answer RRs: 3
    Authority RRs: 0
    Additional RRs: 1
    Queries
        poste34.etudiant.iut: type A, class IN
            Name: poste34.etudiant.iut
            [Name Length: 20]
            [Label Count: 3]
            Type: A (Host Address) (1)
            Class: IN (0x0001)
    Answers
        poste34.etudiant.iut: type A, class IN, addr 10.254.34.1
        poste34.etudiant.iut: type A, class IN, addr 10.34.221.254
        poste34.etudiant.iut: type A, class IN, addr 10.34.121.254
    Additional records
        <Root>: type OPT
    [Request In: 2]
    [Time: 0.000445029 seconds]
```

Dans le champs **DNS -> Answers** Nous voyons bien les trois réponses, de plus la section *Additionnal*, car nous avons pas d'**alias** 

# Création de zones de recherches inverses

Les deux numéros de réseaux que notre serveur DNS doit gérer sont **10.34.121.0/24** et le **10.34.221.0/24** .

Les FQDN des zones:
- **221.34.10.in-addr.arpa.** : Pour la zone 10.34.221.0
- **121.34.10.in-addr.arpa.** : Pour la zone 10.34.121.0

Tout d'abord on déclare les deux zones dans le fichier **named.conf.default-zones**. On ajoute ces lignes:
```
zone "221.34.10.in-addr.arpa"{
    type master;
    file "/etc/bind/meszones/db.10.34.221";
};
zone "121.34.10.in-addr.arpa"{
    type master;
    file "/etc/bind/meszones/db.10.34.121";
};
```

Nous créons les fichiers de zones:
```
touch /etc/bind/meszones/db.10.34.121
touch /etc/bind/meszones/db.10.34.221
```

Le contenu de **db.10.34.121**:
```
root@S34:/etc/bind/meszones# cat db.10.34.121
;
; BIND reverse data file for local loopback interface
;
$ORIGIN 121.34.10.in-addr.arpa.
$TTL    604800
@       IN      SOA     poste34.etudiant.iut. poste34.etudiant.iut. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      poste34.etudiant.iut.
@       IN      A       10.34.121.254
254     IN      PTR     poste34.etudiant.iut.

1       IN      PTR     g34.poste34.etudiant.iut.
```

Puis nous vérifions le fichier de zones:
```
named-checkzone 121.34.10.in-addr.arpa. ./db.10.34.121
```

Le contenu de **db.10.34.121**:
```
root@S34:/etc/bind/meszones# cat db.10.34.221
;
; BIND reverse data file for local loopback interface
;
$ORIGIN 221.34.10.in-addr.arpa.
$TTL    604800
@       IN      SOA     poste34.etudiant.iut. poste34.etudiant.iut. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      poste34.etudiant.iut.
@       IN      A       10.34.221.254
254     IN      PTR     root.poste34.etudiant.iut.

1       IN      PTR     d34.poste34.etudiant.iut.
```

Puis nous vérifions le fichier de zones:
```
named-checkzone 221.34.10.in-addr.arpa. ./db.10.34.221
```

Nous testons avec la commande **dig -x @IP** le paramètre **-x** permet de préciser que nous faisons une résolution inverse.
```
root@D34:~# dig -x 10.34.121.243

; <<>> DiG 9.16.27-Debian <<>> -x 10.34.121.243
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 24271
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: d78aadee7d38a76e010000006357eb5eaf49b7a86611758c (good)
;; QUESTION SECTION:
;243.121.34.10.in-addr.arpa.    IN      PTR

;; AUTHORITY SECTION:
121.34.10.in-addr.arpa. 604800  IN      SOA     poste34.etudiant.iut. root.poste34.etudiant.iut. 1 604800 86400 2419200 604800

;; Query time: 8 msec
;; SERVER: 10.34.221.254#53(10.34.221.254)
;; WHEN: Tue Oct 25 13:43:23 CEST 2022
;; MSG SIZE  rcvd: 144

root@D34:~#
```

### Wireshark
La question:
```
root@G34:~/Documents# cat dns3
Frame 2: 107 bytes on wire (856 bits), 107 bytes captured (856 bits) on interface eth0, id 0
Ethernet II, Src: HewlettP_10:ea:0b (d4:85:64:10:ea:0b), Dst: D-LinkIn_32:fa:ed (10:62:eb:32:fa:ed)
Internet Protocol Version 4, Src: 10.34.121.1, Dst: 10.34.121.254
User Datagram Protocol, Src Port: 50646, Dst Port: 53
Domain Name System (query)
    Transaction ID: 0x2693
    Flags: 0x0120 Standard query
    Questions: 1
    Answer RRs: 0
    Authority RRs: 0
    Additional RRs: 1
    Queries
        1.221.34.10.in-addr.arpa: type PTR, class IN
            Name: 1.221.34.10.in-addr.arpa
            [Name Length: 24]
            [Label Count: 6]
            Type: PTR (domain name PoinTeR) (12)
            Class: IN (0x0001)
    Additional records
        <Root>: type OPT
    [Response In: 3]
```

La réponse:
```
Frame 3: 161 bytes on wire (1288 bits), 161 bytes captured (1288 bits) on interface eth0, id 0
Ethernet II, Src: D-LinkIn_32:fa:ed (10:62:eb:32:fa:ed), Dst: HewlettP_10:ea:0b (d4:85:64:10:ea:0b)
Internet Protocol Version 4, Src: 10.34.121.254, Dst: 10.34.121.1
User Datagram Protocol, Src Port: 53, Dst Port: 50646
Domain Name System (response)
    Transaction ID: 0x2693
    Flags: 0x8580 Standard query response, No error
    Questions: 1
    Answer RRs: 1
    Authority RRs: 0
    Additional RRs: 1
    Queries
        1.221.34.10.in-addr.arpa: type PTR, class IN
            Name: 1.221.34.10.in-addr.arpa
            [Name Length: 24]
            [Label Count: 6]
            Type: PTR (domain name PoinTeR) (12)
            Class: IN (0x0001)
    Answers
        1.221.34.10.in-addr.arpa: type PTR, class IN, d34.poste34.etudiant.iut
    Additional records
        <Root>: type OPT
    [Request In: 2]
    [Time: 0.000238089 seconds]
```

# Fiabilisation du fonctionnement de 3 zones
Visualisation du fichier *usr.sbin.named*
```
root@D34:~# cat /etc/apparmor.d/usr.sbin.named
# vim:syntax=apparmor
# Last Modified: Fri Jun  1 16:43:22 2007
#include <tunables/global>

profile named /usr/sbin/named flags=(attach_disconnected) {
  #include <abstractions/base>
  #include <abstractions/nameservice>

  capability net_bind_service,
  capability setgid,
  capability setuid,
  capability sys_chroot,
  capability sys_resource,

  # /etc/bind should be read-only for bind
  # /var/lib/bind is for dynamically updated zone (and journal) files.
  # /var/cache/bind is for slave/stub data, since we're not the origin of it.
  # See /usr/share/doc/bind9/README.Debian.gz
  /etc/bind/** r,
  /var/lib/bind/** rw,
  /var/lib/bind/ rw,
  /var/cache/bind/** lrw,
  /var/cache/bind/ rw,

  # Database file used by allow-new-zones
  /var/cache/bind/_default.nzd-lock rwk,

  # gssapi
  /etc/krb5.keytab kr,
  /etc/bind/krb5.keytab kr,

  # ssl
  /etc/ssl/openssl.cnf r,

  # root hints from dns-data-root
  /usr/share/dns/root.* r,

  # GeoIP data files for GeoIP ACLs
  /usr/share/GeoIP/** r,

  # dnscvsutil package
  /var/lib/dnscvsutil/compiled/** rw,

  # Allow changing worker thread names
  owner @{PROC}/@{pid}/task/@{tid}/comm rw,

  @{PROC}/net/if_inet6 r,
  @{PROC}/*/net/if_inet6 r,
  @{PROC}/sys/net/ipv4/ip_local_port_range r,
  /usr/sbin/named mr,
  /{,var/}run/named/named.pid w,
  /{,var/}run/named/session.key w,
  # support for resolvconf
  /{,var/}run/named/named.options r,

  # some people like to put logs in /var/log/named/ instead of having
  # syslog do the heavy lifting.
  /var/log/named/** rw,
  /var/log/named/ rw,

  # gssapi
  /var/lib/sss/pubconf/krb5.include.d/** r,
  /var/lib/sss/pubconf/krb5.include.d/ r,
  /var/lib/sss/mc/initgroups r,
  /etc/gss/mech.d/ r,

  # ldap
  /etc/ldap/ldap.conf r,
  /{,var/}run/slapd-*.socket rw,

  # dynamic updates
  /var/tmp/DNS_* rw,

  # dyndb backends
  /usr/lib/bind/*.so rm,

  # Samba DLZ
  /{usr/,}lib/@{multiarch}/samba/bind9/*.so rm,
  /{usr/,}lib/@{multiarch}/samba/gensec/*.so rm,
  /{usr/,}lib/@{multiarch}/samba/ldb/*.so rm,
  /{usr/,}lib/@{multiarch}/ldb/modules/ldb/*.so rm,
  /var/lib/samba/bind-dns/dns.keytab rk,
  /var/lib/samba/bind-dns/named.conf r,
  /var/lib/samba/bind-dns/dns/** rwk,
  /var/lib/samba/private/dns.keytab rk,
  /var/lib/samba/private/named.conf r,
  /var/lib/samba/private/dns/** rwk,
  /etc/samba/smb.conf r,
  /dev/urandom rwmk,
  owner /var/tmp/krb5_* rwk,

  # Site-specific additions and overrides. See local/README for details.
  #include <local/usr.sbin.named>
}
root@D34:~#
```

Il doit stocker dans le répertoire */var/cache/bind/* il possède les autorisations de lectures et d'écritures.

On doit d'abord  ajouter au niveau du serveur DNS maître dans chaque zones l'option **allow-transfer { @ip_esclave; }** .

Fichier du maître **named.conf.default-zones**:
```
root@S34:/etc/bind# cat named.conf.default-zones
// prime the server with knowledge of the root servers
zone "." {
        type hint;
        file "/usr/share/dns/root.hints";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
        type master;
        file "/etc/bind/db.local";
};
zone "poste34.etudiant.iut"{
    type master;
    file "/etc/bind/meszones/db.poste34.etudiant.iut";
    allow-transfer {10.34.121.1;};
};

zone "127.in-addr.arpa" {
        type master;
        file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
        type master;
        file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
        type master;
        file "/etc/bind/db.255";
};
zone "221.34.10.in-addr.arpa"{
    type master;
    file "/etc/bind/meszones/db.10.34.221";
    allow-transfer {10.34.121.1;};
};
zone "121.34.10.in-addr.arpa"{
    type master;
    file "/etc/bind/meszones/db.10.34.121";
    allow-transfer {10.34.121.1;};
};

root@S34:/etc/bind#
```


Fichier du slave **named.conf.default-zones**:
```
// prime the server with knowledge of the root servers
zone "." {
        type hint;
        file "/usr/share/dns/root.hints";
};

// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912

zone "localhost" {
        type master;
        file "/etc/bind/db.local";
};

zone "127.in-addr.arpa" {
        type master;
        file "/etc/bind/db.127";
};

zone "0.in-addr.arpa" {
        type master;
        file "/etc/bind/db.0";
};

zone "255.in-addr.arpa" {
        type master;
        file "/etc/bind/db.255";
};

zone "poste34.etudiant.iut"{
    type slave;
    file "/var/cache/bind/db.poste34.etudiant.iut";
    masters {10.34.121.254;};
};
zone "221.34.10.in-addr.arpa"{
    type slave;
    file "/var/cache/bind/db.10.34.221";
    masters {10.34.121.254;};
};
zone "121.34.10.in-addr.arpa"{
    type slave;
    file "/var/cache/bind/db.10.34.121";
    masters {10.34.121.254;};
};
```

On modifie les options du **named.conf.options** du serveur esclave:
```
root@G34:/var/cache# cat /etc/bind/named.conf.options
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

         forwarders {
                10.254.0.254;
         };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation no;
        forward only;
        listen-on-v6 { any; };
        listen-on { any; };
        allow-query { any; };
        allow-recursion { any;};
};
```

En vérifiant la configuration, on observe que le dossier /var/cache/bind/ n'existe pas.
```
root@G34:/etc/bind# nano named.conf.options
root@G34:/etc/bind# named-checkconf named.conf.options
named.conf.options:2: change directory to '/var/cache/bind' failed: file not found
```

On le crée et on lui donne les droits;
```
mkdir /var/cache/bind/
chown bind:bind /var/cache/bind
```

Puis on redémarre le service. 

On observe bien des échanges entre le *maître* et le *slave* 
![[dns4.jpeg]]
Pour le test. Nous ajoutons dans le fichier *resolv.conf*  la ligne *nameserver 10.34.121.1*. On stop le service bind de S34. et on fait un dig depuis un client.

## Wireshark
Il fait une requête vers S34: *dig -x poste34.etudiant.iut*
```
Frame 15: 96 bytes on wire (768 bits), 96 bytes captured (768 bits) on interface eth0, id 0
Ethernet II, Src: HewlettP_10:ea:0b (d4:85:64:10:ea:0b), Dst: D-LinkIn_32:fa:ed (10:62:eb:32:fa:ed)
Internet Protocol Version 4, Src: 10.34.121.1, Dst: 10.34.121.254
User Datagram Protocol, Src Port: 52720, Dst Port: 53
Domain Name System (query)
    Transaction ID: 0x4c8c
    Flags: 0x0120 Standard query
    Questions: 1
    Answer RRs: 0
    Authority RRs: 0
    Additional RRs: 1
    Queries
        poste34.etudiant.iut: type A, class IN
            Name: poste34.etudiant.iut
            [Name Length: 13]
            [Label Count: 3]
            Type: A (Host Address) (1)
            Class: IN (0x0001)
    Additional records
        <Root>: type OPT
```

Le service bind est éteint il nous retourne un **Destination unreachable**:
```
Frame 16: 124 bytes on wire (992 bits), 124 bytes captured (992 bits) on interface eth0, id 0
Ethernet II, Src: D-LinkIn_32:fa:ed (10:62:eb:32:fa:ed), Dst: HewlettP_10:ea:0b (d4:85:64:10:ea:0b)
Internet Protocol Version 4, Src: 10.34.121.254, Dst: 10.34.121.1
Internet Control Message Protocol
    Type: 3 (Destination unreachable)
    Code: 3 (Port unreachable)
    Checksum: 0x0490 [correct]
    [Checksum Status: Good]
    Unused: 00000000
    Internet Protocol Version 4, Src: 10.34.121.1, Dst: 10.34.121.254
    User Datagram Protocol, Src Port: 52720, Dst Port: 53
Domain Name System (query)
    Transaction ID: 0x4c8c
    Flags: 0x0120 Standard query
    Questions: 1
    Answer RRs: 0
    Authority RRs: 0
    Additional RRs: 1
    Queries
        poste34.etudiant.iut: type A, class IN
            Name: poste34.etudiant.iut
            [Name Length: 13]
            [Label Count: 3]
            Type: A (Host Address) (1)
            Class: IN (0x0001)
    Additional records
        <Root>: type OPT
    [Retransmitted request. Original request in: 15]
```

Le client va solliciter le second serveur DNS **10.34.121.1** 
```
Frame 17: 96 bytes on wire (768 bits), 96 bytes captured (768 bits) on interface eth0, id 0
Ethernet II, Src: HewlettP_10:ea:0b (d4:85:64:10:ea:0b), Dst: D-LinkIn_32:fa:ed (10:62:eb:32:fa:ed)
Internet Protocol Version 4, Src: 10.34.121.1, Dst: 10.254.0.254
User Datagram Protocol, Src Port: 45520, Dst Port: 53
Domain Name System (query)
    Transaction ID: 0x5961
    Flags: 0x0110 Standard query
    Questions: 1
    Answer RRs: 0
    Authority RRs: 0
    Additional RRs: 1
    Queries
        poste34.etudiant.iut: type A, class IN
            Name: poste34.etudiant.iut
            [Name Length: 13]
            [Label Count: 3]
            Type: A (Host Address) (1)
            Class: IN (0x0001)
    Additional records
        <Root>: type OPT
    [Response In: 18]
```

La réponse du serveur:
```
Frame 18: 100 bytes on wire (800 bits), 100 bytes captured (800 bits) on interface eth0, id 0
Ethernet II, Src: D-LinkIn_32:fa:ed (10:62:eb:32:fa:ed), Dst: HewlettP_10:ea:0b (d4:85:64:10:ea:0b)
Internet Protocol Version 4, Src: 10.254.0.254, Dst: 10.34.121.1
User Datagram Protocol, Src Port: 53, Dst Port: 45520
Domain Name System (response)
    Transaction ID: 0x5961
    Flags: 0x8190 Standard query response, No error
    Questions: 1
    Answer RRs: 1
    Authority RRs: 0
    Additional RRs: 1
    Queries
        www.google.fr: type A, class IN
            Name: www.google.fr
            [Name Length: 13]
            [Label Count: 3]
            Type: A (Host Address) (1)
            Class: IN (0x0001)
    Answers
        poste34.etudiant.iut: type A, class IN, addr 10.254.34.1
        poste34.etudiant.iut: type A, class IN, addr 10.34.221.254
        poste34.etudiant.iut: type A, class IN, addr 10.34.121.254
    Additional records
        <Root>: type OPT
    [Request In: 17]
    [Time: 0.004033893 seconds]
```
