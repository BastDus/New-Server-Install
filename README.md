# DEBIAN SERVER
*Fresh install from scratch*

- [🧰Instalation du RAID 1](#instalation-du-raid-1)
    - [⚙️ Admin conf](#admin-conf)
    - [💽 Création du RAID 1](#création-du-raid-1)
    - [🐧Fin de l'installation de l'OS](#fin-de-linstallation-de-los)
- [🖥️Configuration du server](#configuration-du-server)
    - [😎Installer sudo](#installer-sudo)
    - [👤Ajout de l'admin au group sudoers](#ajout-de-ladmin-au-group-sudoers)
    - [❓Récupération des firmewares manquants (facultatif)](#récupération-des-firmewares-manquants-facultatif)
    - [📶Mise en place du Wifi (facultatif)](#mise-en-place-du-wifi-facultatif)
    - [✔️Vérifications des ports ouverts sur la machine](#vérifications-des-ports-ouvertssur-la-machine)
    - [🔗Modification du port SSH](#modification-du-port-ssh)
    - [🚮Suppresion de la connexion pour Root en SSH](#suppresion-de-la-connexion-pour-Root-en-ssh)
    - [🎨Modifier la couleur du prompt](#modifier-la-couleur-du-prompt)
    - [🐋Installer Docker](#installer-docker)
- [🔁Ouverture des port sur la box](#🔁ouverture-des-port-sur-la-box)
- [📦Containers](#containers)
    - [🧭Nginx Proxy Manager](#🧭nginx-proxy-manager)
    - [⚓Portainer](#⚓portainer)

## 🧰Instalation du RAID 1
Installer l'iso Debian sur une clé USB et booter dessus

### ⚙️Admin conf
- nom: ServerAix
- domaine: 
- passwd: xxxxxxxxxxxxx *(root)*
- login: superadmin *(admin x2)*
- login: superadmin
- passwd: xxxxxx

### 💽Création du RAID 1
- partitionement Manuel
- raz des disk et avoir une seule partition "espace libre" sur les deux disk
- Partitions DISK 1 :
    - 1GB -> "début" -> EFI
    - 2GB -> "début" -> SWAP
    - tout le reste -> "début" -> RAID ⚠️ Pas de point de montage (on le ferra plus tard)
- Partitions DISK 2 :
    - 1GB -> "début" -> EFI
    - 2GB -> "début" -> SWAP
    - tout le reste -> "début" -> RAID ⚠️ Pas de point de montage (on le ferra plus tard)
    
**⚠️Monter les 2 disk EXCACTEMENT de la même façon  et dans le même ordre!!**
- séléctionner en haut : "*RAID avec gestion logicielle*"
- création multidisk
- séléctionner les 2 partition en RAID
    - type: ext4
    - mount: racine /
    - laisser le reste par default

### 🐧Fin de l'installation de l'OS
- noter s'il manque un firmeware (ex: la puce wifi) on le récupèrera plus tard *<firmeware-manquant>*
- mandataire:
- pas de GUI/GNOME
- ajouter un server SSH

## 🖥️Configuration du server
### 😎Installer sudo
```
su -
apt-get install sudo
```
*(mot de passe root)*

### 👤Ajout de l'admin au group sudoers
```
su -
adduser superadmin sudo
reboot
```
*(mot de passe root)*

### ❓Récupération des firmewares manquants (facultatif)
mise à jours des sources apt (ajout des sources non open-source):
```
sudo nano /etc/apt/sources-list
```
puis ajouter `non-free` à la fin des lignes après les `main`
```
sudo apt update
sudo apt install <firmeware-manquant>
sudo reboot
```

### 📶Mise en place du Wifi (facultatif)
```
sudo apt install net-tools
sudo reboot
```
récupération de l'interface wifi *(ex: wlp20S)*:
```
ip a
```
installation de l'outils pour la connexion wifi:
```
sudo apt install network-manager
sudo nmtui
```

### ✔️Vérifications des ports ouverts sur la machine
```
netstat -antup | grep LIST
sudo service <service-inutile> stop
sudo uninstall <service-inutile>
netstat -antup | grep LIST
```

### 🔗Modification du port SSH
```
sudo nano /etc/ssh/sshd_config
```
Modifier:
```
Port 2222 (peu importe)
AddressFamily inet
ListenAddress 0.0.0.0 (écoute toutes les ip exterieures)
# ListenAddress :: (évite les écoutes d'ip ipv6)
```
Vérifier le changement de Port
```
netstat -antup | grep LIST
```

### 🚮Suppresion de la connexion pour Root en SSH
Se connecter en SSH, et si la commande `su -` fonctionne, alors on peut éviter que root puisse se connecter depuis l'exterieure
```
sudo nano /etc/ssh/sshd_config
```
Modifier
```
PermitRootLogin no
```

### 🎨Modifier la couleur du prompt
[+ d'infos](https://www.howtogeek.com/307701/how-to-customize-and-colorize-your-bash-prompt/)
```
sudo nano ~/.bashrc
```
modifier la deuxieme ligne PS1 par la même que la première :
```
if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
fi
```


### 🐋Installer Docker
[+ d'infos](https://docs.docker.com/engine/install/debian/)
```
sudo apt update
sudo apt upgrade
sudo apt-get install ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo docker run hello-world
```
Ajouter l'utilisateur au groupe docker pour eviter de taper sudo avant docker...
```
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker 
```

## 🔁Ouverture des port sur la box (facultatif)
Allé sur la Box (ex: Orange = 192.168.1.1)
- Mettre une IP fixe à notre serveur (sur le server directement ou sur la box)
- redémarer le server
- ouvrir uniquement les ports 80, 443, et le port ssh configuré plus tôt ici : 2222
- redirigé ces ports sur l'IP fixe de notre server

    
## 📦Containers
Créer un dossier Docker_Container et créer un dossier par container à l'interrieur
```
mkdir Docker_Container
cd Docker_Container
```

### 🧭Nginx Proxy Manager:
```
mkdir nginx-proxy-manager
cd nginx-proxy-manager
```
y placer le fichier `docker-compose.yml` déjà préconfiguré
```
docker-compose up -d
docker ps
```
Puis créer un sous domaine (ou pas) sur [OVH](https://www.ovh.com/manager/web/index.html#/configuration/domain/bastien-duseaux.com?tab=REDIRECTION) qui redirige vers l'IP public du server
- Aller sur <IP DU SERVER LOCAL>:81 pour accéder au dashboard de NginxPM
- créer un nouveau proxy host qui redirige vers <IP DU SERVER LOCAL>:81



    
### ⚓Portainer:
```
mkdir portainer
cd portainer
```
y placer le fichier `docker-compose.yml` déjà préconfiguré
```
docker-compose up -d
docker ps
```
Puis créer un sous domaine (ou pas) sur [OVH](https://www.ovh.com/manager/web/index.html#/configuration/domain/bastien-duseaux.com?tab=REDIRECTION) qui redirige vers l'IP public du server
