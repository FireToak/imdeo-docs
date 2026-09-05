# S1 - Procédure - Mise en place du SSH CISCO

**Situation 1 : Préparation de l'infrastructure réseau d'IMDEO : Adressage et maquettage**

**Contexte : IMDEO**

![](https://ap-bts-sio-louis.github.io/imdeo/assets/logo_imdeo.jpg)

---
## Informations

- **Créateur :** Louis MEDO
- **Date de création :** 22/03/2026
- **Validation technique :**

---
## Sommaire

1. Configuration des prérequis
2. Génération des clés cryptographiques
3. Configuration des accès et sécurisation VTY
4. Vérification et tests de fonctionnement

---
## 1. Configuration des prérequis

1. **Définition de l'identité de l'équipement.** Le protocole SSH nécessite obligatoirement un nom d'hôte et un nom de domaine configurés pour pouvoir générer les clés cryptographiques.

    ```cisco
    Router> enable
    Router# configure terminal
    Router(config)# hostname R1-IMDEO
    R1-IMDEO(config)# ip domain-name imdeo.local
    ```

    **`enable`** : Fait passer l'utilisateur du mode utilisateur simple (limité) au mode d'exécution privilégié.

    **`configure terminal`** : Permet d'entrer dans le mode de configuration globale du routeur pour modifier ses paramètres.

    **`hostname`** : Modifie le nom système de l'équipement.

    **`ip domain-name`** : Attribue un nom de domaine au routeur. Ce paramètre sera combiné au nom d'hôte (`R1-IMDEO.imdeo.local`) pour générer le certificat matériel.

-----

## 2. Génération des clés cryptographiques

1. **Création des clés RSA.** Le SSH est un protocole chiffré. Il faut générer une paire de clés (publique et privée) pour chiffrer les échanges. La bonne pratique impose un minimum de 2048 bits.

    ```cisco
    R1-IMDEO(config)# crypto key generate rsa modulus 2048
    ```

    **`crypto key generate rsa`** : Ordonne au routeur de créer une paire de clés asymétriques utilisant l'algorithme RSA.

    **`modulus 2048`** : Définit la taille de la clé. Plus elle est longue, plus elle est difficile à casser. 2048 bits est le standard de sécurité recommandé aujourd'hui.

-----

## 3. Configuration des accès et sécurisation VTY

1. **Création du compte et restriction des lignes.** Il faut créer un administrateur local sécurisé, forcer le standard SSH version 2 (plus robuste) et fermer les accès non chiffrés.

    ```cisco
    R1-IMDEO(config)# username admin privilege 15 secret M0tDeP@sseF0rt!
    R1-IMDEO(config)# ip ssh version 2
    R1-IMDEO(config)# line vty 0 4
    R1-IMDEO(config-line)# login local
    R1-IMDEO(config-line)# transport input ssh
    R1-IMDEO(config-line)# end
    R1-IMDEO# write memory
    ```

    **`username [...] privilege 15 secret [...]`** : Crée un compte utilisateur. `privilege 15` lui accorde les droits d'administration totaux. `secret` stocke le mot de passe dans la configuration de manière chiffrée (contrairement à `password` qui le stocke en clair).

    **`ip ssh version 2`** : Désactive la version 1 de SSH (qui contient des vulnérabilités connues) pour exiger la version 2.

    **`line vty 0 4`** : Entre dans le mode de configuration des 5 premières lignes de "Virtual Teletype" (accès distants virtuels).

    **`login local`** : Indique au routeur d'utiliser sa propre base de données locale (l'utilisateur `admin` créé plus haut) pour authentifier les connexions entrantes.

    **`transport input ssh`** : Restreint les connexions entrantes au seul protocole SSH. Cela bloque notamment Telnet, qui est un protocole non chiffré et dangereux.

    **`write memory`** : Sauvegarde la configuration active dans la mémoire non volatile (NVRAM) pour qu'elle survive à un redémarrage.

-----

## 4. Vérification et tests de fonctionnement

1. **Contrôle du service et connexion.** Vérifier côté serveur (routeur) que le service est opérationnel, puis tester la connexion côté client (PC).

    ```cisco
    R1-IMDEO# show ip ssh
    ```

    **Résultat attendu :**

    ```cisco
    SSH Enabled - version 2.0
    Authentication timeout: 120 secs; Authentication retries: 3
    ```

2. **Test depuis une machine cliente (PC).** Ouvrir un terminal sur un ordinateur de maquette et s'y connecter :

    ```cisco
    PC> ssh -l admin 192.168.1.254
    ```

    **`show ip ssh`** : Affiche l'état d'activation, la version de SSH qui tourne, ainsi que les délais d'attente d'authentification.

    **`ssh -l`** : Commande client pour initier la connexion. L'argument `-l` (login) permet de spécifier le nom d'utilisateur, suivi de l'adresse IP de l'interface du routeur ciblé.

-----

## Ressources

  - [Documentation Cisco - Configuration de Secure Shell](https://www.cisco.com/c/fr_ca/support/docs/security-vpn/secure-shell-ssh/4145-ssh.html)