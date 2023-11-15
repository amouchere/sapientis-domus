
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
