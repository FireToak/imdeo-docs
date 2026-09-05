# Procédure - Configuration du relais DHCP (IP Helper) sur un routeur Cisco

**Situation : Permettre aux équipements d'un réseau local d'obtenir une adresse IP depuis un serveur DHCP situé sur un autre réseau.**

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

- A. Configuration de l'IP Helper sur l'interface
- B. Vérification et sauvegarde

---
## A. Configuration de l'IP Helper sur l'interface

> Contrairement à un switch de niveau 3, un routeur effectue déjà du routage par défaut. La commande de relais DHCP s'applique directement sur l'interface physique (ou la sous-interface) connectée au réseau local des postes clients.

1. **Application de l'adresse du serveur DHCP.**

    ```CISCO
    ! Passage en mode de configuration globale
    router# configure terminal

    ! Sélection de l'interface connectée au LAN (ex: GigabitEthernet 0/0)
    router(config)# interface g0/0

    ! Ajout de l'IP du serveur DHCP distant (Ex: 192.168.100.5)
    router(config-if)# ip helper-address 192.168.100.5
    
    ! Retour au mode privilégié
    router(config-if)# end
    ```

    **Explications des commandes :**

    `configure terminal` : Accède au mode de configuration globale.

    `interface g0/0` : Sélectionne l'interface physique. *Note : Si l'architecture utilise du routage inter-VLAN (Router-on-a-Stick), il faut sélectionner la sous-interface correspondante, par exemple `interface g0/0.10`.*

    `ip helper-address [IP_SERVEUR]` : Convertit les requêtes DHCP broadcast (locales) en requêtes unicast envoyées directement vers le serveur DHCP distant.

    `end` : Quitte instantanément le mode de configuration d'interface pour revenir à la racine du mode privilégié.

---
## B. Vérification et sauvegarde

1. **Vérification de l'interface.** S'assurer que le routeur a bien enregistré l'adresse de relais.

    ```CISCO
    router# show ip interface g0/0
    ```

    **Résultat attendu (extrait) :**

    ```BASH
    GigabitEthernet0/0 is up, line protocol is up
      Internet address is 192.168.10.254/24
      Broadcast address is 255.255.255.255
      Address determined by setup command
      MTU is 1500 bytes
      Helper address is 192.168.100.5 # Cette ligne confirme le relais
    ```

2. **Sauvegarde de la configuration.**

    ```CISCO
    router# write
    ```

    **Explications des commandes :**

    `show ip interface [INTERFACE]` : Affiche les informations IP de l'interface, dont la présence ou non de l'adresse Helper.
    
    `write` : Sauvegarde la configuration active (`running-config`) dans la mémoire NVRAM (`startup-config`).

---
## Annexe

- [Procédure - Configuration du routage inter-VLAN (Router-on-a-Stick) sur un routeur Cisco](#)