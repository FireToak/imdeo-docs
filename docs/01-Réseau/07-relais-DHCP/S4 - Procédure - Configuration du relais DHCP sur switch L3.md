# Procédure - Configuration du relais DHCP (IP Helper) sur un commutateur L3

**Situation : Permettre aux équipements d'un VLAN d'obtenir une adresse IP depuis un serveur DHCP situé dans un autre sous-réseau.**

**Contexte : IMDEO**

![](https://ap-bts-sio-louis.github.io/imdeo/assets/logo_imdeo.jpg)

---
## Informations générales

* **Créateur :** Louis MEDO
* **Date de création :** 26/03/2026
* **Dernière modification :** 26/03/2026
* **Validation technique :** Validé par Louis MEDO

---
## Sommaire

- A. Prérequis : Activation du routage
- B. Configuration de l'IP Helper sur l'interface
- C. Vérification et sauvegarde

---
## A. Prérequis : Activation du routage

> Sur un commutateur de niveau 3, le routage IP doit être activé globalement pour permettre le transfert des paquets entre les VLAN et l'envoi des requêtes vers le serveur DHCP distant.

1. **Activation du routage global.**

    ```CISCO
    ! Passage en mode configuration
    switch# conf t

    ! Activation du routage IP
    switch(config)# ip routing
    ```

    **Explications :**
    
    `conf t` : Raccourci de *configure terminal*, permet d'entrer en mode de configuration globale.
    
    `ip routing` : Active les fonctionnalités de routage de niveau 3 sur l'équipement.

---
## B. Configuration de l'IP Helper sur l'interface

> Le relais DHCP s'applique directement sur l'interface virtuelle (SVI) du VLAN où se trouvent les postes clients.

1. **Application de l'adresse du serveur DHCP.**

    ```CISCO
    ! Sélection de l'interface VLAN (Remplacer 10 par votre VLAN client)
    switch(config)# interface vlan 10

    ! Ajout de l'IP du serveur DHCP distant (Ex: 192.168.100.5)
    switch(config-if)# ip helper-address 192.168.100.5
    
    ! Retour au mode privilégié
    switch(config-if)# end
    ```

    **Explications :**

    `interface vlan [ID]` : Permet de configurer l'interface virtuelle associée au VLAN ciblé.

    `ip helper-address [IP_SERVEUR]` : Intercepte les requêtes DHCP (diffusées en broadcast) des clients locaux et les encapsule en requêtes unicast directes vers l'adresse IP du serveur spécifié.

    `end` : Quitte le mode de configuration spécifique pour revenir directement à la racine du mode privilégié.

---
## C. Vérification et sauvegarde

1. **Vérification de l'interface.** S'assurer que le commutateur a bien pris en compte l'adresse de relais.

    ```CISCO
    switch# show ip interface vlan 10
    ```

    **Résultat attendu (extrait) :**

    ```BASH
    Vlan10 is up, line protocol is up
      Internet address is 192.168.10.254/24
      Broadcast address is 255.255.255.255
      Address determined by setup command
      MTU is 1500 bytes
      Helper address is 192.168.100.5 # Vérifier la présence de cette ligne
    ```

2. **Sauvegarde de la configuration.**

    ```CISCO
    switch# write
    ```

    **Explications :**

    `show ip interface [INTERFACE]` : Affiche l'état et la configuration IP détaillée de l'interface, incluant l'adresse du helper.

    `write` : Raccourci très courant de *copy running-config startup-config*. Enregistre la configuration active dans la NVRAM pour qu'elle persiste au prochain redémarrage.

---
## Annexe

- [Procédure - Déploiement d'un serveur DHCP sous Windows Server](#)