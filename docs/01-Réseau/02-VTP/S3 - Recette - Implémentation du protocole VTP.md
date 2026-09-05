# S3 - Recette - Implémentation du protocole VTP

**Situation 3 : Distribution automatique de la base Vlan sur l'ensemble des commutateurs de la société**

**Contexte : IMDEO**

![](https://ap-bts-sio-louis.github.io/imdeo/assets/logo_imdeo.jpg)

---

## Sommaire

- A. Vérification serveur VTP
- B. Vérification client VTP

---

## A. Vérification serveur VTP

### 1. Statut de configuration VTP

- **Description :** 
  Exécuter la commande `show vtp status`. *(Cette commande vérifie les ports configurés en mode agrégation (Trunk). Le protocole VTP ne transmet ses mises à jour qu'à travers ces liens).*

- **Résultat attendu :** 
  Le champ "VTP Operating Mode" indique **Server**. Le "VTP Domain Name" est correctement renseigné. Le "Configuration Revision" est supérieur à 0 (indiquant que des modifications ont été apportées).

- **Commentaire :** 

--

- [ ] Reçu
- [ ] Reçu avec réserve
- [ ] Refusé

### 2. Base de données des VLANs locaux

- **Description :** 
  Exécuter la commande `show vlan brief`. *(Cette commande liste tous les VLANs existants sur le commutateur, leurs noms et les ports qui y sont affectés).*

- **Résultat attendu :** 
  Tous les VLANs créés pour l'infrastructure IMDEO sont présents, actifs et correctement nommés sur ce commutateur maître.

- **Commentaire :** 

--

- [ ] Reçu
- [ ] Reçu avec réserve
- [ ] Refusé

---
## B. Vérification client VTP

### 1. Vérification des liaisons Trunk

- **Description :** 
  Exécuter la commande `show interfaces trunk`. _(Cette commande vérifie les ports configurés en mode agrégation (Trunk). Le protocole VTP ne transmet ses mises à jour qu'à travers ces liens)._

- **Résultat attendu :** 
  Les interfaces reliant le client au serveur (ou aux autres commutateurs) apparaissent dans la liste avec le statut "trunking".

- **Commentaire :** 

--

- [ ] Reçu
- [ ] Reçu avec réserve
- [ ] Refusé

### 2. Statut d'appairage VTP du client

- **Description :** 
  Exécuter la commande `show vtp status`.

- **Résultat attendu :** 
  Le champ "VTP Operating Mode" indique **Client**. Le nom de domaine VTP correspond exactement à celui du serveur. Le "Configuration Revision" est **identique** à celui du serveur.

- **Commentaire :** 

--

- [ ] Reçu
- [ ] Reçu avec réserve
- [ ] Refusé

### 3. Synchronisation de la base VLAN

- **Description :** 
  Exécuter la commande `show vlan brief`.

- **Résultat attendu :** 
  L'intégralité des VLANs créés sur le serveur VTP apparaît automatiquement sur ce commutateur, sans aucune configuration manuelle préalable.

- **Commentaire :** 

--

- [ ] Reçu
- [ ] Reçu avec réserve
- [ ] Refusé

---
## C. Vérification globale

### 1. Test de propagation dynamique d'un VLAN

 - **Description :** 
   Sur le serveur VTP, créer un nouveau VLAN de test (ex: `vlan 999` puis `name Test_VTP`). Patienter quelques secondes, puis sur le commutateur client, exécuter la commande `show vlan brief`.
   
 - **Résultat attendu :** 
   Le VLAN 999 nommé "Test_VTP" apparaît automatiquement dans la liste des VLANs du client, confirmant la bonne propagation via VTP.
   
 - **Commentaire :** 

--

 - [ ] Reçu
 - [ ] Reçu avec réserve
 - [ ] Refusé