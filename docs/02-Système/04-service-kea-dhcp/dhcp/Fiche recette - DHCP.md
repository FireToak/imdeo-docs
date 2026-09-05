# Fiche recette - DHCP

![Bannière IMDEO](https://imdeo.bts.loutik.fr/assets/banniere_imdeo.png)

---

## Informations

  - **Mainteneur :** MEDO Louis
  - **Date :** 13/03/2026
  - **Documentation :** [IMDEO documentation technique](https://ap-bts-sio-louis.github.io/imdeo/)

---

## A. Vérification des serveurs DHCP (172.16.51.51 & 172.16.51.52)

Avant de tester le client, nous devons nous assurer que le service est opérationnel sur les deux serveurs KEA.

1.  **Vérifier l'état du service KEA sur les deux serveurs.**

    ```bash
    sudo systemctl status kea-dhcp4-server
    ```

      * **`systemctl`** : Outil de gestion des services systemd.
      * **`status`** : Affiche l'état actuel du processus.

    **Résultat attendu :** 
    
    Statut `Active: active (running)` sur les deux serveurs. 
    
    [ ] **OK** / [ ] **KO**

2.  **Surveiller les logs DHCP en temps réel (optionnel pour la suite du test).**

    ```bash
    sudo tail -f /var/log/kea-dhcp4.log
    ```

      * **`tail`** : Commande qui affiche la fin d'un fichier.
      * **`-f`** (follow) : Maintient le fichier ouvert et affiche les nouvelles lignes ajoutées en temps réel. Utile pour voir les requêtes arriver lors des tests clients.

----

## B. Vérification du relais DHCP (Routeur Cisco)

Le routeur doit être capable d'intercepter les requêtes DHCP diffusées (broadcast) par le client et de les rediriger (unicast) vers les serveurs KEA.

1.  **Afficher la configuration de l'interface passerelle (ex: VLAN 10).**

    ```cisco
    show running-config interface vlan 10
    ```

      * **`show running-config`** : Affiche la configuration active en mémoire vive (RAM) de l'équipement Cisco.
      * **`interface vlan 10`** : Cible la vérification sur l'interface virtuelle (ou physique) servant de passerelle par défaut aux clients.

    **Résultat attendu :**

    ```cisco
    interface Vlan10
     ip address 192.168.10.254 255.255.255.0
     ip helper-address 172.16.51.51
     ip helper-address 172.16.51.52
    ```

      * **`ip helper-address`** : Commande Cisco qui active la fonction de relais DHCP (DHCP Relay Agent). Elle transforme la requête DHCPDISCOVER (broadcast) du client en requête unicast dirigée vers les adresses IP spécifiées.
      * **Résultat de la vérification :** Présence des deux IP Helpers confirmée. [ ] **OK** / [ ] **KO**

-----

## C. Vérification de l'attribution IP (Poste Client Windows)

Nous allons simuler l'arrivée d'un nouveau poste sur le réseau pour vérifier la chaîne complète (Client -\> Routeur -\> Serveur).

1.  **Libérer le bail DHCP actuel.**

    ```cmd
    ipconfig /release
    ```

      * **`ipconfig`** : Utilitaire réseau Windows affichant et gérant la configuration TCP/IP.
      * **`/release`** : Envoie un message DHCPRELEASE au serveur pour abandonner l'adresse IP actuelle.

2.  **Demander un nouveau bail DHCP.**

    ```cmd
    ipconfig /renew
    ```

      * **`/renew`** : Déclenche le processus DORA (Discover, Offer, Request, Acknowledge) pour obtenir une nouvelle adresse IP.

3.  **Vérifier les paramètres réseau attribués.**

    ```cmd
    ipconfig /all
    ```

      * **`/all`** : Affiche la configuration détaillée, incluant l'adresse MAC, le serveur DNS, et surtout l'adresse du serveur DHCP qui a répondu.
      
    **Résultat attendu :** 
    
    Le client obtient une adresse IP dans la bonne plage, avec la bonne passerelle, et le champ "Serveur DHCP" indique `172.16.51.51` ou `172.16.51.52`. 
    
    [ ] **OK** / [ ] **NON**

-----

## D. Test de tolérance aux pannes (Haute Disponibilité)

Puisque nous avons deux serveurs DHCP configurés en IP Helper, nous devons vérifier que le second prend le relais si le premier tombe.

1.  **Simuler une panne sur le serveur DHCP principal (172.16.51.51).**

    ```bash
    sudo systemctl stop kea-dhcp4-server
    ```

      * **`stop`** : Arrête immédiatement le service.

2.  **Refaire une demande de bail sur le client.**

    ```cmd
    ipconfig /release
    ipconfig /renew
    ```

3.  **Vérifier la provenance du nouveau bail.**

    ```cmd
    ipconfig /all
    ```

      * **Résultat attendu :** Le client obtient une adresse IP avec succès, et le champ "Serveur DHCP" indique désormais `172.16.51.52` (le serveur secondaire). [ ] **OK** / [ ] **KO**

4.  **Remettre en service le serveur principal.**

    ```bash
    sudo systemctl start kea-dhcp4-server
    ```