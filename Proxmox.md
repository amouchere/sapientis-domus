# Installation de proxmox

* Création de clé USB bootable
* Installation en bootant sur la clé USB
* Configuration par défaut des disques et du réseau

## post install :

* Execution du script **TTECK** de post install

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

    ![debian software installation](assets/Debian%20-%20Software%20installation.PNG)

  * Installation grub boot loader sur la partition unique
* A la fin de l'installation et du reboot, s'authentifier avec l'utilisateur créé pendant l'installation.

# Customisation du template Debian

## Activer les QEMU Guest Agent
Dans les options de la VM, activer QEMU puis reboot la VM. 

![QEMU AGENT](assets/Debian%20-%20QEMU%20Agent.PNG)

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



  # Sources

  https://www.derekseaman.com/2023/06/home-assistant-proxmox-ve-8-0-quick-start-guide.html
