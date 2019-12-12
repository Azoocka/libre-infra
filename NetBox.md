# INFORMATIONS SYSTEMES

          /:-------------:\          azoocka@Latitude 
       :-------------------::        ---------------- 
     :-----------/shhOHbmp---:\      OS: Fedora 29 (Workstation Edition) x86_64 
   /-----------omMMMNNNMMD  ---:     Host: Latitude E7270 
  :-----------sMMMMNMNMP.    ---:    Kernel: 5.3.11-100.fc29.x86_64 
 :-----------:MMMdP-------    ---\   Uptime: 1 hour, 43 mins 
,------------:MMMd--------    ---:   Packages: 2011 (rpm), 6 (snap) 
:------------:MMMd-------    .---:   Shell: bash 4.4.23 
:----    oNMMMMMMMMMNho     .----:   Resolution: 1920x1080 
:--     .+shhhMMMmhhy++   .------/   DE: GNOME 3.30.2 
:-    -------:MMMd--------------:    Theme: Adwaita [GTK2/3] 
:-   --------/MMMd-------------;     Icons: Adwaita [GTK2/3] 
:-    ------/hMMMy------------:      Terminal: gnome-terminal 
:-- :dMNdhhdNMMNo------------;       CPU: Intel i5-6300U (4) @ 3.000GHz 
:---:sdNMMMMNds:------------:        GPU: Intel Skylake GT2 [HD Graphics 520] 
:------:://:-------------::          Memory: 2695MiB / 7636MiB 
:---------------------:/

### Désactivation de SELinux
```
sudo vi /etc/sysconfig/selinux
```
### Installation de docker
```
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager     --add-repo     https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf config-manager --set-disabled docker-ce-nightly
sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose
sudo systemctl enable --now docker
sudo docker run hello-world
sudo usermod a-G docker <username>
```

### Installation d'un container Netbox
```
docker search netbox
docker pull netboxcommunity/netbox
git clone -b master https://github.com/netbox-community/netbox-docker.git
cd netbox-docker
docker-compose pull
docker-compose up -d
open "http://$(docker-compose port nginx 8080)/"
