# Procédure - Configuration de KEA DHCP

![Bannière IMDEO](https://imdeo.bts.loutik.fr/assets/banniere_imdeo.png)

---

## Informations

  - **Mainteneur :** MEDO Louis
  - **Date :** 13/03/2026
  - **Documentation :** [IMDEO documentation technique](https://ap-bts-sio-louis.github.io/imdeo/)

---

## 1. Configuration de KEA DHCP

1.  **Identifier l'interface réseau.** Nous allons identifier l'interface réseau utilisée par la machine, qui va ensuite nous servir dans la configuration de KEA DHCP.

    ```bash
    ip a
    ```

      * **`ip`** : Outil principal d'administration réseau sous Linux (remplace l'obsolète `ifconfig`).
      * **`a`** (raccourci pour `address`) : Affiche la liste des interfaces réseau et leurs adresses IP associées.

    **Exemple d'interface :**

    ```bash
    ens3
    ```

2.  **Sauvegarde de la configuration.** Avant toute modification, nous sauvegardons le fichier de configuration afin de pouvoir revenir en arrière en cas de problème.

    *Copie de la configuration actuelle :*

    ```bash
    sudo cp /etc/kea/kea-dhcp4.conf /etc/kea/kea-dhcp4.conf.bak
    ```

      * **`sudo`** : Exécute la commande avec les droits d'administrateur.
      * **`cp`** : Commande de copie de fichiers.
      * **`/etc/.../kea-dhcp4.conf`** : Le fichier source à copier.
      * **`/etc/.../kea-dhcp4.conf.bak`** : Le fichier de destination. L'extension `.bak` est une convention standard en administration système pour désigner une sauvegarde (backup).

3.  **Ouvrir le fichier de configuration de KEA DHCP.**

    *Édition de la configuration actuelle :*

    ```bash
    sudoedit /etc/kea/kea-dhcp4.conf
    ```

      * **`sudoedit`** : Méthode sécurisée pour éditer un fichier système. Contrairement à `sudo nano` ou `sudo vim`, elle copie le fichier dans un espace temporaire, l'ouvre avec les droits de l'utilisateur standard, puis écrase le fichier original avec les droits root lors de la sauvegarde.

    **💡 Astuce VIM :** Si vous utilisez VIM, vous pouvez taper `gg` (pour aller au début du fichier) puis `dG` en mode normal (mode commande) pour effacer tout le contenu du fichier d'un coup.

4.  **Configuration de KEA DHCP.**

    👉 [Configuration kea DHCP01](./kea-dhcp4.conf)

5.  **Vérification de la syntaxe.**

    ```bash
    sudo -u _kea kea-dhcp4 -t /etc/kea/kea-dhcp4.conf
    ```

      * **`-u _kea`** : Indique à `sudo` d'exécuter la commande en tant que l'utilisateur système spécifique `_kea` (l'utilisateur sous lequel le service tourne), et non en tant que root.
      * **`kea-dhcp4`** : Appel direct au binaire du service KEA IPv4.
      * **`-t`** (test) : Demande à KEA d'analyser le fichier de configuration pour détecter d'éventuelles erreurs de syntaxe, sans démarrer le service.
      * **`/etc/...`** : Le chemin absolu vers le fichier à tester.

6.  **Redémarrer kea DHCP.**

    ```bash
    sudo systemctl restart kea-dhcp4-server.service
    ```

      * **`systemctl`** : Outil de gestion des services systemd.
      * **`restart`** : Arrête puis redémarre immédiatement le service. Indispensable pour que le démon charge le nouveau fichier de configuration.

7.  **Vérification du bon fonctionnement de KEA DHCP.**

    ```bash
    sudo systemctl status kea-dhcp4-server.service
    ```

      * **`status`** : Affiche l'état actuel du processus, son temps de disponibilité, et les dernières lignes de logs pour confirmer qu'il n'y a pas d'erreur au lancement.

    **Résultat attendu :**

    ```bash
    ● kea-dhcp4-server.service - Kea IPv4 DHCP daemon
      Loaded: loaded (/lib/systemd/system/kea-dhcp4-server.service; enabled; vendor preset: enabled)
      Active: active (running)
      Main PID: 3427 (kea-dhcp4)
    ```

---

## Annexe

  - [Debian : Installer et configurer le serveur DHCP avec kea](https://www.linuxtricks.fr/wiki/debian-installer-et-configurer-le-serveur-dhcp-avec-kea)
  - [Installer et configurer un serveur DHCP avec KEA sur Debian](https://qdemouliere.github.io/sisr1/documentation/Services/02--dhcp/)