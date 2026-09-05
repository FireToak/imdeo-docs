# S3 - Procédure - Implémentation du protocole VTP

**Situation 3 : Distribution automatique de la base Vlan sur l'ensemble des commutateurs de la société**

**Contexte : IMDEO**

![Bannière IMDEO](https://imdeo.bts.loutik.fr/assets/banniere_imdeo.png)

---
## Informations générales

* **Créateur :** Louis MEDO
* **Date de création :** 20/01/2026
* **Dernière modification :** 05/02/2026
* **Validation technique :** Validé par Louis MEDO

---
## Sommaire

- A. Configuration du serveur VTP
- B. Configuration client VTP

---
## A. Configuration du serveur VTP

> Avant toutes configuration du protocole VTP sur l'infrastructure, il convient de bien remettre à zéro le switch afin d'avoir une configuration propre est d'évité des d'éventuelles conflits. 

1. **Vérification.** Vérifier que le serveur VTP est activé sur le switch avant de commencé.

    ```CISCO
    show vtp status
    ```

    **Résultat attendu :**

    ```BASH
    VTP Operating Mode : Server # Cette doit afficher server
    Maximum VLANs supported locally : 255
    Number of existing VLANs : 5
    Configuration Revision : 0
    MD5 digest : 0x7D 0x5A 0xA6 0x0E 0x9A 0x72 0xA0 0x3A
    0xF0 0x58 0x10 0x6C 0x9C 0x0F 0xA0 0xF7
    ```

    > Si le mode n'est pas "server" vous pouvez l'activer avec la commande `vtp mode server`

2. **Configuration du domaine VTP.**

    ```
    sw_vtp(config)# vtp domain IMDEO
    ```

3. **Mise en place du mot de passe VTP.**

    ```CISCO
    sw_vtp(config)# vtp password imdeo_007
    ```

4. **Activation du VTP version 2.**

    ```CISCO
    sw_vtp(config)# vtp version 2
    ```

5. **Création des vlan sur le switch.**

    ```
    ! Mode enable
    switch> en

    ! Mode configuration
    switch# conf t

    ! Création d'un vlan
    switch(config)# vlan 99
    switch(config-if)# name Admin
    ```

6. **Vérification de la bonne configuration.**

    ```
    sw_vtp# show vtp status
    ```

    **Résultat attendu :**

    ```
    VTP Version capable : 1 to 2
    VTP version running : 2
    VTP Domain Name : IMDEO
    VTP Pruning Mode : Disabled
    VTP Traps Generation : Disabled
    Device ID : 0060.3E86.2500
    Configuration last modified by 0.0.0.0 at 3-1-93 00:28:57
    Local updater ID is 0.0.0.0 (no valid interface found)

    Feature VLAN :
    --------------
    VTP Operating Mode : Server
    Maximum VLANs supported locally : 255
    Number of existing VLANs : 6
    Configuration Revision : 3
    MD5 digest : 0x24 0x15 0x21 0x17 0x88 0xFB 0x4C 0xAC
    0x22 0x08 0x7F 0x4A 0xE7 0xC5 0xF3 0x0D
    ```

7. **Configuration du port trunk.** Ce port trunk permet de partager les VLAN entre les différents commutateurs.

    ```
    ! Changer par votre interface
    sw_vtp(config)# interface fa 0/1
    sw_vtp(config-if)# description Liaison TRUNK VTP
    sw_vtp(config-if)# switch mode trunk
    sw_vtp(config-if)# switch trunk allow vlan 3,2,9,4,5,99
    ```

---
## B. Configuration client VTP

1. **Configuration du port trunk.** Ce port trunk permet de partager les VLAN entre les différents commutateurs.

    ```
    ! Changer par votre interface
    sw(config)# interface fa 0/1
    sw(config-if)# description Liaison TRUNK VTP
    sw(config-if)# switch mode trunk
    sw(config-if)# switch trunk allow vlan 3,2,9,4,5,99
    ```

2. **Passage en mode VTP 2.**

    ```CISCO
    sw(config)# vtp version 2
    ```

2. **Passage du service VTP en mode client.**

    ```CISCO
    sw(config)# vtp mode client
    ```

3. **Configuration du mot de passe.**

    ```
    sw(config)# vtp password imdeo_007
    ```

4. **Vérification.** Vérifier que les vlans sont bien récupérer par le switch.

    ```
    sw# sh vtp status
    ```

    **Résultat attendu :**

    ```
    VTP Version capable : 1 to 2
    VTP version running : 2
    VTP Domain Name : IMDEO
    VTP Pruning Mode : Disabled
    VTP Traps Generation : Disabled
    Device ID : 0001.9626.3300
    Configuration last modified by 0.0.0.0 at 3-1-93 00:58:20

    Feature VLAN :
    --------------
    VTP Operating Mode : Client
    Maximum VLANs supported locally : 255
    Number of existing VLANs : 11
    Configuration Revision : 13
    MD5 digest : 0x1B 0x2F 0x09 0xC6 0x69 0x0C 0x53 0x23
    0x7D 0x84 0xBC 0xFF 0xB0 0x88 0xF4 0x63
    ```

---
## Annexe

- [S3 - Playbook - Numéro de révision client supérieur au serveur VTP](https://ap-bts-sio-louis.github.io/imdeo/01-R%C3%A9seau/02-VTP/S3%20-%20Playbook%20-%20Num%C3%A9ro%20de%20r%C3%A9vision%20client%20sup%C3%A9rieur%20au%20serveur%20VTP/)