# 🧾 Informations
[![LTSP](https://img.shields.io/badge/LTSP-Thin%20Clients-darkgreen?style=flat-square&logo=server&logoColor=white)](https://ltsp.org/)

- 📦 **LTSP version** : `23.02-1+deb12u1`  
- 🐧 **Distribution** : Debian 12

> [!caution]
> ✅ Ces scripts ont été **testés et validés** dans un environnement conforme aux prérequis.   
> ❌​​​ Si vous rencontrez des **problèmes**, il est probable que cela provienne **de votre configuration**.
---

# ⚙️ Description du script

**📄 Ce script assure la **mise à jour automatisée de l’image LTSP** utilisée par les postes clients dans un environnement en réseau.**

**Avant toute action, il vérifie la présence d’un **fichier flag** afin d’éviter les exécutions multiples ou simultanées, ce qui pourrait provoquer des conflits.**

**Ensuite, il effectue une **synchronisation via `rsync`** avec un serveur distant, tout en **excluant certains dossiers critiques** ou temporaires (comme `/Bureau`, `/Images`, etc.) pour garantir la stabilité de l’image.**

**Une fois la mise à jour terminée, le script **déclenche automatiquement un redémarrage** du système pour que les modifications soient prises en compte dès le prochain démarrage des clients LTSP.**

---

> [!important]
> ✅ **Ce script doit être exécuté automatiquement avant le démarrage de la session utilisateur.**   
> 🧰​ N'oubli**e**z pas de **modifier** `visudo` si c'est un compte utilisateur qui le lance.
```bash
nano /etc/systemd/system/reloadimage.service
# Vous pouvez changer le nom reloadimage par autre chose
```
Copier ceci dedans : 
```ini
[Unit]
Description=Reload Image LTSP
Before=display-manager.service
After=network.target
ConditionKernelCommandLine=|root=/dev/nfs
ConditionKernelCommandLine=|nfsroot

[Service]
Type=oneshot
ExecStart=/etc/script/nodisplay.sh

[Install]
WantedBy=multi-user.target
```
```bash
systemctl daemon-reload
```
```bash
systemctl enable reloadimage
systemctl status reloadimage
```
Ce script se lancera **à** chaque démarrage **et** aura aucun effet sur le serveur **mais** aura un effet sur les clients 

---

**🐧​ - Script Bash :**
```bash
nano /etc/script/nodisplay.sh
```
Copier ceci dedans
```bash
#!/bin/bash

# Chemin du dossier et du fichier flag
tag_dir="/home/internet/tags"
flag_file="$tag_dir/1.flag"

# Vérifier si le dossier tags existe, sinon le créer
if [ ! -d "$tag_dir" ]; then
    sudo mkdir -p "$tag_dir"
fi

# Si le flag existe déjà → on quitte directement
if [ -f "$flag_file" ]; then
    exit 0
fi

sleep 5

sudo rm /home/internet/tags/*

sudo touch "$flag_file"
sync  # Force l'écriture sur le disque

# Attendre 5 seconds
sleep 5

clear

# Synchronisation des fichiers
sudo rsync -av --progress /etc/home/internet/Bureau/ /home/internet/Bureau/

sleep 2

sudo rsync -av --progress --delete-after \
    --exclude='*/tags/' \
    --exclude='*/Bureau/' \
    --exclude='*/Images/' \
    --exclude='*/Documents/' \
    --exclude='*/Téléchargements/' \
    --exclude='*/Vidéos/' \
    --exclude='*/Musique/' \
    --exclude='*/.cache/' \
    --exclude='*/.thunderbird/' \
    /etc/home/internet /home/

sudo rm -f /home/internet/Bureau/ALCASAR*.desktop

sleep 2

clear

for i in {10..1}
do
    echo -ne "\rRedémarrage dans $i secondes..."
    sleep 1
done

# Redémarrage pour permettre la fermeture de session
systemctl reboot
```
