# TP1 - systemd
# Melvin Aubert - M1 Sécurité




## 1. First steps

### Configuration du SSH:

Ajout d'une interface Vbox ( Pont entre Wifi et interface virtuel )
Téléchargement de Putty 
Connexion à l'adaptateur virtuel : ssh azoocka@192.168.56.1

### Connexion à Docker

```
sudo dnf install -y grubby
sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=0"
sudo reboot
sudo dnf config-manager --add-repo=https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install docker-ce
sudo systemctl enable --now docker
```

```
Created symlink /etc/systemd/system/multi-ser.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
```

```
systemctl status docker
sudo groupeadd docker
sudo usermod -aG docker USERNAME
```

### Désactivation de SELINUX

```
sudo setenforce
```

### Vérification de la version de systemd :

```
$systemctl --version : 
systemd 243 (v243.4-1.fc31)
+PAM +AUDIT +SELINUX +IMA -APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD +IDN2 -IDN +PCRE2 default-hierarchy=unified
```
### S'assurer que systemd est PID1:
```
$Ps -ef -> 

      1 root      20   0  108696  15868   9568 S   0,0   0,8   0:02.26 systemd
```

### Brève description des processus systèmes : 
 - PS-ef est un processus qui liste les processus
 - Sshd est un processus qui permet active les fonctionnalitées SSH
 - Chronyd est un daemon de synchronisation de l'horloge du système soit en utilisant un serveur ntp soit en utilisant une  horloge de référence.
- modemManager est un daemon qui va gérer les connexions mobiles communes ( 2G/3G/4G/…)
- Sd-pam est le processus qui gère l'authentification pour les applications ainsi que les différents services de la machine.

## Gestion du temps
```
$timedatectl
```

### Différence entre local time, Universal time, RTC time:

- Local time : Le temps local est l'heure actuelle par rapport au GMT
- Universal time : L'heure qu'il est réellement au niveau du GMT
- RTC : Le RTC est le Real Time Clock de la machine. Celui-ci est maintenu grâce a une pile lorsque la machine est hors tension.

### Expliquer dans quel cas il peut-être pertinent d'utiliser le RTC time :

Le RTC peut-être nécessaire en cas de déconnexion internet ou de coupure entre le service et le serveur NTP choisi / mis en place. Ainsi la machine garde un référentiel d'heure ( nécessaire pour les logs )

### Timezone :
``` 
sudo timedatectl set-timezone Europe/Madrid
timedatectl 
               Local time: ven. 2019-11-29 17:21:25 CET
           Universal time: ven. 2019-11-29 16:21:25 UTC
                 RTC time: ven. 2019-11-29 16:21:23
                Time zone: Europe/Madrid (CET, +0100)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```
### Activation/Désactivation du NTP : 
```
sudo timedatectl st-ntp 0 
timedatectl 
               Local time: ven. 2019-11-29 17:23:40 CET
           Universal time: ven. 2019-11-29 16:23:40 UTC
                 RTC time: ven. 2019-11-29 16:23:37
                Time zone: Europe/Madrid (CET, +0100)
System clock synchronized: yes
              NTP service: inactive
          RTC in local TZ: no
```

## Gestion de noms

### Changement du hostname : 
```
sudo hostnamectl set-hostname fedora-azoo
```
- Static Hotname : C'est le nom traditionnel de la machine
- Pretty Hotsname : Simplement pour la présentation du nom d'hote au User
- Transient Hostname : C'est un nom de machine dynamique utilisé par le kernel. Il n'est pas nécessaire et sera modifié au prochain redémarrage automatiquement

Concernant les machines de production , il serait préférable d'utiliser le hostanme --static afin de pouvoir le réutiliser lors des configurations car celui-ci restera fixe.
```
$systemctl set-deployment <NAME> 
```
Permet d'ajouter un environnement à la machine. Cette information peut nous permettre de nous aider lorsqu'on effectue un inventaire de parc, afin de remonter l'information de l'environnement de la machine ( Production, développement, intégration, …)

## Gestion du réseau - NetworkManager
Deux outils pour gérer le réseau : NetworkManager / sytemd-networkd

## Network Manager :

