# S3 - Numéro de révision client supérieur au serveur VTP

**Situation 3 : Distribution automatique de la base Vlan sur l'ensemble des commutateurs de la société**

**Contexte : IMDEO**

![Bannière IMDEO](https://imdeo.bts.loutik.fr/assets/banniere_imdeo.png)

---
## Informations générales

- **Créateur :** Louis MEDO
- **Date de création :** 20/01/2026
- **Dernière modification :** 05/02/2026
- **Validation technique :** Validé par Louis MEDO

---
## Sommaire

- A. Concepts du VTP et Numéro de révision
- B. Procédure 1 : Réinitialisation via le Nom de Domaine
- C. Procédure 2 : Réinitialisation via le Mode VTP

---
## A. Concepts du VTP et Numéro de révision

Le protocole VTP (VLAN Trunking Protocol) est utilisé pour créer, gérer et maintenir un grand réseau comportant plusieurs commutateurs sur le même réseau local physique. Il permet de gérer l'ajout, la suppression et le renommage des VLAN depuis un serveur central sans intervention manuelle, réduisant ainsi la charge administrative.

Le numéro de révision de configuration est un nombre sur 32 bits indiquant le niveau de révision d'un paquet VTP. Ce numéro s'incrémente de un à chaque modification de VLAN sur un équipement VTP. Cette information sert à déterminer si les données reçues sont plus récentes que la version actuelle.

---
## B. Procédure 1 : Réinitialisation via le Nom de Domaine

Cette méthode consiste à changer le nom de domaine VTP puis à restaurer la configuration initiale.

1. **Vérification de l'état initial.** Consulter le numéro de révision actuel.

    ```CISCO
    SW1> enable
    SW1# show vtp status
    ```

    `enable` : Permet de passer du mode utilisateur au mode d'exécution privilégié (nécessaire pour exécuter des commandes d'affichage avancées ou de configuration).
    `show vtp status` : Affiche l'état du VTP, notamment la version, le numéro de révision actuel, le nombre de VLANs et le nom de domaine.

2. **Modification du nom de domaine VTP.** Entrer un domaine temporaire.

    ```CISCO
    SW1# configure terminal
    SW1(config)# vtp domain OTHER
    ```

    `configure terminal` : Ouvre le mode de configuration globale pour modifier les paramètres du commutateur.
    `vtp domain OTHER` : Change le nom de domaine VTP actuel par la valeur "OTHER".

3. **Restauration du domaine initial.** Remettre le nom de domaine d'origine.

    ```CISCO
    SW1(config)# vtp domain CISCO
    ```

    `vtp domain CISCO` : Restaure le nom de domaine tel qu'il était avant la modification (ici "CISCO").

4. **Vérification de la réinitialisation.** Confirmer que le numéro est bien redescendu à zéro.

    ```CISCO
    SW1# show vtp status
    ```

---
## C. Procédure 2 : Réinitialisation via le Mode VTP

Il s'agit de la méthode privilégiée et la plus rapide pour effacer le numéro de révision. Elle consiste à passer le commutateur en mode "Transparent" puis à le remettre dans son mode initial.

1. **Vérification de l'état initial.** S'assurer du numéro de révision avant manipulation.

    ```CISCO
    SW1> enable
    SW1# show vtp status
    ```

2. **Changement du mode VTP vers Transparent.** Modifier le mode de fonctionnement.

    ```CISCO
    SW1# configure terminal
    SW1(config)# vtp mode transparent
    ```

    `vtp mode transparent` : Bascule le commutateur en mode transparent. Dans ce mode, le commutateur ne synchronise plus sa configuration VLAN avec les autres équipements, ce qui force la réinitialisation de son propre numéro de révision.

3. **Restauration du mode VTP Serveur.** Rétablir le rôle principal du commutateur.

    ```CISCO
    SW1(config)# vtp mode server
    ```

    `vtp mode server` : Replace le commutateur en mode serveur, lui permettant à nouveau de gérer et propager la configuration VLAN sur le réseau.

4. **Vérification de la réinitialisation.** Contrôler la mise à zéro du compteur.

    ```CISCO
    SW1# show vtp status
    ```