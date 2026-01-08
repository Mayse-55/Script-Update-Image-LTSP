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
> ✅ **Ce script doit être exécuté automatiquement au démarrage de la session utilisateur.**   
> 🧰​ Il est écrit en **Bash** et doit être lancé avec les droits nécessaires.
```bash
chmod +x /etc/script/reloadimage
# Remplacez /etc/script/autoscan par le chemin réel de votre script
```

---

**🐧​ - Script Bash :**
```bash
#!/bin/bash

# Chemin du dossier et du fichier flag
tag_dir="/home/internet/tags"
flag_file="$tag_dir/test.flag"

# Vérifier si le dossier tags existe, sinon le créer
if [ ! -d "$tag_dir" ]; then
    sudo mkdir -p "$tag_dir"
fi

# Si le flag existe déjà → on quitte directement
if [ -f "$flag_file" ]; then
    exit 0
fi

sleep 5

# Si on est pas encore dans un vrai terminal, alors ouvrir xfce4-terminal
if [ "$TERM" = "dumb" ] || [ -z "$TERM" ]; then
    xfce4-terminal --hold --command="$0"
    exit 0
fi

# === Ici, on est dans une vraie fenêtre de terminal ===


echo "#######################################################"
echo "######                                           ######"
echo "####            Mise à jour en cours ...           ####"
echo "######                                           ######"
echo "#######################################################"

touch "$flag_file"
sync  # Force l'écriture sur le disque

# Attendre 5 seconds
sleep 5

clear

echo "#######################################################"
echo "######                                           ######"
echo "####    Synchronisation au serveur en cours ...    ####"
echo "######                                           ######"
echo "#######################################################"
echo ""

# Synchronisation des fichiers
rsync -av --progress --delete-after \
    --exclude='*/tags/' \
    --exclude='*/Bureau/' \
    --exclude='*/Images/' \
    --exclude='*/Documents/' \
    --exclude='*/Téléchargements/' \
    --exclude='*/Vidéos/' \
    --exclude='*/Musique/' \
    --exclude='*/.cache/' \
    --exclude='*/.thunderbird/' \
    --exclude='*/.mozilla/' \
    /etc/bpx /home/

# Attendre 5 seconds
sleep 2

clear

echo "######################################################"
echo "######                                          ######"
echo "####            Mise à jour terminé !             ####"
echo "######                                          ######"
echo "######################################################"
echo ""

for i in {10..1}
do
    echo -ne "\rRedémarrage dans $i secondes..."
    sleep 1
done

# Redémarrage pour permettre la fermeture de session
systemctl reboot
```
