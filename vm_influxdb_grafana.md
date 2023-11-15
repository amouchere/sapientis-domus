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

```shell
sudo service influxdb start
sudo service influxdb status
```

Avec un navigateur, se rendre sur : http://192.168.1.72:8086/

Créer un utilisateur, une organisation et un bucket. 

![Influxdb installation](assets/influxdb%20-%20user%20creation.png) 

Copier l'API token.

## Grafana

```shell
# Installation des pré-requis
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
* Installer une datasource influxdb

![Grafana datasource](assets/Grafana%20-%20datasource%20influxdb.png) 


# exemple de requête

exemple request:
from(bucket: "homeassistant")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "°C" or r["_measurement"] == "KiB/s")
  |> filter(fn: (r) => r["_field"] == "value")
  |> filter(fn: (r) => r["domain"] == "sensor")
  |> filter(fn: (r) => r["entity_id"] == "orange_livebox_vitesse_de_telechargement")
  |> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)
  |> yield(name: "mean")


