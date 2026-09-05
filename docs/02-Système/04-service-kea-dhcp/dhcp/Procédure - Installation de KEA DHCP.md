# Procédure - Installation de KEA DHCP

![Logo IMDEO](https://github.com/AP-BTS-SIO-Louis/imdeo/raw/main/docs_imdeo/docs/assets/logo_imdeo.jpg)

---

## Informations

  - **Mainteneur :** MEDO Louis
  - **Date :** 13/03/2026
  - **Documentation :** [IMDEO documentation technique](https://ap-bts-sio-louis.github.io/imdeo/)

---

## 1. Installation de KEA DHCP

1.  **Vérification et mise à jour Debian.** Avant de procéder à l'installation de KEA DHCP, nous allons vérifier que le serveur Debian possède bien les dernières mises à jour.

    ```bash
    sudo apt update && sudo apt upgrade -y
    ```

    **`sudo`** : Exécute la commande avec les privilèges d'administrateur (root).

    **`apt`** : Outil de gestion des paquets sous Debian/Ubuntu.

    **`update`** : Met à jour l'index local des paquets disponibles depuis les dépôts.

    **`&&`** : Opérateur logique de chaînage. Exécute la seconde commande uniquement si la première réussit (code retour 0).

    **`upgrade`** : Télécharge et installe les nouvelles versions des paquets déjà installés.

    **`-y`** : Argument optionnel (mais recommandé en procédure) qui valide automatiquement les demandes de confirmation ("yes").

2.  **Installer le paquet KEA DHCP.**

    ```bash
    sudo apt install kea-dhcp4-server
    ```

    **`install`** : Indique au gestionnaire de paquets de télécharger et d'installer un nouveau logiciel.

    **`kea-dhcp4-server`** : Nom exact du paquet ciblé, correspondant spécifiquement au service DHCP pour le protocole IPv4.

3.  **Vérification de l'installation.**

    ```bash
    sudo systemctl status kea-dhcp4-server
    ```

    **`systemctl`** : Commande principale pour interagir avec *systemd*, le système d'initialisation et de gestion des services sous Linux.

    **`status`** : Demande l'affichage de l'état actuel d'un service (démarré, arrêté, en erreur, logs récents).

    **Résultat attendu :**

    ```bash
    ● kea-dhcp4-server.service - Kea IPv4 DHCP daemon
      Loaded: loaded (/lib/systemd/system/kea-dhcp4-server.service; enabled; vendor preset: enabled)
      Active: active (running)
      Main PID: 3427 (kea-dhcp4)
    ```

---

## Annexe

  - [Installer et configurer un serveur DHCP avec KEA sur Debian](https://qdemouliere.github.io/sisr1/documentation/Services/02--dhcp/)