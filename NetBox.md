# INFORMATIONS SYSTEMES
azoocka@Latitude
---------------- 
* OS: Fedora 29 (Workstation Edition) x86_64 
* Host: Latitude E7270 
* Kernel: 5.3.11-100.fc29.x86_64 
* Uptime: 1 hour, 43 mins 
* Packages: 2011 (rpm), 6 (snap)
* Shell: bash 4.4.23 
* Resolution: 1920x1080
* DE: GNOME 3.30.2 
* Theme: Adwaita [GTK2/3]
* Icons: Adwaita [GTK2/3] 
* Terminal: gnome-terminal 
* CPU: Intel i5-6300U (4) @ 3.000GHz
* GPU: Intel Skylake GT2 [HD Graphics 520] 
* Memory: 2695MiB / 7636MiB 
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