### Lister les interfaces et les informations liées à une interface : 
```
nmcli con show
nmcli con show <INTERFACE_NAME>
```
## Modifier la configuration d'une interface : 
```
$sudo nano (vi) /etc/sysconfig/network-scripts/ifcfg-<INTERFACE_NAME>
```
## Recharger le Network Manager :
```
$sudo systemctl reload NetworkManager
```
## Redémarrer un interface : 
```
sudo nmcli con up <INTERFACE_NAME>
```
## Modification du fichier de l'interface : 
```
sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
```
```
TYPE=Ethernet
BOOTPROTO=dhcp
NAME=enp0s3
DEVICE=enp0s3
ONBOOT=yes
```
Modificiations apportés au fichier
```
TYPE=Ethernet
BOOTPROTO=no
IPADDR=10.0.2.16
NETMASK=255.255.255.0
DHCPV6C=no
NAME=enp0s3
DEVICE=enp0s3
ONBOOT=yes
```
On désactive le protocole de boot et on indique une adresse IPV4 au poste. N'aimant pas utiliser l'IPV6, j'ai également désactiver l'option dhcpv6 qui évite que la machine demande une adresse DHCPv6 au Master

### Relire les fichiers de configurations : 
```
sudo systemctl reload NetworkManager
```
### On redémarre l'interface:
```
sudo nmcli con down enp0s3
sudo nmcli con up enp0s3
```
### On affiche les nouvelles informations DHCP de l'interface : 
```
sudo nmcli con show enp0s3 | grep DHCP4
sudo nmcli con show enp0s3 | grep IP6
```

```
sudo dnf provides nmtui
sudo dnf install nmtui
```

## Gestion du réseau - systsemd-networkd

### Stopper et déactiver le démarrage de NetworkManager:
```
sudo systemctl stop NetworkManager
sudo systemctl disable NetworkManager
sudo systemctl status NetworkManager
● NetworkManager.service - Network Manager
   Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:NetworkManager(8)
```


### Démarrer et activer le démarrage de systemd-networkd:
```
sudo systemctl enable systemd-networkd
sudo systemctl start systemd-networkd
sudo systemctl status systemd-networkd

● systemd-networkd.service - Network Service
   Loaded: loaded (/usr/lib/systemd/system/systemd-networkd.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2019-12-03 14:45:17 CET; 2min 42s ago
     Docs: man:systemd-networkd.service(8)
 Main PID: 1269 (systemd-network)
   Status: "Processing requests..."
    Tasks: 1 (limit: 2341)
   Memory: 2.2M
   CGroup: /system.slice/systemd-networkd.service
           └─1269 /usr/lib/systemd/systemd-networkd
```

### Edition de la configuration d'un interface de la VM avec un fichier .network
```
cd /etc/systemd/network
sudo vi test.network
```
```
[Match]
Key=enp0s8

[Network]
DHCP=yes
```

Par la suite j'ai redémarré le service. Cette carte me permettant l'utilisation du ssh , je me connectais a ma machine en .101 après les modifications , je suis en.102.

### Résolution de nom : systemd-resolved

## Activer la résolution de noms par systemd-resolved:
```
sudo systemctl enable systemd-resolved
sudo systemctl start systemd-resolved
sudo systemctl status systemd-resolved
● systemd-resolved.service - Network Name Resolution
   Loaded: loaded (/usr/lib/systemd/system/systemd-resolved.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2019-12-03 16:26:19 CET; 56s ago
```

### Contrôler la présence du serveur DNS local:
```
$sudo ss -ltunp:

udp   UNCONN 0      0              127.0.0.53%lo:53          0.0.0.0:*     users:(("systemd-resolve",pid=853,fd=18))

tcp   LISTEN 0      128            127.0.0.53%lo:53          0.0.0.0:*     users:(("systemd-resolve",pid=853,fd=19))
```

On demande au système d'écouter (l) sur les ports de types TCP (t) et UDP (u) en évitant la résolution des hostnames des différents serveurs (n). De plus la commande nous indique le processus utilisé (p)

