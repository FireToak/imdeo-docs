# S1 - Procédure - Mise en place du routage sur un switch L3 CISCO

**Situation 1 : Préparation de l'infrastructure réseau d'IMDEO : Adressage et maquettage**

**Contexte : IMDEO**

![Bannière IMDEO](https://imdeo.bts.loutik.fr/assets/banniere_imdeo.png)

---
## Informations

- **Créateur :** Louis MEDO
- **Date de création :** 21/03/2026
- **Validation technique :** 

---
## Sommaire

1. Activation du routage IP global.
2. Création des VLANs et de leurs interfaces de routage (SVI).
3. Configuration des liaisons inter-commutateurs (Trunk).
4. Mise en place de la route statique par défaut.

---
## 1. Activation du routage IP global

1. **Activer le routage.** Par défaut, un commutateur (même de niveau 3) agit au niveau 2 (Liaison de données). Il est obligatoire d'activer la fonctionnalité de routage pour qu'il puisse transmettre des paquets entre différents sous-réseaux.

```cisco
enable
configure terminal
ip routing
```

**`enable`** (ou `en`) : Passe du mode utilisateur au mode d'exécution privilégié.

**`configure terminal`** (ou `conf t`) : Entre dans le mode de configuration globale pour modifier les paramètres de l'équipement.

**`ip routing`** : Active le moteur de routage IP du commutateur L3.

---
## 2. Création des VLANs et de leurs interfaces (SVI)

1. **Déclarer le VLAN et sa passerelle.** Pour chaque réseau (ex: COM), il faut créer le VLAN, puis lui attribuer une interface virtuelle (SVI) avec une adresse IP qui servira de passerelle par défaut pour ce service.

```Cisco
vlan 3
name COM
exit
interface vlan 3
ip address 192.168.1.30 255.255.255.224
exit
```

`vlan 3` : Crée le VLAN numéro 3 dans la base de données.

`name COM` : Nomme le VLAN pour l'identifier facilement.

`interface vlan 3` : Crée et entre dans l'interface de routage virtuelle associée au VLAN.

`ip address 192.168.1.30 255.255.255.224` : Assigne l'adresse IP et le masque, définissant ainsi la passerelle du réseau.

`exit` (ou `ex`) : Remonte d'un niveau dans le menu de configuration.

---
## 3. Configuration des liaisons inter-commutateurs (Trunk)

1. **Configurer le port en mode agrégation.** Pour transporter le trafic de plusieurs VLANs vers un autre commutateur, le port doit être en mode Trunk, en y autorisant strictement les VLANs nécessaires.

```Cisco
interface gigabitEthernet 0/1
switchport mode trunk
switchport trunk allowed vlan 2,3,4,5,9
exit
```

`interface gigabitEthernet 0/1` (ou `gig0/1`) : Sélectionne l'interface physique à configurer.

`switchport mode trunk` : Force le port de manière statique à agir comme un lien Trunk 802.1Q.

`switchport trunk allowed vlan ...` : Par mesure de sécurité, restreint le trafic du Trunk aux seuls VLANs utilisés en production.

---
## 4. Mise en place de la route statique par défaut

1. **Définir la passerelle de dernier recours.** Le switch L3 connaît les VLANs directement connectés, mais a besoin d'une route pour envoyer le trafic inconnu (comme Internet) vers le routeur de sortie (FAI).

```Cisco
ip route 0.0.0.0 0.0.0.0 192.168.1.97
```

`ip route` : Commande de création d'une route statique.

`0.0.0.0 0.0.0.0` : Désigne tous les réseaux de destination avec n'importe quel masque (la route par défaut). (Note technique : l'utilisation de `255.255.255.255` vue dans les maquettes est atypique pour une route par défaut, on privilégiera des zéros).

`192.168.1.97` : Adresse IP de l'équipement de prochain saut (l'interface interne du routeur NAT).

---
## Ressources

- [Documentation CISCO - Configuration du routage inter-VLAN](https://www.cisco.com/c/en/us/support/docs/lan-switching/inter-vlan-routing/41260-189.html)