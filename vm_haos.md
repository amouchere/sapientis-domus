
# Création de la VM HAOS

L'installation de Home Assistant est simplifier par l'utilisation d'un script **TTECK**.

* Dans le shell du PVE, lancer la commande: 
```shell
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/vm/haos-vm.sh)"
```

* Choisir la version stable
* Laisser les paramètres par défaut.


![HAOS fin d'install](assets/HAOS%20-%20fin%20d'installation.PNG)

* A la fin de la création, lancer le shell de la VM
* Récupérer l'IP qui a été associé à la VM. 
* Se rendre sur ```http://<IP>:8123/``` et HA est prêt à être configuré.


![HAOS fin d'install](assets/HAOS%20-%20IP.PNG)

## Config HA:

Une fois la VM influxdb instanciée, il faut configurer HA pour l'utiliser. 

``` yml
influxdb:
  api_version: 2
  ssl: false
  host: 192.168.1.64
  port: 8086
  token: <TOKEN INFLUXDB>
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