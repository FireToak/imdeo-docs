# S4 - Recette - Implémentation du relais DHCP (IP Helper)

**Situation 4 : Distribution des adresses IP via un serveur DHCP distant sur différents sous-réseaux**

**Contexte : IMDEO**

![](https://ap-bts-sio-louis.github.io/imdeo/assets/logo_imdeo.jpg)

---

## Sommaire

- A. Vérification de la configuration de l'équipement relais
- B. Validation de l'accessibilité réseau
- C. Validation fonctionnelle côté client

---

## A. Vérification de la configuration de l'équipement relais

### 1. Présence de l'adresse de relais (IP Helper)

- **Description :** Exécuter la commande `show ip interface vlan 10` (remplacer par l'interface appropriée). *(Cette commande affiche les paramètres IP détaillés de l'interface et permet de vérifier si une adresse de transfert pour les diffusions/broadcasts est bien active).*

- **Résultat attendu :** La ligne **"Helper address is [IP_DU_SERVEUR_DHCP]"** est présente et l'adresse IP renseignée correspond exactement à celle du serveur DHCP distant.

- **Commentaire :** 

--

- [ ] Reçu
- [ ] Reçu avec réserve
- [ ] Refusé

---
## B. Validation de l'accessibilité réseau

### 1. Test de connectivité vers le serveur DHCP

- **Description :** Depuis l'équipement relais (Switch L3 ou Routeur), exécuter la commande `ping [IP_DU_SERVEUR_DHCP]`. *(Cette commande envoie des paquets ICMP pour s'assurer que le trafic unicast peut bien atteindre le serveur DHCP distant depuis le réseau local).*

- **Résultat attendu :** Le taux de réussite (Success rate) est de 100%. Le serveur DHCP est joignable.

- **Commentaire :** 

--

- [ ] Reçu
- [ ] Reçu avec réserve
- [ ] Refusé

---
## C. Validation fonctionnelle côté client

### 1. Obtention d'un bail DHCP dynamique

- **Description :** Connecter un poste client sur le port d'accès du VLAN correspondant. Sur le poste (si Windows), ouvrir l'invite de commande et exécuter `ipconfig /release` puis `ipconfig /renew`. *(Cette action force la carte réseau du PC à diffuser une nouvelle requête DHCP sur le réseau, qui sera interceptée par l'équipement relais).*

- **Résultat attendu :** Le poste client obtient avec succès une adresse IP valide appartenant au sous-réseau de son VLAN, ainsi que le bon masque de sous-réseau et la bonne passerelle par défaut.

- **Commentaire :** 

--

- [ ] Reçu
- [ ] Reçu avec réserve
- [ ] Refusé

### 2. Vérification côté Serveur DHCP

- **Description :** Sur le serveur DHCP distant, ouvrir la console d'administration DHCP et consulter la liste des baux actifs (ou exécuter `show ip dhcp binding` si le serveur est un équipement Cisco). *(Cela permet de confirmer que c'est bien ce serveur qui a fourni l'adresse et d'assurer le suivi des attributions).*

- **Résultat attendu :** L'adresse MAC du poste client de test apparaît dans la liste avec l'adresse IP qui vient de lui être attribuée.

- **Commentaire :** 

--

- [ ] Reçu
- [ ] Reçu avec réserve
- [ ] Refusé