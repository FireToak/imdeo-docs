# Procédure - Gestion des snapshots avec LVM

![Logo IMDEO](https://github.com/AP-BTS-SIO-Louis/imdeo/raw/main/docs_imdeo/docs/assets/logo_imdeo.jpg)

---
## Informations

- **Mainteneur :** MEDO Louis
- **Date :** 13/03/2026
- **Documentation :** [IMDEO documentation technique](https://ap-bts-sio-louis.github.io/imdeo/)

---
## Objectif

L'objectif de la procédure est de montrer comment créer, restaurer et supprimer un snapshot LVM en toute sécurité.

---
## Sommaire

1. Configuration de LVM pour les snapshots
2. Bonnes pratiques d'administration

---
## 1. Configuration de LVM pour les snapshots

### Étape 1 : Vérification

Vérifier qu'il reste de la place (`VFree`) dans le groupe de volumes afin d'allouer de l'espace au snapshot.

```Bash
sudo vgs
```

**`vgs`** : Affiche un résumé de l'état des groupes de volumes (Volume Groups), notamment l'espace total et libre.

**Résultat :**

```Bash
  VG     #PV #LV #SN Attr   VSize   VFree   
  vgsio1   2   5   0 wz--n- <39,06g 1020,00m
```

### Étape 2 : Identification

Identifier le volume logique cible.

```Bash
sudo lvs
```

**`lvs`** : Liste les volumes logiques (Logical Volumes) et affiche leurs attributs et tailles.

**Résultat :**

```Bash
  LV     VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Sync Convert
  lvsrv  vgsio1 -wi-ao----  9,00g # Volume logique cible  
```

### Étape 3 : Création du snapshot

Création du snapshot nommé `snapsrv` basé sur `lvsrv`.

```Bash
sudo lvcreate -L 1000M -s -n snapsrv /dev/vgsio1/lvsrv
```

**`lvcreate`** : Crée un volume logique.
**`-L 1000M`** : Définit une taille fixe de 1000 Mo pour stocker les modifications.
**`-s`** : Indique qu'il s'agit d'un snapshot.
**`-n snapsrv`** : Nomme le nouveau snapshot.

---
### Étape 4 : Restauration (Rollback)

**Cas A : Restaurer un snapshot démontable (à chaud)**

Si le volume n'est pas critique pour le système (ex: `/srv`), on peut le démonter pour appliquer la restauration immédiatement.

```Bash
sudo umount /srv 
sudo lvconvert --merge /dev/vgsio1/snapsrv
sudo mount -a
```

**`umount`** : Déconnecte le système de fichiers de l'arborescence.
**`lvconvert --merge`** : Fusionne le snapshot avec l'origine. Cela écrase le volume d'origine pour le ramener à l'état du snapshot, puis **supprime** automatiquement le snapshot.
**`mount -a`** : Remonte tous les systèmes de fichiers définis dans le `/etc/fstab`.

**Cas B : Restaurer un snapshot non démontable (système)**

Si le volume est en cours d'utilisation par le système (ex: `/`), la fusion est programmée au prochain redémarrage.

```Bash
sudo lvconvert --merge /dev/vgsio1/snapsrv
sudo shutdown -r now
```

**`shutdown -r now`** : Force un redémarrage immédiat (`-r` pour reboot) du système pour appliquer la fusion au boot.

---
### Étape 5 : Nettoyage (Validation des changements)

Si l'intervention s'est bien passée et que vous souhaitez **conserver** les modifications, il faut supprimer le snapshot. _Attention : Cela ne fusionne rien, cela détruit simplement l'historique de retour en arrière._

```Bash
sudo lvremove /dev/vgsio1/snapsrv
```

**`lvremove`** : Supprime définitivement un volume logique (ici, le snapshot) et libère l'espace dans le groupe de volumes.

---
## 2. Bonnes pratiques du SysAdmin

**Ne jamais conserver un snapshot indéfiniment :** Un snapshot dégrade les performances d'écriture (I/O) du volume d'origine. Supprime-le dès que ton intervention est validée.

**Surveiller le remplissage :** Si un snapshot atteint 100 % de sa capacité (les modifications dépassent la taille allouée), il devient **invalide et inutilisable**. Vérifie régulièrement avec `lvs` (colonne `Data%`).

**Un snapshot n'est pas une sauvegarde :** Si le disque physique lâche, le volume d'origine et le snapshot sont perdus. Utilise toujours un vrai système de backup externe en parallèle.

---