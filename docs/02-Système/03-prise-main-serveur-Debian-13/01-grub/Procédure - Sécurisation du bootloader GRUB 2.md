# Procédure - Sécurisation du bootloader GRUB 2

![Logo IMDEO](https://github.com/AP-BTS-SIO-Louis/imdeo/raw/main/docs_imdeo/docs/assets/logo_imdeo.jpg)

---
## Informations

- **Mainteneur :** MEDO Louis
- **Date :** 06/03/2026

---
## Sommaire

- [A. Configuration du clavier en AZERTY](#a-configuration-du-clavier-en-azerty)
- [B. Création de l'utilisateur et du mot de passe](#b-creation-de-lutilisateur-et-du-mot-de-passe)
- [C. Application des modifications](#c-application-des-modifications)

---
## A. Configuration du clavier en AZERTY

1. **Création du répertoire des dispositions.** Par défaut, GRUB ne reconnaît que le clavier en QWERTY.

    ```bash
    sudo mkdir /boot/grub/layouts
    ```

    - `mkdir` (Make Directory) crée le dossier cible. `sudo` exécute la commande avec les privilèges d'administration.

2. **Génération du fichier de disposition.** On génère la disposition du clavier dans un fichier reconnu par GRUB.

    ```Bash
    sudo grub-kbdcomp -o /boot/grub/layouts/fr.gkb fr
    ```

    - `grub-kbdcomp` convertit la disposition clavier du système vers le format GRUB. Le paramètre `-o` définit le fichier de sortie, et `fr` indique la langue source.

3. **Modification des paramètres par défaut.** Il est nécessaire de définir ce changement dans la configuration du bootloader.

    ```Bash
    sudoedit /etc/default/grub
    # Ajouter la ligne :
    GRUB_TERMINAL_INPUT=at_keyboard
    ```

    - `sudoedit` permet d'éditer un fichier de manière sécurisée. La directive
    - `GRUB_TERMINAL_INPUT=at_keyboard` indique à GRUB d'utiliser le clavier physique tel que configuré plutôt que l'entrée standard.

4. **Ajout du clavier dans 40_custom.** 

    ```bash
    # Commande pour modifier le grub
    sudoedit /etc/grub.d/40_custom 

    # Ajout des informations dans le fichier
    # Attention : Ajouter ce code APRÈS le premier "exec" du fichier (et non le deuxième).  
    insmod keylayouts  
    keymap fr
    ```

    - `insmod` insère le module GRUB gérant les dispositions de clavier. 
    - `keymap fr` applique la cartographie française définie à l'étape 2.*

5. **Ouvrir le script générateur de Debian.** Nous allons ouvrir le générateur de script Debian pour ajouter l'exception au script de démarrage Debian.

   ```bash
   sudoedit /etc/grub.d/10_linux
   ```
   **Rappel** : `sudoedit` édite le fichier de manière sécurisée avec les privilèges d'administration.

6. **Ajouter l'autorisation de démarrage.** Chercher la variable `CLASS` (souvent située dans les 50 premières lignes).

    ```bash
    CLASS="--class gnu-linux --class gnu --class os --unrestricted"
    ```
   
---
## B. Création de l'utilisateur et du mot de passe {#b-creation-de-lutilisateur-et-du-mot-de-passe}

1. **Génération du condensat (hash) du mot de passe.** Il faut créer un utilisateur et un mot de passe afin de s'authentifier. La première étape consiste à obtenir le hash.

    ```Bash
    grub-mkpasswd-pbkdf2
    ```

    -  L'outil `grub-mkpasswd-pbkdf2` invite à saisir un mot de passe et génère une empreinte cryptographique PBKDF2 sécurisée. Conservez la chaîne générée.

2. **Définition des identifiants.** Il faut ensuite fournir ce hash et un nom d'utilisateur dans le fichier de configuration.

    ```Bash
    # Commande pour modifier le grub
    sudoedit /etc/grub.d/40_custom

    # Toujours après le premier "exec", à la suite de la configuration clavier :
    set superusers="adminsio"
    password_pbkdf2 adminsio <VOTRE_HASH>
    ```

    - `set superusers` définit "adminsio" comme un compte ayant les privilèges de modifier les paramètres d'amorçage. 
    - `password_pbkdf2` associe ce compte au mot de passe haché que vous avez généré.

---
## C. Application des modifications

1. **Mise à jour et redémarrage.** Il reste enfin à prendre en compte ces nouveaux paramètres et à redémarrer.

    ```Bash
    sudo update-grub
    sudo shutdown -r now
    ```

    - `update-grub` compile les scripts du dossier `/etc/grub.d/` pour générer le fichier de configuration final lu par le bootloader au démarrage. 
    - `shutdown -r now` déclenche un redémarrage immédiat. Dorénavant, une authentification sera demandée pour modifier la séquence de boot avec la touche "e".