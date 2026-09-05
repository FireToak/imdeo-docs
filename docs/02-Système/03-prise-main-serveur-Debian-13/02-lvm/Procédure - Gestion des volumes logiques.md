# Procédure - Gestion des volumes logiques

![Bannière IMDEO](https://imdeo.bts.loutik.fr/assets/banniere_imdeo.png)

---
## Informations

- **Mainteneur :** MEDO Louis
- **Date :** 13/03/2026
- **Documentation :** [IMDEO documentation technique](https://ap-bts-sio-louis.github.io/imdeo/)

---
## Objectif

Cette procédure explique comment étendre la capacité d'un volume logique (LVM) existant en intégrant un nouveau disque physique au système, ainsi que les étapes complètes pour créer, formater et rendre accessible un nouveau volume logique.

---
## Sommaire

A. Ajouter de l'espace supplémentaire sur un volume LVM
B. Créer un nouveau volume logique

---
## A. Ajouter de l'espace supplémentaire sur un volume LVM

1. **Préparer le disque et étendre le groupe.** Permet au système LVM d'utiliser le nouveau disque physique (ex: `/dev/sdb`).

```Bash
# Pour voir le point de montage de votre nouveau disque
sudo lsblk

# Initialiser le disque sur LVM
sudo pvcreate /dev/sdb
sudo vgextend vgsio1 /dev/sdb
```

`pvcreate` : Initialise le nouveau disque en tant que Volume Physique (Physical Volume).
`vgextend` : Ajoute ce volume physique au Groupe de Volumes (Volume Group) existant, ici nommé `vgsio1`.

2. **Voir les volumes LVM.** Cette commande nous permet de voir le nom du volume que vous souhaitez augmenter ainsi que le chemin exact du volume.

```Bash
sudo lvdisplay
```
    
`lvdisplay` : Affiche les propriétés détaillées des Volumes Logiques (nom, chemin, taille).

**Exemple de résultat :**

```Bash
  --- Logical volume ---
  LV Path                /dev/vgsio1/lvhome # Vous pouvez copier le chemin ici
  LV Name                lvhome
  VG Name                vgsio1
  LV UUID                oGwe50-UcEs-8Lim-UC2x-q2UB-rXj0-SHw0Tv
  LV Write Access        read/write
  LV Creation host, time srv0, 2026-03-06 15:02:32 +0100
  LV Status              available
  # open                 1
  LV Size                <4,66 GiB
  Current LE             1192
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     4096
  Block device           254:1
```

3. **Augmenter le volume LVM.**

```Bash
sudo lvextend -L +20G /dev/vgsio1/lvhome
```
    
`lvextend` : Modifie la taille du Volume Logique pour l'agrandir.
`-L +20G` : Option pour ajouter exactement 20 Go d'espace supplémentaire.

4. **Redimensionner le système de fichiers.** L'espace LVM est augmenté, mais le système de fichiers doit maintenant l'occuper.

**Pour ext4 :**

```Bash
    sudo resize2fs /dev/vgsio1/lvhome
```

`resize2fs` : Étend le système de fichiers de type _ext4_ pour qu'il corresponde à la nouvelle taille du volume (pour un système _XFS_, utilisez `xfs_growfs`).

**Pour BTRFS :**

```bash
sudo btrfs filesystem resize max /home
```

`btrfs filesystem resize` : Utilitaire spécifique à BTRFS permettant de modifier la taille du système de fichiers en direct (à chaud).
`max` : Paramètre indiquant au système de s'étendre pour occuper la totalité de l'espace libre nouvellement alloué au volume logique LVM.
`/home` : Le **point de montage** actif. Contrairement à `resize2fs` qui cible le chemin du périphérique matériel (`/dev/gvsio1/lvhome`), BTRFS applique son redimensionnement directement sur le dossier où le volume est monté.

---
## B. Créer un nouveau volume logique

1. **Créer le volume logique.** Alloue un nouvel espace dédié depuis le groupe de volumes existant.

```Bash
sudo lvcreate -L 10G -n lvdata vgsio1
```
    
`lvcreate` : Commande de création d'un nouveau Volume Logique.
`-L 10G` : Définit la taille du nouveau volume à 10 Go.
`-n lvdata` : Attribue le nom `lvdata` au volume.
`vgsio1` : Cible le Groupe de Volumes parent où prendre l'espace.

2. **Formater le volume.** Préparer la structure du stockage pour accueillir des données.

```Bash
sudo mkfs.ext4 /dev/vgsio1/lvdata
```

`mkfs.ext4` : Formate (Make FileSystem) le volume brut avec le système de fichiers standard Linux _ext4_.

3. **Créer le point de montage et monter le volume.** Rend l'espace de stockage utilisable par l'OS.

```Bash
sudo mkdir -p /srv
sudo mount /dev/vgsio1/lvsrv /srv
```

`mkdir -p` : Crée le répertoire cible `/srv` (l'option `-p` évite les erreurs si le dossier existe déjà).
`mount` : Attache temporairement le volume au répertoire, permettant d'y lire et écrire des fichiers.

4. **Ouvrir le fichier fstab.** Ce fichier contient les volumes monter par défaut lors du démarrage du système.

```bash
sudoedit /etc/fstab
```

`fstab` : permet de monter des partitions, des logical volumes, ou des partages réseaux automatiquement au démarrage. 

5. **Ajouter le volume au fichier.** Ajouter les lignes suivantes pour que le volume soit automatiquement monter au démarrage de la machine.

```bash
/dev/vgsio1/lvsrv /srv ext4 defaults 0 2
```

`/dev/vgsio1/lvsrv` : Le chemin de ton volume logique (tu peux aussi utiliser le `UUID` obtenu lors de ton formatage).
`/srv` : Le dossier cible (point de montage).
`ext4` : Le type de système de fichiers que tu as appliqué avec `mkfs.ext4`.
`defaults` : Utilise les options standards (lecture/écriture, montage automatique au boot, etc.).
`0` : Option pour l'utilitaire de sauvegarde `dump` (0 = désactivé).
`2` : Ordre de vérification du disque par `fsck` au démarrage (2 = après la partition racine).

6. **Vérification.** Tester que la configuration est bien appliqué sur le système.

```bash
sudo mount -a
```

`mount -a` (all) : Lit le fichier `/etc/fstab` et monte tout ce qui ne l'est pas encore. **C'est une étape de sécurité indispensable** : si la commande passe sans erreur, cela confirme que ta ligne dans `fstab` est correcte et que ton serveur redémarrera sans bloquer.

---
