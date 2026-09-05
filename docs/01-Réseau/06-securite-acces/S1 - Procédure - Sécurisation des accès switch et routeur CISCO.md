# S1 - Procédure - Sécurisation des accès switch et routeur CISCO

**Situation 1 : Préparation de l'infrastructure réseau d'IMDEO : Adressage et maquettage**

**Contexte : IMDEO**

![Bannière IMDEO](https://imdeo.bts.loutik.fr/assets/banniere_imdeo.png)

---
## Informations

- **Créateur :** Louis MEDO
- **Date de création :** 22/03/2026
- **Validation technique :**

---
## Sommaire

1. Sécurisation du mode d'exécution privilégié (Enable)
2. Sécurisation de l'accès physique (Console)
3. Chiffrement global des mots de passe
4. Vérification et tests de fonctionnement

---
## 1. Sécurisation du mode d'exécution privilégié (Enable)

1. **Protection du mode administrateur.** Restreindre l'accès au mode privilégié, ce qui protège par extension l'accès au mode `configure terminal`.

    ```cisco
    Switch> enable
    Switch# configure terminal
    Switch(config)# enable secret M0tDeP@sseF0rt!
    ```

    `enable secret` : Crée un mot de passe chiffré (avec l'algorithme MD5 ou SHA selon les versions) requis pour passer en mode privilégié. Il remplace la commande obsolète `enable password` qui stockait le mot de passe en clair.

-----

## 2. Sécurisation de l'accès physique (Console)

1. **Protection du port de management.** Verrouiller la connexion par câble physique (console).

    ```cisco
    Switch(config)# line con 0
    Switch(config-line)# password M0tDeP@sseConsole!
    Switch(config-line)# login
    Switch(config-line)# exit
    ```

    `line con 0` : Entre dans le mode de configuration de l'unique ligne physique de console (numérotée 0).

    `password` : Définit le mot de passe requis pour cette ligne.

    `login` : Active la vérification du mot de passe à la connexion (sans cela, le mot de passe est ignoré).

    `exit` : Remonte d'un niveau dans l'arborescence de configuration.

-----

## 3. Chiffrement global des mots de passe

1. **Sécurisation du fichier de configuration.** Chiffrer tous les mots de passe qui seraient autrement stockés en texte clair (comme celui de la console).

    ```cisco
    Switch(config)# service password-encryption
    Switch(config)# end
    Switch# write memory
    ```

    `service password-encryption` : Active une fonction globale qui applique un algorithme de hachage faible (type 7) sur tous les mots de passe en clair présents dans la configuration active.

    `end` : Quitte instantanément le mode de configuration globale pour revenir au mode privilégié.

    `write memory` : Enregistre la configuration en cours dans la mémoire NVRAM.

-----

## 4. Vérification et tests de fonctionnement

1. **Vérification du chiffrement.** Contrôler que les mots de passe n'apparaissent plus en clair dans la configuration.

    ```cisco
    Switch# show running-config
    ```

    **`show`** : Commande permettant d'afficher des informations sur l'état du système.

    **`running-config`** : Argument ciblant le fichier de configuration actuellement en cours d'exécution dans la mémoire vive (RAM).

2. **Test de la chaîne de connexion.** Simuler une déconnexion totale pour valider que les mots de passe sont bien exigés.

    ```cisco
    Switch# exit
    ```

    **`exit`** : Depuis le mode privilégié, cette commande ferme complètement la session utilisateur en cours.

    **Résultat attendu :**

    ```cisco
    Switch con0 is now available

    Press RETURN to get started.

    User Access Verification

    Password: 
    Switch> enable
    Password: 
    Switch#
    ```

-----

## Ressources

  - [Documentation Cisco - Sécurité des mots de passe des routeurs et commutateurs](https://www.cisco.com/c/fr_ca/support/docs/smb/switches/cisco-small-business-300-series-managed-switches/smb5563-configure-password-settings-on-a-switch-through-the-command.htmll)