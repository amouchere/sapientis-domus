# Installation de proxmox

* Création de clé USB bootable
* Installation en bootant sur la clé USB
* Configuration par défaut des disques et du réseau

## post install :

* passage du script **TTECK** de post install

``` shell
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/post-pve-install.sh)"
```

Répondre **Y** à toutes les questions.


# Préparation d'un template de VM Debian
* Dans le storage local (PVE), téléverser une image ISO de Debian (AMD64)
  * Les images sont disponibles [ici](https://www.debian.org/releases/)
* Créer une VM basée sur l'image Debian. Laisser les paramètres par défaut sauf si besoin spécifique de RAM/CPU/Storage
* Lancer la VM et procéder à l'installation de l'OS
  * Remplir les options relatives à la localisation
  * Remplir les options relatives aux comptes et à la sécurité
  * Utiliser tout le disque pour l'installation (pas de partitionnement)
  * Pour la partie software :
    * Décocher *desktop environnement* et *gnome*
    * Cocher ssh server
  * Installation grub boot loader sur la partition unique
* A la fin de l'installation et du reboot, s'authentifier avec l'utilisateur créé pendant l'installation.

# Customisation du template Debian

## Activer les QEMU Guest Agent
Dans les options de la VM, activer QEMU puis reboot la VM. 

L'adresse IP de la machine devrait alors apparaître dans le *summary* de la VM.

## Sudo 

Installation de Sudo et ajout du user au groupe **sudo**
``` shell
su -
apt get install sudo
adduser antoine sudo
exit 
```

Permettre l'utilisation de sudo sans systématiquement rentrer le mdp root.
``` shell
sudo visudo
# Chercher la ligne:  
# Defaults        env_reset
# Et modifier cette ligne en:
Defaults          env_reset,timestamp_timeout=-1
```
## Customisation du shell prompt

``` shell
sudo apt install git zsh curl

sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

vi ~/.zshrc
# Ajout de la ligne suivante à la fin du fichier
PROMPT="$fg[cyan]%}$USER@%{$fg[blue]%}%m ${PROMPT}"
```
Se relogguer pour voir le nouveau prompt.

## Construction du template

🎉 La customisation est terminée !  
* Faire un back de la VM.  
* Eteindre la VM.  
* Dans la partie **more**, lancer un **convert to template**


# Création de la VM HAOS

L'installation de Home Assistant est simplifier par l'utilisation d'un script **TTECK**.

* Dans le shell du PVE, lancer la commande: 
```shell
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/vm/haos-vm.sh)"
```

* Choisir la version stable
* Laisser les paramètres par défaut.
* A la fin de la création, lancer le shell de la VM
* Récupérer l'IP qui a été associé à la VM. 
* Se rendre sur ```http://<IP>:8123/``` et HA est prêt à être configuré.



# Création de la VM MQTT & Zigbee2MQTT

* Cliquer droit sur le template et choisir **clone**
* Choisir un nom : *Debian-mqtt*
* Choisir Full clone plutot que Linked clone. Voir la [doc](https://pve.proxmox.com/wiki/VM_Templates_and_Clones)
* Changer le **hostname** de la VM qui a été hérité du template avec la commande : 
```shell 
sudo vi /etc/hosts
# Remplacer debian-modele par debian-mqtt :
127.0.0.1       localhost
127.0.1.1       debian-mqtt.home        debian-mqtt

# Exécuter 
sudo hostnamectl set-hostname debian-mqtt
```

## SSH
* Créer ou récuperer une clé publique afin de faciliter les connexions ssh. (si besoin de la créer, utiliser : **ssh-keygen**)
* Sur la VM, créer un fichier de clés autorisées:
```shell 
mkdir .ssh
vi .ssh/authorized_keys

# Coller ensuite la clé publique dans le fichier créé
```

## Mosquitto 
https://www.tutos.eu/4910

```shell 
# Installation de Mosquitto et d'un client MQTT
sudo apt install mosquitto mosquitto-clients

# Création d'un compte mqtt_u (pwd à définir)
sudo mosquitto_passwd -c /etc/mosquitto/passwd mqtt_u
```

###  Mise à jour de la config

Création d'un fichier de config

```shell
sudo vi /etc/mosquitto/conf.d/default.conf 

# contenu du fichier
listener 1883 0.0.0.0  
allow_anonymous false  
password_file /etc/mosquitto/passwd 
persistence true
```

### Tests 

Logs disponible dans **/var/log/mosquitto**

```shell
tail -f /var/log/mosquitto/mosquitto.log
```

Publication et consommation d'un message mqtt. Ouvrir deux shell distincts.

```shell
# Souscription à un topic
mosquitto_sub -h localhost -t topic/test -u mqtt_u -P mqtt_p

# Envoi d'un message à un topic
mosquitto_pub -h localhost -t topic/test -m "Voici un test" -u mqtt_u -P mqtt_p
```


## Zigbee2Mqtt

https://www.zigbee2mqtt.io/guide/installation/01_linux.html#installing

### Installation de Node et npm
```shell
# Installation de NVM
cd ~
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash

# Installation de la dernière version de node & npm
nvm install v20.9.0 

# Vérification 
npm -version
node --version
```

### Installation de Zigbee2Mqtt

```shell
sudo apt-get install -y make g++ gcc

# Create a directory for zigbee2mqtt and set your user as owner of it
sudo mkdir /opt/zigbee2mqtt
sudo chown -R ${USER}: /opt/zigbee2mqtt

# Clone Zigbee2MQTT repository
git clone --depth 1 https://github.com/Koenkk/zigbee2mqtt.git /opt/zigbee2mqtt

# Install dependencies
cd /opt/zigbee2mqtt
npm ci

# Build the app
npm run build
```

### Configuration Zigbee2Mqtt
```shell
cp /opt/zigbee2mqtt/data/configuration.example.yaml /opt/zigbee2mqtt/data/configuration.yaml
vi /opt/zigbee2mqtt/data/configuration.yaml
```


# Création de la VM InfluxDB & Grafana

* Cliquer droit sur le template et choisir **clone**
* Choisir un nom : *Debian-InfluxDB-Grafana*
* Choisir Full clone plutôt que Linked clone. Voir la [doc](https://pve.proxmox.com/wiki/VM_Templates_and_Clones)
* Changer le **hostname** de la VM qui a été hérité du template avec la commande : 
```shell 
sudo vi /etc/hosts
# Remplacer debian-modele par debian-mqtt :
127.0.0.1       localhost
127.0.1.1       debian-influxdb-grafana.home        debian-influxdb-grafana

# Exécuter 
sudo hostnamectl set-hostname debian-influxdb-grafana
```

## SSH
* Créer ou récuperer une clé publique afin de faciliter les connexions ssh. (si besoin de la créer, utiliser : **ssh-keygen**)
* Sur la VM, créer un fichier de clés autorisées:
```shell 
mkdir .ssh
vi .ssh/authorized_keys

# Coller ensuite la clé publique dans le fichier créé
```

## Influxdb

Documentation : https://docs.influxdata.com/influxdb/v2/install/?t=Linux

```shell
# Ubuntu/Debian AMD64
curl -O https://dl.influxdata.com/influxdb/releases/influxdb2_2.7.3-1_amd64.deb
sudo dpkg -i influxdb2_2.7.3-1_amd64.deb

```

L'installation créé automatiquement un service.
```
sudo service influxdb start
sudo service influxdb status
```

Avec un navigateur, se rendre sur : http://192.168.1.72:8086/

Créer un utilisateur, une organisation et un bucket. 

Copier l'API token.

## Grafana

```shell
# Installation des pré-requiq
sudo apt-get install -y apt-transport-https software-properties-common wget

# Récupération de la clé GPG du repo apt grafana
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null

# Ajout du repository apt 
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# Installation 
sudo apt-get update
sudo apt-get install grafana

# Service
sudo systemctl enable grafana-server.service
sudo systemctl status grafana-server.service
```

* Avec un navigateur, se rendre sur : http://192.168.1.72:3000/
* Se logguer avec admin/admin
* Soumettre un nouveau mot de passe admin


## Config HA:

``` yml
influxdb:
  api_version: 2
  ssl: false
  host: 192.168.1.64
  port: 8086
  token: tiVxYB1f8EoKlDLvcWZuU_5TsDPkCW1EpkWBmuWgmTkkrAKO9ryC2DmMRNnfsabXeaf_DiT3NGObk-hnEiIxwQ==
  organization: moung
  bucket: homeassistant
  tags:
    source: HA
  tags_attributes:
    - friendly_name
  default_measurement: units
  include:
    entities:
      - sensor.garage_temperature
      - sensor.garage_humidite
      - sensor.orange_livebox_vitesse_de_telechargement
```

# Grafana

see capture

exemple request:
from(bucket: "homeassistant")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "°C" or r["_measurement"] == "KiB/s")
  |> filter(fn: (r) => r["_field"] == "value")
  |> filter(fn: (r) => r["domain"] == "sensor")
  |> filter(fn: (r) => r["entity_id"] == "orange_livebox_vitesse_de_telechargement")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")










  # Sources

  https://www.derekseaman.com/2023/06/home-assistant-proxmox-ve-8-0-quick-start-guide.html
