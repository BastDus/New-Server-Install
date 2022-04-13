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
    - [🚮Suppresion de la connexion pour Root en SSH](#suppresion-de-la-connexion-pour-root-en-ssh)
    - [🎨Modifier la couleur du prompt](#modifier-la-couleur-du-prompt)
    - [🐋Installer Docker](#installer-docker)
- [🔁Ouverture des port sur la box](#ouverture-des-port-sur-la-box)
- [📦Containers](#containers)
    - [🧭Nginx Proxy Manager](#nginx-proxy-manager)
    - [⚓Portainer](#portainer)

## 🧰Instalation du RAID 1
Installer l'iso Debian sur une clé USB et booter dessus

### ⚙️Admin conf
- **nom:** ServerAix
- **domaine:** 
- **passwd:** xxxxxxxxxxxxx *(root)*
- **login:** superadmin *(admin x2)*
- **login:** superadmin
- **passwd:** xxxxxx

### 💽Création du RAID 1
- Choisir **Partitionement Manuel**
- RAZ des disks et avoir une seule partition "espace libre" sur les deux disks
- Partitions DISK 1 :
    - 1GB -> "début" -> **EFI**
    - 2GB -> "début" -> **SWAP**
    - tout le reste -> "début" -> **RAID ⚠️ Pas de point de montage (on le ferra plus tard)**
- Partitions DISK 2 :
    - 1GB -> "début" -> **EFI**
    - 2GB -> "début" -> **SWAP**
    - tout le reste -> "début" -> **RAID ⚠️ Pas de point de montage (on le ferra plus tard)**
    
**⚠️Monter les 2 disk EXCACTEMENT de la même façon  et dans le même ordre.**
- séléctionner en haut : **RAID avec gestion logicielle**
- Création Multidisk
- Séléctionner les 2 partitions en RAID
    - **type**: ext4
    - **mount**: racine /
    - laisser le reste par default

### 🐧Fin de l'installation de l'OS
- Noter s'il manque un firmeware *(ex: la puce wifi)* on le récupèrera plus tard **<firmeware-manquant>**
- **mandataire:** __
- **pas de GUI/GNOME**
- **ajouter un server SSH**

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
Mise à jours des sources apt *(ajout des sources non open-source)*:
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
Récupération de l'interface wifi *(ex: wlp20S)*:
```
ip a
```
Installation de l'outils pour la connexion wifi:
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
Vérifier le changement de Port:
```
netstat -antup | grep LIST
```

### 🚮Suppresion de la connexion pour Root en SSH
Se connecter en SSH, et si la commande `su -` fonctionne, alors on peut éviter que Root puisse se connecter depuis l'exterieure:
```
sudo nano /etc/ssh/sshd_config
```
Modifier:
```
PermitRootLogin no
```

### 🎨Modifier la couleur du prompt
[+ d'infos](https://www.howtogeek.com/307701/how-to-customize-and-colorize-your-bash-prompt/)
```
sudo nano ~/.bashrc
```
Modifier la deuxieme ligne PS1 par la même que la première :
```
if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
fi
```


### 🐋Installer Docker
(TODO => et docker-compose)
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
(TODO => à mettre au niveau ## Avant la configuration du server)
- Allé sur ma Box internet *(ex: Orange = 192.168.1.1; Free = mafreebox.freebox.fr...etc...)*
- Mettre une **IP fixe** à notre serveur (sur le server directement ou sur la box)
- Redémarer le server
- Ouvrir uniquement les ports 80, 443, 81 *(qu'on supprimera plus tard)* et le port SSH configuré plus tôt ici : 2222
- Rediriger tous ces ports sur l'**IP fixe** de notre server

    
## 📦Containers
Créer un dossier `Docker_Container` à la racine de l'user et créer un dossier par container à l'interrieur
```
cd
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
- Créer un sous domaine (ou pas) sur [OVH](https://www.ovh.com/manager/web/index.html#/configuration/domain/bastien-duseaux.com?tab=REDIRECTION) qui redirige vers l'IP public du server
- Aller sur **ip fixe:81** pour accéder au dashboard de Nginx Proxy Manager
- Créer un nouveau *proxy host* avec le nom de domaine créér sur OVH qui redirige vers **ip fixe:81**
- Tester le nom de domaine et si ça marche => retourner sur notre box et retirer le port 81
- Re-tester le nom de domaine.

**💡 Si le container a le même network que Nginx Proxy Manager, alors on peut set le "Forward Hostname / IP" avec le nom du container lors de la création d'un proxy host à la place de l'adresse IP fixe du server**

    
### ⚓Portainer:
```
mkdir portainer
cd portainer
```
Y placer le fichier `docker-compose.yml` déjà préconfiguré
```
docker-compose up -d
docker ps
```
-  Créer un sous domaine (ou pas) sur [OVH](https://www.ovh.com/manager/web/index.html#/configuration/domain/bastien-duseaux.com?tab=REDIRECTION) qui redirige vers l'IP public du server
-  Allé sur Nginx Proxy Manager afin de créer un *proxy host* avec le nom de domaine créér sur OVH qui redirige vers **ip fixe:9000** *(voir le port dans le docker-compose.yml)*

    
(TODO ==> fail2ban)