### Faire une résolution en utilisant explicitement le serveur DNS indiqué par systemd-resolvd:
```
$dig google.com @127.0.0.53

;; ANSWER SECTION:
google.com.             299     IN      A       216.58.215.46
```
### Effectuer une requête DNS avec systemd-resolve.
```
systemd-resolve query google.com

google.com: 216.58.215.46                      -- link: enp0s3
```
### Mise en place du lien symbolique entre resolv.conf et stub-resolv.conf:
```
cd /etc/
sudo mv resolv.conf resolv.conf.old
sudo ln -s /run/systemd//resolve//stub-resolv.conf /etc/resolv.conf
ls -l
lrwxrwxrwx.  1 root root        39  5 déc.  10:48 stub-resolv.conf -> /run/systemd//resolve//stub-resolv.conf
```
### Modifier la configuration de systemd-resolved:
```
cd /etc/systemd
sudo nano resolved.conf
```
Modifications:
```
[Resolve]
DNS=127.0.0.53
FallbackDNS=1.1.1.1 8.8.8.8 1.0.0.1 8.8.4.4

resolvectl
Fallback DNS Servers: 1.1.1.1
                      8.8.8.8
                      1.0.0.1
                      8.8.4.4
```

### Mise en place de DNS over TLS
L'avantage de DNS over TLS c'est l'encryption des requêtes DNS , on remarque également le changement de port 56 -> 853
```
$sudo vi /etc/systemd/resolved.conf
```
```
[Resolve]
DNS=1.1.1.1
#FallbackDNS=8.8.4.4
#Domains=
LLMNR=no
MulticastDNS=no
DNSSEC=no
DNSOverTLS=yes
#Cache=yes
#DNSStubListener=yes
#ReadEtcHosts=yes
```
```
systemd-resolve query dnsprivacy.org

dnsprivacy.org: 62.232.251.199                 -- link: enp0s3

-- Information acquired via protocol DNS in 3.9ms.
-- Data is authenticated: no
```
```
sudo tcpdump -n -nn -v -i enp0s3

12:30:07.165687 IP (tos 0x0, ttl 64, id 3090, offset 0, flags [none], proto TCP (6), length 40)
    1.1.1.1.853 > 10.0.2.15.48884: Flags [.], cksum 0x603f (correct), ack 788, win 65535, length 0
```

### Activation du DNSSEC:
```
$sudo vi /etc/systemd/resolved.conf
DNSSEC=yes
systemd-resolve query dnsprivacy.org

dnsprivacy.org: 62.232.251.199                 -- link: enp0s3

-- Information acquired via protocol DNS in 94.7ms.
-- Data is authenticated: yes
```
On peut voir que les data sont authentifiées , on peut alors dire que le serveur distant a également l'option DNSSEC d'activé.

##Login

Gestion des session ouverte avec les informations de session ( Type, UT, …)
```
1 - azoocka (1000)
           Since: Thu 2019-12-05 14:47:57 CET; 53min ago
          Leader: 891 (sshd)
             TTY: pts/0
          Remote: 192.168.56.1
         Service: sshd; type tty; class user
           State: active
            Unit: session-1.scope
                  ├─ 891 sshd: azoocka [priv]
                  ├─ 904 sshd: azoocka@pts/0
                  ├─ 905 -bash
                  ├─1133 man loginctl
                  └─1143 less
```
### Ejecter un user de sa session : 
```
loginctl kill-session (IDDESESSION
```

## Gestion d'unité basique (service)

### Ajout d'une unité c'est dans /etc/systemd/system.

Afin de trouver l'unité associé au service "chronyd" , on peut utiliser plusieurs moyens:
```
ps -e -o pid,cmd,unit | grep chronyd
```
On affiche les processus en cours ainsi que le PID, le CMD, et l'UNIT Si on connait le PID du processus à l'avance:
```
$systemctl status <PID>
```

## Boot et Logs

générer un graphe de la séquence de boot:

Afin de générer le graphe , on execute cette commande : 
```
Systemd-analyze plot > graphe.svg
```
(On génère le graphe dans un fichier qu'on appelera graphe sous format svg ( format vectorielle interpretable par n'importe quel navigateur web.

Afin d'interpréter ce graphique , on lance un serveur web :

On utilisera le service firewall-cmd déjà disponible sur la machine.
```
Firewal-cmd --list-all
```
On affiche la configuration actuelle du FW
```
Firewall-cmd --add-port=8888/tcp
```
On réalise une ouverture de port 
```
Python -m http.server 88888
```
Via python , on lance un serveur web sur le port 8888 ( port précedemment ouvert)

Ce qui est pratique c'est que le serveur web se lancera depuis le dossier courant. On peut donc directement lire les données:

On détermine donc le temps de lancement du service sshd à 164ms pour CE démarrage de machine. Il fait partie des derniers services à se lancer.

On retrouvera la même donnée depuis la commande systemd-analyze blame.



