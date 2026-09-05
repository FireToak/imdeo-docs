# Procédure - Configuration et Sécurisation d'un accès SSH

![Bannière IMDEO](https://imdeo.bts.loutik.fr/assets/banniere_imdeo.png)

---
## Informations

- **Mainteneur :** MEDO Louis
- **Date :** 13/03/2026
- **Documentation :** [IMDEO documentation technique](https://ap-bts-sio-louis.github.io/imdeo/)

---
## 1. Installation du service SSH

1. **Installation du paquet.**

    ```Bash
    sudo apt update && sudo apt install openssh-server
    ```

    **`openssh-server`** : Installe le démon (service en arrière-plan) permettant à la machine de recevoir des connexions SSH entrantes.

2. **Vérification du service.**

    ```Bash
    sudo systemctl status ssh
    ```

    **`systemctl status`** : Permet de s'assurer que le service est bien `active (running)`.

---
## 2. Connexion au serveur et identification

1. **Première connexion SSH.** Pour se connecter à distance, il faut l'IP du serveur et l'utilisateur cible :

    ```Bash
    ssh etudiant@172.16.51.51
    ```

    Lors de la première connexion, ce message d'avertissement apparaît :

    ```Bash
    The authenticity of host '172.16.51.51 (172.16.51.51)' can't be established.
    ED25519 key fingerprint is SHA256:IcllB0qMr5000JffqN62WHttQouyipyA1CNVi3D9hVQ.
    Are you sure you want to continue connecting (yes/no/[fingerprint])?
    ```

    > **Explication :** Ce message avertit que l'empreinte du serveur n'est pas encore enregistrée sur votre machine cliente (dans le fichier `~/.ssh/known_hosts`).

    **Vérifier l'empreinte de la clé public du serveur :**

    ```bash
    ssh-keygen -lvf /etc/ssh/ssh_host_ed25519_key.pub
    ```

    **À quoi servent les clés du serveur ?**

    Les fichiers `/etc/ssh/ssh_host_*_key` (privée) et `/etc/ssh/ssh_host_*_key.pub` (publique) servent à **prouver l'identité du serveur** (authentifie le serveur) au client. Cela empêche les attaques de l'homme du milieu (MITM), où un serveur pirate tenterait de se faire passer pour le vôtre.

---
## 3. Authentification par clés asymétriques

1. **Création d'une paire de clés SSH (Sur le poste client).**

    ```Bash
    ssh-keygen -t ed25519 -C "votre_email@example.com"
    ```

    **`ssh-keygen`** : L'outil de génération de clés.

    **`-t ed25519`** : Spécifie l'algorithme. Ed25519 est aujourd'hui la norme recommandée (plus rapide et plus sécurisée que RSA).

    **`-C`** : Ajoute un commentaire (souvent l'email) pour identifier facilement à qui appartient la clé.

    > **Bonne pratique :** Il est crucial de définir une **passphrase** lors de la création. Elle chiffre votre clé privée locale (`id_ed25519`). Si un attaquant vole ce fichier, il ne pourra pas l'utiliser sans ce mot de passe.

2. **Mise en place de la clé publique sur le serveur.** La méthode la plus propre et sécurisée est d'utiliser l'outil dédié plutôt que de copier manuellement, car il gère automatiquement les droits d'accès des dossiers cibles.

    ```Bash
    ssh-copy-id -i ~/.ssh/id_ed25519.pub etudiant@172.16.51.51
    ```

    **`ssh-copy-id`** : Va se connecter au serveur (avec le mot de passe de l'utilisateur) et copier le contenu de votre clé publique (`.pub`) dans le fichier `~/.ssh/authorized_keys` du serveur distant.

---
## 4. Hardening de la configuration SSH

Maintenant que la connexion par clé fonctionne, nous allons sécuriser le serveur en modifiant son fichier de configuration principal.

1. **Édition du fichier de configuration.**

    ```Bash
    sudoedit /etc/ssh/sshd_config
    ```

2. **Paramètres à modifier ou décommenter :**

    ```Plaintext
    PermitRootLogin no
    PasswordAuthentication no
    PubkeyAuthentication yes
    ```

    **`PermitRootLogin no`** : Interdit la connexion SSH directe avec le compte `root`. C'est la règle d'or en sécurité. On se connecte avec un utilisateur standard, puis on utilise `sudo`.

    **`PasswordAuthentication no`** : Désactive totalement l'authentification par mot de passe. Seules les clés SSH seront acceptées.

    **`PubkeyAuthentication yes`** : Activation l'authentification via une paire de clé.

3. **Application des modifications.**

    ```Bash
    sudo systemctl restart ssh
    ```

    **`systemctl restart`** : Relance le service pour qu'il lise la nouvelle configuration. _(Attention : Ne fermez pas votre session SSH actuelle avant d'avoir testé l'accès par clé sur un autre terminal, au risque de vous retrouver bloqué à l'extérieur !)_

    > **Note sur le port par défaut :**
    > 
    > Changer le port d'écoute (ex: `Port 2222`) n'ajoute pas de vraie sécurité (un scan de ports le trouvera en quelques secondes). Cependant, cela reste une "bonne pratique de confort" car cela **réduit drastiquement la pollution de vos logs** causée par les robots qui tentent de faire du brute-force en permanence sur le port 22.

---
## 5. Gestion des empreintes serveurs

1. **Voir l'empreinte d'un serveur SSH**

    ```bash
    ssh-keygen -H -F <ip ou domaine>
    ```

    **`ssh-keygen`** : Outil standard de création et de gestion des clés d'authentification SSH.

    **`-F <ip ou domaine>`** : Recherche (Find) et affiche les entrées correspondant à cet hôte précis dans ton fichier client known_hosts.

    **`-H`** : Indique que la recherche gère les noms d'hôtes hachés (une mesure de sécurité qui masque les IP et domaines dans le fichier known_hosts).

    **Astuce** : L'ajout de l'option -l (ssh-keygen -l -F <ip>) est plus utile pour lire l'empreinte (fingerprint) courte et lisible (SHA256) au lieu de la clé brute.

2. **Supprimer une clé publique d'un serveur SSH**

    ```bash
    ssh-keygen -R <ip ou domaine>
    ```

    **`ssh-keygen`** : L'outil de gestion des clés.

    **`-R <ip ou domaine>`** : Supprime (Remove) toutes les clés associées à cet hôte du fichier known_hosts. C'est la commande incontournable pour résoudre la fameuse alerte de sécurité "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!" qui survient quand un serveur a été réinstallé ou a changé de clé cryptographique.

---
⚠️ Approche TOFU = Trust on the first use

SSH FP => publier l'empreinte SSH dans la zone de l'entreprise