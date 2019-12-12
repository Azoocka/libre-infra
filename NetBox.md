# INFORMATIONS SYSTEMES

### Désactivation de SELinux
sudo vi /etc/sysconfig/selinux

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
